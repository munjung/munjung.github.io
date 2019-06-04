---
layout: post
title: 안드로이드 카카오맵 api 현재위치 받아오기
subtitle: 현재 주소 받아오기, 원 그리기
tags: [Android, Kotlin]
---

이틀동안 헤맸던 카카오맵 현재 위치 받아오기를 했다! 신난다 신나 🤸‍♀🤸‍♀🤸‍♀  
지도를 나오게 하는 것 까지는 성공했지만 첫 화면을 현재위치가 나오도록 설정해야 했다. 어떻게 해야할지 감이 오지 않아서 수업 시간에 계속 검색해보고 찾아봤다! 예전에 다음지도를 이용해서 개발한 지인이 있어서 코드를 참고하고 싶어 깃을 잠깐 열어 달라고 부탁했다. 지도를 구현한 부분의 코드를 쭉 보면서 현재위치를 받아오는 방법을 집중적으로 찾아봤다. 그 분은 안드로이드 내장 센서를 이용해서 GPS 정보를 알아오는 클래스를 구현했다. 다른 방법들을 찾아보고 없으면 나도 클래스를 직접 구현해야겠다 생각했는데 다행히 더 쉬운 방법을 찾았다!
[어떤 개발자님](https://webnautes.tistory.com/1319)께서 현재 위치를 받아오는 코드를 구현하셨다. 정말 천사 그자체👼
나와 달랐던 것은 `그분은 Java`로 개발을 하셨고, `나는 Kotlin` 이었다는 것이다. 코틀린으로 코드를 바꾸면서 기초 문법 공부도 덤으로 할 수 있었다. 또한 [카카오 샘플 코드](http://apis.map.daum.net/android/sample/)에서도 많이 참고했다. `LocationDemoActivity.java` 코드 위주로 보면 아주 도움이 된다.


- layout

~~~
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context="activity.MyLocationActivity"
        android:background="#ffffff">

    <RelativeLayout
            android:id="@+id/layout_search"
            android:layout_width="match_parent"
            android:layout_height="45dp"
            android:layout_marginTop="23dp"
            android:layout_marginLeft="20dp"
            android:layout_marginRight="20dp"
            android:layout_marginBottom="10dp"
            android:background="#d6d6d6">

        <TextView
                android:id="@+id/txt_my_location"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="현재 내 위치 : 서울역"
                android:textSize="18sp"
                android:textColor="#000000"
                android:layout_centerInParent="true"/>

        <Button
                android:id="@+id/btn_close"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="닫기"
                android:layout_alignParentRight="true"/>


    </RelativeLayout>
    <RelativeLayout
            android:id="@+id/mapView"
            android:background="#F1F8FD"
            android:layout_width="match_parent"
            android:layout_below="@+id/layout_search"
            android:layout_above="@+id/layout_menu_bar"
            android:layout_height="match_parent">
    </RelativeLayout>
    <RelativeLayout
            android:id="@+id/layout_path"
            android:background="#F1F8FD"
            android:layout_width="match_parent"
            android:layout_below="@+id/layout_search"
            android:layout_above="@+id/layout_menu_bar"
            android:layout_height="match_parent"
            android:visibility="invisible">

    </RelativeLayout>

    <RelativeLayout
            android:id="@+id/layout_menu_bar"
            android:layout_width="match_parent"
            android:layout_height="50dp"
            android:background="#0b78ad"
            android:layout_alignParentBottom="true">

        <Button
                android:id="@+id/btn_play"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_centerInParent="true"
                android:text="재생"/>

        <Button
                android:id="@+id/btn_map_change"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_alignParentRight="true"
                android:layout_centerVertical="true"
                android:text="지도변환"/>

    </RelativeLayout>

</RelativeLayout>
~~~

아직 아이콘은 없지만 전체적인 레이아웃은 아래와 같다. 가운데 하늘색 부분에 지도가 들어가야 한다.
![result_screen](/img/190604/190604_img_1.png){: width="400" height="600"}

- activity

~~~
package activity

import android.annotation.SuppressLint
import android.content.Context
import android.content.Intent
import android.content.pm.PackageManager
import android.graphics.Color
import android.location.LocationManager
import android.os.Bundle
import android.support.v4.app.ActivityCompat
import android.support.v4.content.ContextCompat
import android.support.v7.app.AlertDialog
import android.support.v7.app.AppCompatActivity
import android.util.Log
import android.widget.RelativeLayout
import android.widget.Toast
import com.km.deodeumi.R
import kotlinx.android.synthetic.main.activity_my_location.*
import mapapi.MapApiConst
import net.daum.mf.map.api.MapCircle
import net.daum.mf.map.api.MapPoint
import net.daum.mf.map.api.MapReverseGeoCoder
import net.daum.mf.map.api.MapView


class MyLocationActivity : AppCompatActivity(),MapView.CurrentLocationEventListener,MapReverseGeoCoder.ReverseGeoCodingResultListener {

    // var : 읽기/쓰기 가능한 일반 변수 val : 읽기만 가능한 final 변수
    private val GPS_ENABLE_REQUEST_CODE: Int = 200
    private val PERMISSIONS_REQUEST_CODE: Int = 100
    private var REQUIRED_PERMISSIONS = arrayOf<String>(android.Manifest.permission.ACCESS_FINE_LOCATION)

    private lateinit var mapView: MapView
    private lateinit var reverseGeoCoder: MapReverseGeoCoder


    //private var txt_location = findViewById<TextView>(R.id.txt_my_location)


    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_my_location)
        val mapViewContainer = findViewById<RelativeLayout>(R.id.mapView)
        mapView = MapView(this)
        mapViewContainer.addView(mapView)
        mapView.setCurrentLocationEventListener(this)

        txt_my_location.text = "현재 내 위치: "

        if(!checkLocationServiceStatus()){
            showDialogForLocationServiceSetting()
        }else{
            checkRunTimePermission()
        }

    }

    @Override
    override fun onDestroy() {
        super.onDestroy()
        mapView.currentLocationTrackingMode = MapView.CurrentLocationTrackingMode.TrackingModeOff
        mapView.setShowCurrentLocationMarker(false)
    }

    @Override
    override fun onRequestPermissionsResult(requestCode: Int, permissions: Array<out String>, grantResults: IntArray) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)

        if(requestCode == PERMISSIONS_REQUEST_CODE && grantResults.size == REQUIRED_PERMISSIONS.size){
            // 요청 코드가 PERMISSIONS_REQUEST_CODE 이고, 요청한 퍼미션 개수만큼 수신되었다면
            var check_result : Boolean = false
            for(index in 0..grantResults.size){
                if(index != PackageManager.PERMISSION_GRANTED){
                    check_result = true
                    break
                }
            }

            if(check_result){
                mapView.currentLocationTrackingMode = MapView.CurrentLocationTrackingMode.TrackingModeOnWithHeading
            }else{
                if(ActivityCompat.shouldShowRequestPermissionRationale(this, REQUIRED_PERMISSIONS[0])){
                    Toast.makeText(this, "퍼미션이 거부되었습니다. 앱을 다시 실행하여 퍼미션을 허용해주세요.", Toast.LENGTH_LONG).show()
                    finish()
                }
            }

        }
    }

    @Override
    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)
        when(requestCode){
            GPS_ENABLE_REQUEST_CODE -> {
                if(checkLocationServiceStatus()){
                    checkRunTimePermission()
                    return
                }
            }

        }
    }

    // MapView.CurrentLocationEventListener
    override fun onCurrentLocationUpdate(p0: MapView?, p1: MapPoint?, p2: Float) {
        val mapPointGeo : MapPoint.GeoCoordinate = p1!!.mapPointGeoCoord

        reverseGeoCoder = MapReverseGeoCoder(MapApiConst.DAUM_MAPS_ANDROID_APP_API_KEY,mapView.mapCenterPoint,this, this)
        reverseGeoCoder.startFindingAddress()

        Log.i("MyLocationActivity", String.format("MapView onCurrentLocationUpdate (%f,%f) accuracy (%f)", mapPointGeo.latitude, mapPointGeo.longitude, p2))
    }

    override fun onCurrentLocationUpdateFailed(p0: MapView?) {

    }

    override fun onCurrentLocationUpdateCancelled(p0: MapView?) {

    }

    override fun onCurrentLocationDeviceHeadingUpdate(p0: MapView?, p1: Float) {

    }


    //MapReverseGeoCoder.ReverseGeoCodingResultListener
    override fun onReverseGeoCoderFailedToFindAddress(p0: MapReverseGeoCoder?) {

    }


    override fun onReverseGeoCoderFoundAddress(p0: MapReverseGeoCoder?, p1: String?) {
        txt_my_location.text = "${txt_my_location.text}$p1"
    }


    private fun checkLocationServiceStatus():Boolean {
        var manager: LocationManager = getSystemService(Context.LOCATION_SERVICE) as LocationManager
        return manager.isProviderEnabled(LocationManager.GPS_PROVIDER) ||
                manager.isProviderEnabled(LocationManager.NETWORK_PROVIDER)
    }

    //GPS 활성화를 위한 메소드
    private fun showDialogForLocationServiceSetting(){
        val builder = AlertDialog.Builder(this@MyLocationActivity)
        builder.setTitle("위치 서비스 비활성화")
        builder.setMessage("앱을 사용하기 위해서는 위치 서비스가 필요합니다.\n"
                + "위치 설정을 수정하시겠습니까?")
        builder.setCancelable(true)
        builder.setPositiveButton("설정") { dialog, id ->
            val callGPSSettingIntent = Intent(android.provider.Settings.ACTION_LOCATION_SOURCE_SETTINGS)
            startActivityForResult(callGPSSettingIntent, GPS_ENABLE_REQUEST_CODE)
        }
        builder.setNegativeButton("취소") { dialog, id -> dialog.cancel() }
        builder.create().show()
    }

    //위치 퍼미션을 갖고 있는지 체크
    private fun checkRunTimePermission(){
        // 1. 위치 퍼미션을 가지고 있나
        var hasFineLocationPermission = ContextCompat.checkSelfPermission(this, android.Manifest.permission.ACCESS_FINE_LOCATION)
        // 2. 이미 퍼미션을 가지고 있다면
        // ( 안드로이드 6.0 이하 버전은 런타임 퍼미션이 필요없기 때문에 이미 허용된 걸로 인식합니다.)
        if(hasFineLocationPermission == PackageManager.PERMISSION_GRANTED){
            // 3. 위치값 가져오기
            mapView.currentLocationTrackingMode = MapView.CurrentLocationTrackingMode.TrackingModeOnWithHeading
        }else{ // 4. 퍼미션 요청을 허용한 적이 없다면 퍼미션 요청이 필요합니다. 2가지 경우(3-1, 4-1)가 있습니다.

            // 3-1. 사용자가 퍼미션 거부를 한 적이 있는 경우에는
            if(ActivityCompat.shouldShowRequestPermissionRationale(this,REQUIRED_PERMISSIONS[0])){
                // 3-2. 요청을 진행하기 전에 사용자가에게 퍼미션이 필요한 이유를 설명해줄 필요가 있습니다.
                Toast.makeText(this,"이 앱을 실행하려면 위치 접근 권한이 필요합니다.", Toast.LENGTH_LONG).show()
                // 3-3. 사용자게에 퍼미션 요청을 합니다. 요청 결과는 onRequestPermissionResult에서 수신됩니다.
                ActivityCompat.requestPermissions(this, REQUIRED_PERMISSIONS,
                    PERMISSIONS_REQUEST_CODE)
            }else{
                // 4-1. 사용자가 퍼미션 거부를 한 적이 없는 경우에는 퍼미션 요청을 바로 합니다.
                // 요청 결과는 onRequestPermissionResult에서 수신됩니다.
                ActivityCompat.requestPermissions(this, REQUIRED_PERMISSIONS,
                    PERMISSIONS_REQUEST_CODE)
            }

        }
    }

}

~~~

이렇게 해서 코드를 실행하게 되면 아래와 같이 현재 보고 있는 방향을 포함한 트래킹 모드가 실행된다.  
(트래킹 모드의 핵심은 `mapView.currentLocationTrackingMode = MapView.CurrentLocationTrackingMode.TrackingModeOff` 부분이다!)

![result_screen2](/img/190604/190604_img_2.jpeg){: width="400" height="600"}

사진에서와 같이 위치가 가끔씩 튄다는 문제가 있다. 이 부분은 추후에 보정을 해야겠지만 일단 대체적으로 정확하다.
다음으로 내가 알아야 할 것은 현재 주소였다. 그 부분은 개발자 홈페이지에서도 설명해주셔서 MapReverseGeoCoder 클래스를 사용했다.

![present_location](/img/190604/190604_img_3.png)

~~~
reverseGeoCoder = MapReverseGeoCoder(MapApiConst.DAUM_MAPS_ANDROID_APP_API_KEY,mapView.mapCenterPoint,this, this)
reverseGeoCoder.startFindingAddress()
~~~

주소 또한 사용자의 위치에 따라 주기적으로 업데이트 되어야 하므로 onCurrentLocationUpdate 안에서 사용했다. 주소를 받아오는 것을 성공하면 onReverseGeoCoderFoundAddress가 호출된다고 해서 해당 메서드의 p1을 출력해봤다. 주소가 잘 출력된다! 그래서 onReverseGeoCoderFoundAddress 메서드 안에서 주소를 텍스트뷰에 셋팅하는 작업을 했다. 그런데 현 주소가 안나오고 자꾸 서울 중구로만 설정이 됐다. mapView.mapCenterPoint를 로그로 찍어보니 처음에 서울 중구가 나왔고 그 후로 업데이트가 될 때는 현재 주소로 잘 나왔다. 내 추측이지만 mapView.mapCenterPoint의 기본 값이 서울 중구로 설정된 것 같다.  

그래서 이 문제를 해결하고자 mapView.mapCenterPoint 대신 바로 단말기의 현재 위치를 가져올 수 있는 위도, 경도로 대체하였다.
생각보다 굉장히 간단했다. 아래와 같이 변경해서 지도 맵을 켜자마자 해당 주소로 텍스트를 설정하는 것은 해결!
~~~
var presentPoint = MapPoint.mapPointWithGeoCoord(mapPointGeo.latitude, mapPointGeo.longitude)
       reverseGeoCoder = MapReverseGeoCoder(MapApiConst.DAUM_MAPS_ANDROID_APP_API_KEY,presentPoint,this, this)
       reverseGeoCoder.startFindingAddress()
~~~

onReverseGeoCoderFoundAddress 메서드도 살짝 수정해줬다. p1의 값이 전체 주소로 길게 나온다. 주소를 어디까지 보여줄지에 대해서는 아직 팀원들과 의논하지 않았기에 일단은 임의로 잘랐다.
~~~
var short_address: String = p1!!.substring(0,11) //어떤 기준으로 자를 것인지?
        Log.i("현재 주소",p1)
        if (txt_my_location.text.length == 9){
            txt_my_location.text = "${txt_my_location.text}$short_address"
        }
~~~

마지막으로 해야할 것은 현재위치에서 50m 이내를 원으로 표시하는 것 이었다. 이 부분은 개발자 사이트에서도 워낙 잘 나와있어서 금방 할 수 있었다. 마찬가지로 onCurrentLocationUpdate에 코드를 추가해줬다.
![map_circle](/img/190604/190604_img_4.png)

mapView.moveCamera도 해봤지만 너무 과하게 확대가 되길래 그 부분은 삭제했다. 사실 돌려보면 알겠지만 이 코드에도 문제가 있다. onCurrentLocationUpdate가 주기적으로 업데이트 되기 때문에 circle이 무한 증식하게 된다.ㅋㅋㅋ 이 부분은 나중에 따로 수정을 할 예정이다.
~~~
val circle = MapCircle(
            MapPoint.mapPointWithGeoCoord(mapPointGeo.latitude, mapPointGeo.longitude), // center
            50, // radius (반경 50m)
            Color.argb(128, 255, 203, 203), // strokeColor
            Color.argb(128, 255, 203, 203) // fillColor
        )
        circle.tag = 1
        mapView.addCircle(circle)
~~~

최종 완성 화면! 아직 해야할 것은 산더미로 남았지만 하나씩 차근차근 하고 있는 것 같아서 기쁘당😀
의도치 않은 주밍아웃이지만 블로그 검색을 막아놨으니 괜찮아!
![final_screen](/img/190604/190604_img_5.jpeg){: width="400" height="600"}


코틀린으로 코드를 변경하면서 코틀린에 대해서도 조금씩 알아가는게 재밌다. lateinit, companion object 등 알면 알수록 신기한 문법이 많은 것 같다. 나중에 따로 코틀린 문법에 대해서도 정리해봐야겠다.
