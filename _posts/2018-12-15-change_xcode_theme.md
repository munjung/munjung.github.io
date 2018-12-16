---
layout: post
title: Xcode 테마 변경하기
subtitle: Dracula 테마 설치하기
tags: [Xcode]
---

IOS의 개발 툴인 Xcode의 테마를 변경하는 방법에 대해 포스팅하려고 한다.      
먼저 Xcode의 기본 테마는 하얀색이다.  

![default_theme](/img/181215/181215_img_1.png)


`Xcode - Preferences - Fonts & Colors`에 들어간다.  




![Preferences](/img/181215/181215_img_2.png)
왼쪽에 기본 테마들이 있어서 클릭할 때마다 변하는 것을 확인할 수 있다. 그러니 마음에 드는것을 선택하면 된다.  
나는 Android Studio에서 쓰는  `Dracula`를 좋아해서 해당 테마를 추가하기로 했다.  


---
1. 아래 Github 주소에 있는 파일 다운로드후 압축 해제
- [https://github.com/jesseXu/xcode-theme-Darcula](https://github.com/jesseXu/xcode-theme-Darcula)  
2. Darcula.dvtcolortheme 파일 확인 (형식: [테마명].dvtcolortheme)
3. 해당 폴더로 이동
```
~/Library/Developer/Xcode/UserData/FontAndColorThemes/
```   
* Spotlight 이용하면 간단
![use_spotlight](/img/181215/181215_img_3.png){: width="600" height="250"}
4. 폴더 안에 파일 삽입
5. Xcode 재실행  

---  


Dracula라는 새로운 테마가 추가된 것을 볼 수 있다.
![new_theme_dracula](/img/181215/181215_img_4.png)


아쉽게도 전체 테마 색상은 바뀌지 않고 에디터와 콘솔창에만 적용된다.  

그 외에도 다양한 테마를 원한다면 아래의 주소를 참고할 것

[https://github.com/hdoria/xcode-themes](https://github.com/hdoria/xcode-themes)  
[http://www.codethemes.net/](http://www.codethemes.net/)
