---
layout: post
title: 카카오맵 REST API 사용하기
subtitle: 위경도 WTM 변환 및 직선거리 구하기
tags: [Android, Kotlin]
---

지난 포스팅 이후 기말고사 & 면접 준비로 인해 정신이 없었다. 해커톤 예선 발표가 얼마 남지 않아서 개발에 박차를 가했다. 내가 해야할 일은 현재 위치에서 목적지 까지의 직선 거리를 측정하는 것이었다. 좌표만을 가지고 거리를 어떻게 측정해야할지 몰라 난감했는데 [카카오 DevTalk](https://devtalk.kakao.com/t/topic/42237/3)에 비슷한 글이 있었다. 결론부터 말하면 카카오에서 제공하는 REST API를 이용해 현재 카카오의 기본 좌표계인 `WGS84`형식에서 `WTM`으로 변환해야 한다. 자세한것은 [개발 가이드](https://developers.kakao.com/docs/restapi/local#%EC%A2%8C%ED%91%9C%EA%B3%84-%EB%B3%80%ED%99%98)에 나와있다. 좌표계의 종류가 이렇게 다양한지 처음 알았다.



## - Model

Response 형식은 아래와 같다.

![api_response](/img/190616/190616_img_1.png)

- LocationObject  

~~~
package models

class LocationObject {
    data class meta (
        val total_count: Int
    )

    data class documents (
        val y: String,
        val x: String
    )
}
~~~


- DistanceObject

~~~
package models

object DistanceObject {
    data class Distance (
        val meta: LocationObject.meta,
        val documents: ArrayList<LocationObject.documents>
    )
}
~~~

## - Interface

- DistanceInterface

~~~
interface DistanceInterface {
    // 좌표 변환
    @Headers("Authorization: KakaoAK " + RestAPIKey.kakao)
    @GET("v2/local/geo/transcoord.json")
    fun distanceConverter(
        @Query("x") x:String,
        @Query("y") y:String,
        @Query("input_coord") input_coord:String,
        @Query("output_coord") output_coord:String
        ) : Observable<DistanceObject.Distance>
}
~~~

사실 input_coord, output_coord는 필수적인 값은 아니었지만,
좌표계를 변환하기 위해서 넣어줬다.

## - Retrofit

- LocationService

~~~
package service

object LocationService {
    fun distanceRestAPI(): DistanceInterface {
        return LocationService.retrofitInterface().create(DistanceInterface::class.java)
    }
    private val httpClient = OkHttpClient.Builder()
        .addNetworkInterceptor { chain: Interceptor.Chain -> chain.proceed(chain.request().newBuilder().build()) }!!

    private fun retrofitInterface(): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://dapi.kakao.com/")
            .addConverterFactory(GsonConverterFactory.create())
            .addCallAdapterFactory(RxJava2CallAdapterFactory.createWithScheduler(Schedulers.io()))
            .client(LocationService.httpClient.build())
            .build()
    }
}
~~~

## - Activity

~~~
// WGS84 -> WTM
    private fun locationConverter(x: String,y: String,input_coord: String,output_coord: String) {

        subscription = LocationService.distanceRestAPI().distanceConverter(x,y,input_coord,output_coord)
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(
                { result ->
                    res1 = result.documents[0].x
                    res2 = result.documents[0].y
                },
                { err ->
                    Log.e("Error User",err.toString())
                }
            )

    }
~~~

res1, res2에는 WTM으로 변환된 좌표값이 들어오게 된다. 그래서 현재위치 좌표(x1, y1)와 목적지 좌표(x2, y2)를 각각 WTM으로 변환해서 두 점간 거리 구하는 공식을 이용하면 된다. 그런데 사실 이부분에서 문제가 있었다. 지도에서 임의로 선택한 곳이 목적지라고 가정하고 개발을 했기 때문에 locationConverter 메서드가 총 2번 불려져야 했다.

또한 결과 값인 res1, res2를 계산에 써야 했는데, 로그로 출력해보니 null이 나왔다. 이상하다 싶어 result.documents[0].x 를 출력했는데 그건 또 잘나온다. 😵😵😵 확인 결과 res는 result 스코프 안에서만 값이 존재했다. 알고보니 subscription이 `비동기 방식`이기 때문에 그런 거였다. 어떻게 처리할지 고민하다가 한가지 꼼수를 부리기로 했다.😃 전역 변수로 크기가 4인 비어있는 배열(res)을 선언하고 res[0],res[2]의 값이 null인지 체크해서 res에 좌표가 전부 채워질 때까지 기다리는 것이다.

~~~
// WGS84 -> WTM
    private fun locationConverter(x: String,y: String,input_coord: String,output_coord: String) {

        subscription = LocationService.distanceRestAPI().distanceConverter(x,y,input_coord,output_coord)
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(
                { result ->

                    if(this.res[0] == null){
                        res[0] = result.documents[0].x
                        res[1] = result.documents[0].y
                    }else if(this.res[2] == null){
                        res[2] = result.documents[0].x
                        res[3] = result.documents[0].y
                        calculatorDistance(res[0].toString(),res[1].toString(),res[2].toString(),res[3].toString())
                        for (item in res.indices){
                            res[item] = null
                        }
                    }
                },
                { err ->
                    Log.e("Error User",err.toString())
                }
            )
    }
~~~

배열이 전부 채워지면 거리를 계산하는 calculatorDistance() 메서드가 호출 되고, 배열에 있는 값들은 전부 null로 초기화 시켰다.
그래서 현재 위치와 목적지까지의 직선 거리를 구할 수 있었다. 
