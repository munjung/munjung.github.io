---
layout: post
title: 오토레이아웃(AutoLayout) 정리-2
subtitle: 코드를 이용하여 오토레이아웃 구현하기
tags: [Xcode, swift]
---

## 오토레이아웃?
디바이스 사이즈에 구애받지 않고 `시각적으로 동일한 화면`을 구현해야할 때 필요한 것으로 `view의 제약 사항`을 바탕으로 모든 view의 크기와 위치를 `동적`으로 계산한다.  

### 오토레이아웃을 구현하는 방법
- 코드를 이용하여 구현
- 인터페이스 빌더에서 구현

[이전 포스팅](/2018-12-16-autolayout_by_code_1/)에서 설명한대로 `NSLayoutConstraint`를 이용하여 오토레이아웃을 구현하는 방법을 알아보자.
- 오토레이아웃 방정식  
view1.attr1 = view2.attr2 * multiplier + constant  
item.attribute = toItem.attribute * multiplier + constant  
![autolayout_equation](/img/181217/181217_img_2.png)  

- NSLayoutConstraint 인스턴스 생성 제약조건
![nslayout_condition](/img/181217/181217_img_1.png)  


---
해당 포스트는 [부스트캠프](https://www.edwith.org/boostcourse-ios)의 강의를 정리한 내용입니다.  
