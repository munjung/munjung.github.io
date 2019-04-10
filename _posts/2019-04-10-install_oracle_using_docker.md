---
layout: post
title: Docker를 이용하여 Oracle 11g 설치하기
subtitle: 맥북에서 Oracle 설치 Sql Developer 연동
---

요즘 학교에서 Spring 강의를 듣고 있다. 스프링과 Oracle Database 연동하는 것을 배운 뒤 맥북에도 시도하려고 했으나 맥에서는 오라클을 사용할 수 없단다... 그런데 찾아보니 Docker를 이용하면 맥에서도 오라클을 쓸 수 있다길래 시도해봤다. 생각보다 쉽고 편했다. 그래서 정리해봤다. (중간 삽질도 있음)

## Docker 설치하기  

1.
[Docker 사이트](https://www.docker.com/get-started)에 접속해서 **`Stable`** 버전 선택

![select_stable](/img/190410/190410_img_1.png){: width="600" height="300"}    

2.
터미널에서 명령어 입력하기  

몇몇 블로그에서 **docker pull wnameless/oracle-xe-11g** 명령어를 입력해서 오라클 이미지를 다운받으라고 했지만 오류가 발생하고 말았다. 알고보니 **현재 해당 repository는 막혀있었다.**

![access_denied](/img/190410/190410_img_2.png){:height="200"}      

다른 것들을 찾아보고 두번째 이미지로 선택했다.  
 `docker pull jaspeen/oracle-11g` 명령어 입력하면 설치가 잘된다.  

![select_jaspeen](/img/190410/190410_img_3.png){:height="200"}        


다운받은 이미지를 실행한다.
![run_image](/img/190410/190410_img_4.png)  



## Sql Developer 설치하기
1.
[오라클 사이트](https://www.oracle.com/technetwork/developer-tools/sql-developer/downloads/index.html)에서 다운로드 (이때 `Java 8` 버전 이상이어야 한다.)  
[사용자: system, 비밀번호: oracle, 포트: 1521]
![db_connect](/img/190410/190410_img_5.png){: width="500" height="300"}    
그런데 `locale not recognized`에러가 난다.  

2.
설정 파일 편집  
**맥 os 시에라** 버전부터 생기는 오류라고한다.  
실행중인 SqlDeveloper를 **Control 누르고 클릭 - 패키지 내용 보기**

`Contents/Resources/sqldeveloper/sqldeveloper/bin/sqldeveloper.conf`에 들어가 제일 하단에 두 줄 추가  
![setting](/img/190410/190410_img_6.png){: width="600" height="100"}    
Sql Developer를 다시 실행하면 오류는 사라진다!
