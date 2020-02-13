---
layout: post
title: 자바 내부 클래스 정리
subtitle: 위치에 따른 4가지 형태
tags: [Java]
---

요즘 회사에서 `자발적 공부`를 명목으로 시킨 온라인 강의를 수강중이다. 자바 입문부터 보고 있는데 강의가 상당히 잘 구성되어 있다. 오랜만에 배우는 개념을 다시 한번 익힐 수 있고, 잘 알지 못했던 부분을 새로 배울 수 있어서 재밌게 듣고 있다. 그래서 이번에는 그 중 하나인 내부 클래스에 대해 정리하고자 한다. 👋


## 내부 클래스(Inner Class)?
클래스 안에 선언된 클래스  
### 내부 클래스의 장점
- 내부 클래스에서 외부 클래스의 멤버들을 쉽게 접근할 수 있다.
- 코드의 복잡성을 줄일 수 있다.

내부 클래스는 `어느 위치에 선언`하느냐에 따라 4가지의 형태가 있을 수 있다.

## 클래스 안에 인스턴스 변수
필드를 선언하는 위치에 선언되는 경우로 보통 `중첩 클래스` 혹은 `인스턴스 클래스`라고 한다.
![inner_test_1](/img/200212/200212_img_1.png)
내부에 있는 Test 객체를 생성하기 위해서는 InnerClass 객체를 만든 후에 `InnerClass.Test it = ic.new Test();`와 같은 방법으로 Test 객체를 생성한다.

## 내부 클래스가 static으로 정의
`정적 중첩 클래스` 혹은 `static 클래스`라고 한다.  
![inner_test_2](/img/200212/200212_img_2.png)
필드를 선언할 때 static으로 선언한 것과 같다. 따라서 InnerClass의 객체를 따로 생성할 필요 없이 `new InnerClass.Test();`로 객체를 생성할 수 있다.

## 메소드 안에 클래스를 선언
`지역 중첩 클래스` 혹은 `지역 클래스`라고 한다.
![inner_test_3](/img/200212/200212_img_3.png)
메소드 안에서 해당 클래스를 이용할 수 있다.

## 익명 클래스
`익명 중첩 클래스`는 익명 클래스라고 보통 말하며, 내부 클래스이기도 하다.
~~~
public abstract class Action {
	public abstract void exec();
}

public class ActionExam {
	public static void main(String[] args) {

    /* Action extends */
	 	Action action = new MyAction();
		action.exec();

		/*MyAction을 쓰지 않고 Action을 상속받는 익명 클래스 생성*/
		Action ac = new Action() {

			@Override
			public void exec() {
				System.out.println("inner class test");

			}

		};

		ac.exec();
	}
}
~~~
- 생성자 다음에 중괄호({ })가 나오면, 해당 생성자 이름의 클래스를 상속받는 이름 없는 객체를 만든다는 뜻  
- 괄호 안에는 메서드를 추가하거나 구현이 가능
- 익명 클래스를 만드는 이유는 해당 클래스를 상속받는 클래스를 만들 필요가 없는 경우
- 상속 받는 클래스가 해당 클래스에서만 사용되고 다른 클래스에서는 사용되지 않는 경우
