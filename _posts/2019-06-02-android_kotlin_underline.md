---
layout: post
title: 안드로이드 스튜디오 코틀린 밑줄 제거
subtitle: 변수 var 밑줄 제거하기
tags: [Android]
---

카카오맵 API에서 현재위치로 업데이트를 시도중인데 매우 어렵다. 카카오 개발자 홈페이지에도 나와있지 않고, 다른 블로그를 참고하면서 하고있는데도 계속 오류가 난다. 한 하루 이틀은 더 고생할 것 같은 예감이 든다.후우우우 아직 코틀린도 낯설어서 기본적인 문법도 검색해서 하고있다. 알듯말듯한데 뭔가 JS, Swift를 섞어놓은 것 같다. 혼란하다 혼란해. 그런데 개발하면서도 계속 신경쓰이는 부분이 있었다.

![anroid_underline](/img/190602/190602_img_1.png)  

**manager** 여기 왜 자꾸 밑줄이 쳐지는거냐. 저거 뿐만 아니라 위에있는 변수에도 밑줄이 가있었다.  
뭔가 저기에만 줄이 쳐져 있으니까 내가 잘못한 것 같다는 생각이 들었다. 그래서 이유를 알아보려고 했지만 별다른 소득이 없었다.
자꾸 신경쓰여서 없애고 싶었다. ㅎㅎㅎ

`Preferences -> Editor -> Kotlin -> Properties and Variables`

![setting](/img/190602/190602_img_2.png)  

저기서 오른쪽에 있는 Effects를 제거하면 깔끔해진다. 👍👍👍  
그나저나 왜 현재위치가 안되는걸까... 해결하면 기뻐서 춤도출 수 있을 것 같다.🤸‍♀
