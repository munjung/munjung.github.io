---
layout: post
title: 스프링 No mapping found for HTTP request 해결하기
subtitle: Update Project의 중요성
tags: [Spring]
---

교수님께서 특정 URL로 요청을 보내면 로그나 결과를 출력하면 되는 매우 매우 간단한 과제를 주셨다. 과제를 하던 중 뜻밖의 문제가 생겨서 고생하다가 겨우 해결했다. 지금 생각해보면 너무 황당하다.ㅎㅎ 하지만 열심히 삽질한 덕분에 스프링의 기본 설정에 대해 배울 수 있었다. 그래서 발생한 문제와 해결과정을 기록하고자 한다.

## - 문제

![eclipse_code](/img/190411/190411_img_1.png)  

이 상태에서 localhost:8080/hw01/kmj/getName로 URL을 요청했더니 오류가 발생했다.
> WARN : org.springframework.web.servlet.PageNotFound - No mapping found for HTTP request with URI [/hw01/kmj] in DispatcherServlet with name 'appServlet'

###### (페이지는 없는 상태였다.)
맵핑문제가 왜 발생하는지 궁금해서 구글에 검색하며 여러 블로그들의 포스팅을 봤다. 원인은 `컨트롤러에서 URL을 찾지 못해서` 발생하는 문제라고 한다. 또한 의심되는 **에러 원인** 으로 몇 가지 정도 있다고 해서 하나씩 살펴보며 원인을 찾아봤다.


## 1. web.xml
web.xml에서 맵핑이 되어 있는지, DispatcherServlet 선언이 제대로 되어 있는지 확인해야 한다. 이 부분은 기본 설정이었기 때문에 별다른 문제가 보이지 않았다.
~~~
<servlet>
	<servlet-name>appServlet</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
	</init-param>
	<load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
	<servlet-name>appServlet</servlet-name>
	<url-pattern>/</url-pattern>
</servlet-mapping>
~~~

추가로 stackoverflow에서는 web.xml에 아래의 코드를 추가하라고 해서 해봤으나 그대로였다!
~~~
<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
~~~


## 2. Controller
컨트롤러에서 url 매핑을 제대로 했는지 확인해야 한다. 당시 tomcat의 기본 path는 /hw01로 설정했다.  
(path 설정: Servers-Modules)
![port_setting](/img/190411/190411_img_2.png){: width="600" height="120"}  

클래스 상단에 컨트롤러 선언(`@Controller`), 메서드 호출 방식(`@GetMapping` or `@PostMapping`), 메서드 호출 경로(`"/@@@"`)가 제대로 정의되어 있는지 확인해야 한다. 이 부분도 문제가 보이지 않았다.

~~~
@Controller
@RequestMapping("/kmj/*")
@Log4j
public class Hw01Controller {

	//1
	@GetMapping("/getName")
	public void getName() {
		log.info("KANG MOON JUNG");
	}
}
~~~


## 3.servlet-context.xml
servlet-context.xml에 컨트롤러가 제대로 등록되어있는지 확인해야 한다. `<context:component-scan base-package="컨트롤러의 패키지 경로"/>`를 입력해야하며 등록이 된 컨트롤러는 파일 좌측에 S표시가 뜨게되며 그래프로도 확인할 수 있다.  
(Graph 확인: servlet-context.xml - Beans Graph)
![port_setting](/img/190411/190411_img_3.png){: width="600" height="250"}  

로그를 보니 프로젝트 최상단 패키지로 지정했던 "org.zerock.controller.###"로만 접근하길래 현재 Hw01 클래스가 있는 패키지 이름에 오타가 있었나 확인했다. (없어...없다고)
~~~
<context:component-scan base-package="org.zerock.controller"/>
<context:component-scan base-package="moonjungkang.controller"/>
~~~
찾아보니까 `<context:component-scan base-package="a,b,c"/>` 이런식으로 써야한단다. component-scan 태그는 하나만 써야 하는구나 생각했다. 그러나 여전했다.ㅎㅎ
~~~
<context:component-scan base-package="org.zerock.controller, moonjungkang.controller"/>
~~~  

정말 도저히 원인을 모르겠어서 멘붕이 왔다. 혹시나 라이브러리나 pom.xml 설정이 잘못됐나 싶어 몇번이나 확인했다. 또 위에서 언급한 3가지를 다시 체크하기, 프로젝트 다시 생성, 톰캣 재연동, 그 외의 파일 설정 변경 등등 다양하게 삽질을 했다.  

하루종일 삽질한 결과 원인은 `Properties-Maven-Update Project`에 있었다. 프로젝트 업데이트를 하니 세상 잘 돌아간다~~~ 혹시나해서 3번에서 변경한 component-scan 태그를 원래대로 변경했더니 그래도 잘 돌아갔다. 즉, component-scan 태그는 여러개 써도 되고 하나의 태그에 여러 패키지들을 넣어도 된다는 것이다.

약간 허무하기도 했지만 덕분에 Spring 기본 설정과 작동 방식을 배울 수 있었던 좋은 경험이었다.👊👊👊
