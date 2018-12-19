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

### 코드를 이용하여 오토레이아웃을 구현하는 방법
- Layout Anchor
- NSLayoutConstraint
- Visual Format Language

이번에는 `NSLayoutConstraint`를 이용하여 오토레이아웃을 구현하는 방법을 알아보자.
- 오토레이아웃 방정식  
view1.attr1 = view2.attr2 * multiplier + constant  
item.attribute = toItem.attribute * multiplier + constant  
![autolayout_equation](/img/181217/181217_img_2.png)  

- NSLayoutConstraint 인스턴스 생성 제약조건  
![nslayout_condition](/img/181217/181217_img_1.png)  

- ViewController을 통해 button과 label을 superview에 추가  
![add_button_label_at_superview](/img/181217/181217_img_3.png)  

- superview를 기준으로 button을 중앙 정렬 해준다.  
![add_button_label_at_superview](/img/181217/181217_img_4.png)    

- 실행을 하면 button이 중앙 정렬된 걸 확인할 수 있다.  
![button_center_arrange](/img/181217/181217_img_5.png){: width="300" height="500"}    

- label도 정렬해주자(button을 기준으로!)  
![button_center_arrange](/img/181217/181217_img_6.png)

````
NSLayoutConstraint(item: mylabel,
  attribute: NSLayoutConstraint.Attribute.trailing,
  relatedBy: NSLayoutConstraint.Relation.equal,
  toItem: mybutton,
  attribute: NSLayoutConstraint.Attribute.leading,
  multiplier: 1.0, constant: -20.0)
````
label의 끝나는 지점(trailing)이 button의 시작 지점(leading)에서 20만큼 왼쪽으로 떨어져있게 설정했다.    
![button_center_arrange](/img/181217/181217_img_7.png){: width="300" height="500"}


---
해당 포스트는 [부스트코스](https://www.edwith.org/boostcourse-ios)의 강의를 정리한 내용입니다.  
