---
layout: post
title: 자바스크립트 프로그래스바 추가하기
subtitle: display:none의 조합
tags: [JavaScript, HTML, CSS]
---

서버 개발을 마치고, 2일간은 프론트에 매달렸다. HTML/CSS를 배운지 꽤 됐기 때문에, 잘 기억이 나지 않았다. 특히 원하는 부분에 배치하는 것이 어려워서 다른 블로그들을 보며 CSS를 공부했다. 이번에 정리해볼 것은 프로그래스바를 추가하는 것이다. 이미지 검색시에 API에 request를 요청하고 결과를 받아와 프론트에 출력해줬는데, API에서 분석하는 시간이 대략 4초 정도 걸렸다. 프로그램을 개발한 우리의 입장에서는 단지 검색 버튼을 누르고 기다리면 되는 거였지만, 새로운 사용자가 프로그램을 사용한다고 가정을 했을 때, 4초는 너무나도 긴 시간이라고 생각 되었다. 열심히 이미지를 분석하고 있지만, 누군가에게는 오류로 보일 수 있기 때문이다. 그래서 결과가 나올 동안 로딩 중임을 보여줄 수 있는 프로그래스 바를 추가하기로 하였다.  

## - HTML에 div 추가
~~~
  <div class="loader" style="display:none;">
  </div>
  <div class="container" style="display:none;">
  </div>
~~~

## - CSS 추가
~~~
.loader{
  position: absolute;
  left: 50%;
  top: 50%;
  z-index: 1;
  margin: -75px 0 0 -75px;
  border: 16px solid #f3f3f3;
  border-radius: 50%;
  border-top: 16px solid #ACD3F4;
  width: 80px;
  height: 80px;
  -webkit-animation: spin 2s linear infinite;
  animation: spin 2s linear infinite;
}


@-webkit-keyframes spin {
  0% {-webkit-trnasform: rotate(0deg);}
  100% {-webkit-transform: rotate(360deg);}
}

@keyframes spin {
  0% {transform: :rotate(0deg);}
  100% {transform: rotate(360deg);}
}
~~~
CSS에서 애니메이션을 추가할 수 있다는 것이 신기했다. `keyframes`을 이용하면, 애니메이션의 중간 상태를 점검할 수 있다. 더 자세한 내용은 [여기](https://developer.mozilla.org/ko/docs/Web/CSS/CSS_Animations/Using_CSS_animations)를 참고하는 것이 좋다.

## - JS 추가
~~~
var loader = $("div.loader");
function closeLoader(){
  loader.css("display","none");
}
function openLoader(){
  loader.css("display","");
}
~~~
loader를 보이게 설정하면, 프로그래스바가 돌아가는 것을 확인할 수 있다. 우리의 서비스는 검색 버튼을 누르고 결과가 나올 때까지만 프로그래스 바가 보여야 했기 때문에 저렇게 따로 메서드를 만들어서 필요한 부분에 호출하는 방식으로 제어하였다.
![progress_bar](/img/191112/191112_img_1.png)
그렇게 프로그래스바 추가 완성하기 성공! 사실 이 부분은 확장앱을 제출하고 난 후에, 보완하는 과정에서 추가한 기능이다. 업데이트가 가능하게 되면 얼른 배포하고 싶다.
