---
layout: post
title: 오토레이아웃(AutoLayout) 정리-3
subtitle: 코드를 이용하여 오토레이아웃 구현하기
tags: [Xcode, swift]
---

## 오토레이아웃?
디바이스 사이즈에 구애받지 않고 `시각적으로 동일한 화면`을 구현해야할 때 필요한 것으로 `view의 제약 사항`을 바탕으로 모든 view의 크기와 위치를 `동적`으로 계산한다.  

### 오토레이아웃을 구현하는 방법
- 코드를 이용하여 구현
- 인터페이스 빌더에서 구현

### 코드를 이용하여 오토레이아웃을 구현하는 방법
- Layout Anchor
- NSLayoutConstraint
- Visual Format Language

이번에는 `Visual Format Language`를 이용하여 오토레이아웃을 구현하는 방법을 알아보자.
- Visual Format Language 에서 사용 가능한 기호와 문자열
![available_vfl_sign](/img/181218/181218_img_1.png)  


---
해당 포스트는 [부스트코스](https://www.edwith.org/boostcourse-ios)의 강의를 정리한 내용입니다.  
