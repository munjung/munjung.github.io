---
layout: post
title: 안드로이드 스튜디오 카카오맵 API 연동
subtitle: Kakao Developers 업데이트 해주세요ㅠㅠ
tags: [Android]
---

서울 통합이동서비스(MaaS) 해커톤에 나가게 되었다!(👏👏👏) 사실 저저번주에 1차 사전행사가 진행됐을만큼 꽤나 시간이 흘렀다. 그런데 나도 그렇고 팀원들도 그렇고 모두 취준생들이기 때문에 몹시 바빴다. 즉, 개발을 하나도 진행하지 못했다는 말이다.😀 어제 시청역에서 2차 사전교육을 진행해서 참석했는데 개발에 대한 시급함을 느끼게 되었다. 멘토분들의 말씀을 들어보니 예정보다 개발할 것이 늘어날 수도 있다는 예감이 들었다. 그리고 생각해보니 예선까지는 고작 3주밖에 남지 않았다. 다른팀들 보니까 조금씩은 진행하시고 있는 것 같길래 우리도 오늘부터 개발을 본격적으로 시작하기로 했다. 우리팀은 Kotlin으로 안드로이드 앱을 만들기로 했다. 나는 지도 부분을 맡았다. 우선 카카오맵 API를 사용하기로해서 쉽게 연동할려고 했는데 오늘도 어김없이 삽질했다. 그래서 한 4시간정도 헤매고 해결했다. 기본적인 카카오맵 연동 방법은 다른 블로그에도 많이 나와있으니 그걸 참고하도록! 나는 [우리 팀원](https://sodp2.tistory.com/1) 블로그를 보고했다.

## - 키 해시
![key_hash_image](/img/190601/190601_img_1.png)  

저 아래에 있는 키 해시 부분을 채워줘야 지도가 제대로 보이는 것을 확인할 수 있다. 키 해시를 얻을 수 있는 방법은 2가지가 있다. `안드로이드 스튜디오에서 직접 코드`를 쳐서 확인하는 방법과 `터미널을 이용`하는 방법이 있다.

### 1. 코드로 직접 쳐서 확인

대부분의 사람들이 직접 코드를 쳐서 확인하는 것을 선호하길래 나도 그렇게 했다. 아래의 코드를 치고 실행을 하면 된다고 하길래 입력을 해봤다. your.package.name 에는 프로젝트의 패키지 이름을 넣으면 된다고 하길래 시도를 했다. (자바로 입력해도 저절로 코틀린 언어로 바꿔줘서 신기했다.)
~~~
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    try {
        PackageInfo info = getPackageManager().getPackageInfo("your.package.name", PackageManager.GET_SIGNATURES);
        for (Signature signature : info.signatures) {
            MessageDigest md = MessageDigest.getInstance("SHA");
            md.update(signature.toByteArray());
            Log.d("KeyHash:", Base64.encodeToString(md.digest(), Base64.DEFAULT));
        }
    } catch (PackageManager.NameNotFoundException e) {
        e.printStackTrace();
    } catch (NoSuchAlgorithmException e) {
        e.printStackTrace();
    }
}
~~~

그런데..  
![code_deprecated](/img/190601/190601_img_2.png)  

벌써 두개나 deprecated 되었다. 찾아보니까 재작년까지만해도 저렇게 잘 쓰는 것 같던데, 언제부터 deprecated 된 것일까 너무 슬펐다. 검색해보니까 PackageManager.GET_SIGNATURES은 PackageManager.GET_SIGNING_CERTIFICATES 로 사용한다고 한다. 그리고 그 밖의 코드는 아래와 같이 수정해주었다.
~~~
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        try {
            if (Build.VERSION.SDK_INT >= 28) {
                @SuppressLint("WrongConstant") val packageInfo =
                    packageManager.getPackageInfo("your.package.name", PackageManager.GET_SIGNING_CERTIFICATES)
                val signatures = packageInfo.signingInfo.apkContentsSigners
                val md = MessageDigest.getInstance("SHA")
                for (signature in signatures) {
                    md.update(signature.toByteArray())
                    val signatureBase64 = String(Base64.encode(md.digest(), Base64.DEFAULT))
                    Log.d("Signature Base64", signatureBase64)
                }
            }
        } catch (e: PackageManager.NameNotFoundException) {
            e.printStackTrace()
        }

    }
~~~
오류는 없어졌는데 로그에는 아무것도 뜨지 않았다. if문 에서 걸리나 싶어서 디버그창으로 Build.VERSION.SDK_INT을 확인해봤더니 26이 나왔다. 이상하다 싶어서 build.gradle를 확인해 봤는데 딱히 문제가 없어 보였다. 어떤 분의 말대로 Project Structure - Modules - Build Tool Version을 28로 설정해 주었는데도 계속해서 26이 나왔다. 혹시나하고 프로젝트 내에서 26으로 설정되어있는 부분을 찾아봤지만 없었다. 그래서 Google Developer에서 찾아보기로 했다. 바로 원인을 알 수 있었다!
![build_gradle_check2](/img/190601/190601_img_4.png)

그렇구나 하드웨어 디바이스 문제였다. 내 폰이 문제였던 것이다. ㅎㅎㅎ 바로 업데이트를 시켜줬다. 그리고 아주 빠르게 키 해시 값을 얻을 수 있었다. 아래의 그림과 같이 로그로 잘 출력된다.
![key_hash_value](/img/190601/190601_img_5.png)

### 2. 터미널을 이용해서 확인

~~~
keytool -exportcert -alias androiddebugkey -keystore  
~/.android/debug.keystore  
-storepass android -keypass android | openssl sha1 -binary | openssl base64
~~~

위의 명렁어를 터미널에 입력하게 되면 아래와 같이 해시 값이 출력된다.
코드를 통해 얻은 값과 동일하다.

![key_hash_value_terminal](/img/190601/190601_img_6.png)

## - 라이브러리 추가

카카오 개발자 사이트에 올라와있는 SDK를 다운받고, 프로젝트의 libs 폴더에 추가해주면 된다.   
![sdk_library](/img/190601/190601_img_7.png)
개발자 사이트에서 사진으로도 보여주면서 워낙 친절하게 설명해주셔서 아무런 의심없이 라이브러리를 추가했는데 자꾸 오류가 났다.
분명히 라이브러리를 넣어줬는데 읽어오지 못한다고해서 너무 황당했다!!! 그래서 검색해보니까 카카오 개발자 사이트에 나와있는대로 하면 안된다고 한다.😀😀😀 .so 파일을 담고있는 폴더들은 `Project명 > app > src > main`에서 `jniLibs`라는 디렉토리를 생성하고 그 안에다가 넣어줘야 한다고 한다.  

![sdk_library2](/img/190601/190601_img_8.png){: width="600" height="400"}  

그리고 AndroidManifest.xml 에 Permission 과 APP KEY 추가한다.
~~~
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />

<application>
...
생략
...
<meta-data android:name="com.kakao.sdk.AppKey"
                   android:value="네이티브 앱 키 삽입"/>
</application>
~~~

## - 코드 작성하기
일단 오늘은 카카오 맵과 연동하는 것이 주 목표기 때문에, 전체화면을 꽉 채우기로 했다.

- layout  

~~~
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MapTestActivity">
    <RelativeLayout
            android:id="@+id/mapView"
            android:layout_width="match_parent"
            android:layout_height="match_parent">
    </RelativeLayout>

</RelativeLayout>
~~~

- activity

~~~
class MapTestActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_map_test)

        val mapView = MapView(this)
        val mapViewContainer = findViewById<RelativeLayout>(R.id.mapView)
        mapViewContainer.addView(mapView)


    }
}
~~~

## - 실행하기
당연히 실행이 잘 될거라고 했는데 그렇지 않았다. 😀 그래 역시 맘대로 되는게 어딨어...  
실행자체도 되지 않아서 상당히 당황스러웠다. 대신 이상한 오류 메세지들만 잔뜩 나왔다.

>Process: com.neighbor.jinjubus, PID: 978
java.lang.NoClassDefFoundError: Failed resolution of: Lorg/apache/http/message/HeaderGroup;
at net.daum.mf.map.common.net.NetWebClient...

이때가 한 3시간 반정도 시간이 걸렸을때라 많이 지친 상태였다. 조금 쉬었다가 오류를 찾기 시작했고, 꽤 쉽고 간단하게 해결할 수 있었다.
답은 AndroidManifest에 있었는데 두 줄을 추가하니까 해결됐다.

~~~
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="your.package.name">
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<application>
  <uses-library android:name="org.apache.http.legacy" android:required="false"/>
</application>
</manifest>
~~~

왜 이 두줄을 추가해야만 하는지는 [Google Developer](https://developer.android.com/about/versions/pie/android-9.0-changes-28?hl=ko) 에 가면 자세하게 알 수 있다.  
안드로이드 버전이 9(pie)가 되면서 네트워크나 보안측면이 더욱 강화된 것 같다.   
![result_screen](/img/190601/190601_img_9.jpeg){: width="400" height="600"}  

이게 뭐라고 감동...😂 내일은 현재위치 먼저 잡는 것과 반경 50m표시하는 것을 해야겠다.  
안드로이드 오랜만에 하니까 신나고 어렵다.🤔🤔🤔
