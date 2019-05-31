---
layout: post
title: 스프링 Mybatis PersistenceException 오류 해결하기
subtitle: org.mybatis.spring.MyBatisSystemException
tags: [Spring]
---

저번주에 수업을 한번 못갔었는데 그 사이에 교수님이 폭풍진도를 나가셨다. 원래 수업을 하실 때도 책을 보고 그대로 했기 때문에
별다른 걱정은 없었다. 그래서 그냥 아무생각없이 프로젝트를 실행했는데 난생 처음보는 오류가 발생했다. 구글에 쳐보면 빠르게 해결할 수 있을거라는 생각에 안심했는데 그게 아니었다. 아직 노트북에서는 중간고사 이후의 스프링을 따로 실행한 적이 없다. 그래서 학교에서 수업을 하는 주2일 1시간 15분을 열심히 고치는데 시간을 투자했다. 그사이에 교수님은 한단원을 더 나가셨다. 교수님은 챕터 11인데, 나는 9에서 머물러 있었다. 이 오류를 고치는데 2주는 걸린 것 같다. 강의시간은 촉박하고 프로젝트가 돌아가지 않는다고 ~~물어봐도 교수님도 모르시니~~ 너무 힘들었다. 결국에는 고쳤으니까 됐어! 역시 스프링은 삽질이 제맛인 것 같다. 삽질 과정을 살펴보면 아래와 같다.

## - 문제

![eclipse_code1](/img/190512/190512_img_1.PNG)  

처음에 발생한 오류는 바로 이것이었다. 이 오류의 원인은 다음과 같다고 한다.
1. `Mapper 인터페이스와 맵핑되는 xml파일에 오타`가 있는 경우  
2. classpath에 경로가 잘못된 경우
3. xml이 저장될 경로는 잘못 생성한 경우

대부분의 경우에는 1번이 원인이라고 했다. 그 대표적인 예로는 Mapper interface에서 선언한 메서드의 이름과 xml 파일에서의 id가 불일치하는 경우이다. 또한 xml에서의 id는 항상 소문자로 시작해야 한다. 눈을 부릅뜨고 오타를 찾고 있었는데 진심으로 오타가 보이지 않았다. 서버를 재실행하고 리셋했지마 결과는 그대로였다. 그러다가 오류메세지에 언뜻 Hikari 관련된 메세지가 있어서 그 부분을 중점으로 구글에 검색하게되었다. 그런데 어떤분이 기존의 root_context.xml에 추가하라고 하면서 코드를 남겨주셨다.

## - root_context.xml
~~~
<bean id="hikariConfig" class="com.zaxxer.hikari.HikariConfig">
		<property name="driverClassName"
			value="net.sf.log4jdbc.sql.jdbcapi.DriverSpy">
		</property>		
		<property name="jdbcUrl"
			value="jdbc:log4jdbc:oracle:thin:@localhost:1521:ORCL">
		</property>
		<property name="username" value="id"></property>
		<property name="password" value="pw"></property>
	</bean>


	<bean id="dataSource" class="com.zaxxer.hikari.HikariDataSource"
		destroy-method="close">
		<!-- <constructor-arg ref="hikariConfig"/>  -->
		 <property name="driverClassName" value="net.sf.log4jdbc.sql.jdbcapi.DriverSpy"></property>
		 <property name="jdbcUrl" value="jdbc:log4jdbc:oracle:thin:@localhost:1521:ORCL"></property>
		 <property name="username" value="id"></property>
		 <property name="password" value="pw"></property>
	</bean>

	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
	</bean>


	<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
    <constructor-arg index="0" ref="sqlSessionFactory"/>
	</bean>



	<bean id="sqlSessionFactory"
		class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource"/>
	</bean>
~~~

내 기억상 추가된 부분은 아래부터 세개의 bean들과, 두번째 dataSource bean 안에 있는 property 내용이다.
일단 그렇게 수정을 하고 돌려보니 잘 돌아간다!


![eclipse_code2](/img/190512/190512_img_3.PNG)

저번 프로젝트와 마찬가지로 tomcat의 기본 path를 / 로 바꿔주고 실행했다!  
여기까지는 잘되서 너무 신났다.

![eclipse_code3](/img/190512/190512_img_4.PNG)

메서드 호출 방식을 지정하고, board/list로 url을 요청했을 때 동작을 지정했기 때문에 localhost:8080/board/list를 호출했다. 그런데 Mybatis 오류가 발생한 것이다.

![eclipse_code4](/img/190512/190512_img_5.PNG)

구글에 검색해보니 `오류 메세지에 ###이 있는 것은 xml에 오타`가 있기 때문이라고 한다. 그래서 또 오타문제인가 싶어서 프로젝트를 세세하게 보면서 오타를 찾아봤다. 몇번을 인터페이스와 xml을 비교해보고 그 밖에도 기존의 Java 코드들도 확인해봤는데 그대로였다. 포기할까 싶었는데 오류메세지를 읽어보고 문득 JDBC라는 문구가 눈에 띄었다. 에러 메세지에는 데이터베이스 오류라고 쓰여있던 것이다. 그래서 검색해서 확인해보니 `데이터베이스 서버와 연결이 되지 않아 생기는 오류`라고했다. 그것을 알게된 순간 JDBC 연결과정에서 어떤 문제가 있다는 것을 깨닫게 되었다.

## - JDBC
그 전에 JDBC에 대해서 정확하게 알고자 했다. [해당 사이트](https://dyjung.tistory.com/50)에서 알기쉽게 배울 수 있었는데, JDBC는 Java DataBase Connectivity의 약자로서 자바 프로그램 내에서
DB와 관련된 작업을 처리할 수 있도록 도와주는 일을 한다고 한다. Java에서 데이터베이스를 사용할 때에는 JDBC API를 이용하여 프로그래밍을 하는데 Java는 DBMS 종류에 상관없이 하나의 JDBC API를 사용하여 데이터베이스 작업을 처리할 수 있기 때문에 알아두면 어떤 DBMS든 작업을 처리할 수 있게 된다고 한다.


![eclipse_code5](/img/190512/190512_img_8.PNG)
oracle 데이터베이스의 jdbc drover는 11g까지 maven으로 지원이 되지 않는다고 한다. 결국 jar 파일을 직접 추가해줘야 하는 것이다. ojdbc8 경로는 sqldeveloper/jdbc/lib에 위치한다. ojdbc8는 Java Build Path에 경로를 추가해줘야 한다.
![eclipse_code6](/img/190512/190512_img_9.PNG)  

그리고 또 추가해줘야하는 곳이 있었으니...바로

## - Deployment Assembly

![eclipse_code7](/img/190512/190512_img_10.PNG)  

여기였다는 사실! 나는 build path에만 추가하면 문제가 없는 줄 알았는데 아니었다.

![eclipse_code8](/img/190512/190512_img_12.PNG)
> Add - Java Build Path Entries - jar 추가 - Finish

을 하고 다시 실행을 해보니

![eclipse_code9](/img/190512/190512_img_13.PNG)

2주동안 받은 스트레스 한번에 해소되는 느낌이었다! 흑흑
도대체 Deployment Assembly가 뭐길래 그런가 싶어서 검색을 안 할수가 없었다.
다행히 이해되기 쉽게 정리해주신 분이 있었다. 바로 [이분 사이트](https://lng1982.tistory.com/115)이다.
드디어 진도를 따라잡을 수 있게 되서 너무 신났다. Deployment Assembly는 영원히 잊지 못할 것 같다!!!
