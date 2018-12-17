---
layout: post
title: 오토레이아웃(AutoLayout) 정리-1
subtitle: 코드를 이용하여 오토레이아웃 구현하기
tags: [Xcode, swift]
---

## 오토레이아웃?
디바이스 사이즈에 구애받지 않고 `시각적으로 동일한 화면`을 구현해야할 때 필요한 것으로 `view의 제약 사항`을 바탕으로 모든 view의 크기와 위치를 `동적`으로 계산한다.  
### 오토레이아웃을 구현하는 방법
- 코드를 이용하여 구현
- 인터페이스 빌더에서 구현

먼저 `코드`를 이용하여 오토레이아웃을 구현하는 방법에 대해 알아보자.
오토레이아웃을 구현하기 위해서는 코딩으로 제약(Constraint)을 만들어야 하는데 이때 `앵커(Anchor)`를 사용하게 된다.

- 메인 스토리보드에 button과 label을 추가하고 ViewController.swift와 연결해준다.
![connect_controller](/img/181216/181216_img_1.png)  


- viewDidLoad 메서드 안에 코드를 작성한다.
![view_did_load](/img/181216/181216_img_2.png)  


중앙 앵커(centerXAnchor, centerYAnchor)를 이용하여 view의 중앙에 button을 위치시켰다. 또한 이런 제약을 적용하기 위해서 isActive의 값으로 true를 설정해준다.  

```
translatesAutoresizingMaskIntoConstraints = false
```
오토레이아웃이 도입되기 전에 사용했던 것으로 오토레이아웃을 사용하게 되면 기존 오토리사이징 마스크가 가지고 있던 제약조건으로 충돌하게 될 가능성이 있기 때문에 false로 지정해준다. (참고로 인터페이스 빌더에서 오토레이아웃을 적용한 경우에는 자동으로 값이 false로 설정된다고 한다.)

- 마찬가지로 label에 대해서도 코드를 작성한다.
![view_did_load](/img/181216/181216_img_3.png)  

위에서 작성한 코드와 비슷하지만 제약사항이 추가된걸 볼 수 있다.
```
label.bottomAnchor.constraint(equalTo: button.topAnchor, constant: -20)
```

**"label의 하단 앵커(bottomAnchor)를 button의 상단 앵커(button.topAnchor)로 부터 20만큼의 거리를 둔다."** 라는 뜻이다.
`상단 앵커`를 기준으로 `위`는 거리의 부호가 `-`이다. 결과화면은 이렇게 나온다.  
![view_did_load](/img/181216/181216_img_4.png){: width="300" height="500"}  

따라서 만약 아래와 같이 수정하게 된다면 label의 위치가 button 밑에 보여지는 것을 확인할 수 있다.
```
label.topAnchor.constraint(equalTo: button.bottomAnchor, constant: 20)
```



`NSLayoutConstraint`를 이용하여 오토레이아웃을 구현하는 자세한 방법은 다음 포스팅에서 다루도록 하겠다.  

---
해당 포스트는 [부스트캠프](https://www.edwith.org/boostcourse-ios)의 강의를 정리한 내용입니다.  
