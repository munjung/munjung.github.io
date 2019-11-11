---
layout: post
title: AWS 구축 및 Apache 설치
subtitle: 빠져나올 수 없는 서버의 늪
tags: [Server, Linux, Apache]
---

오랜만에 하고싶은 공모전을 찾았다. 바로 🐳네이버 웨일 콘테스트🐳이다. 네이버 브라우저인 웨일에 들어가는 확장앱을 만드는 공모전이었는데, 확장앱 개발이 궁금해서(상금도 두둑해서🎉) 학교 동기들을 모아 팀을 꾸렸다. 주제는 이미지 검색 서비스로, 쉽게 말하면 네이버 앱에 있는 이미지 검색 서비스를 웹에서도 사용할 수 있게 한 것이었다. 더불어 OCR 기능도 추가하기로 했다. 주어진 개발 기간이 일주일 정도였기 때문에, 프론트에 자신이 없었던 나는 서버를 개발하기로 하였다. 이로써 인생 첫 서버개발을 하게 되었다. ~~(울면서 개발한 5일)~~ 결론부터 말하면 확장앱 제출을 완료했다! 🤸‍♀ 하지만 그동안 여러가지의 어려움도 있었고 배운점도 많기 때문에 기록을 남기려고 한다.

## - AWS 구축

AWS 구축 방법은 많은 블로그에서 설명해주고 있다. 그 중 나는 [이 블로그](https://m.blog.naver.com/wool613/221286931692)를 보고 많은 도움을 받았다. 1년 동안은 무료로 EC2 인스턴스를 사용할 수 있다. 또한 나는 익숙한 우분투 서버를 선택했다. `.pem`파일은 꼭 잘 저장하도록 하자! 나중에 SSH로 AWS를 연결할 때 필요하다.

![aws_connect](/img/191110/191110_img_1.png)

.pem 파일을 설치하고 인스턴스 시작을 클릭했을 때, 위의 사진처럼 나온다면 성공이다! 웹 서버를 구축한 뒤, IPv4 퍼블릭 IP를 웹 브라우저에 입력하면 연결되는 것을 볼 수 있다. 저기서 `연결`을 클릭하면 인스턴스를 연결할 수 있는 방법이 나오게 된다.

![aws_connect2](/img/191110/191110_img_2.png)  

그러면 터미널을 열고 .pem 파일이 있는 곳으로 이동한 뒤(나는 desktop) 예시에 나와있는 명령어를 그대로 복사하면 된다.
(디렉토리로 이동하지 않아도 .pem 파일의 설치경로를 적어줘도 된다.)

~~~
$ cd desktop  
$ ssh -i "searchmong.pem" ubuntu@ec2-15-164-97-135.ap-northeast-2.compute.amazonaws.com
~~~

![aws_connect3](/img/191110/191110_img_3.png)  
위의 사진처럼 뜬다면 연결 성공~~~!

## - Apache 설치

사실 나는 Apache를 설치하기 전, nginx를 사용했었다. 그러나 여러가지 문제로(추후 포스팅에서 작성) Apache를 선택하게 되었다. nginx를 설치하는 방법은 [유튜브](https://youtu.be/28ioY4vgC9I)를 참고했다.
Apache를 설치하기 위해서는 SSH에 접속해야 한다. 그 뒤, 아래의 명렁어를 입력한다.
~~~
sudo apt-get install apache2
~~~
만약, 오류가 난다면 apt-get을 업데이트 해줘야 한다. 그런 다음에 설치를 하면 잘 될 것이다.
~~~
sudo apt-get update
~~~
아파치 버전을 확인해보자!
![check_apache_version](/img/191110/191110_img_4.png)

*아파치 구동 명렁어*
~~~
sudo service apache2 restart // 재시작
sudo service apache2 start // 시작
sudo service apache2 stop // 중단
~~~

## - Apache root 변경

나는 Python을 이용해서 kakao vision api와 naver papago api, research api를 요청하고 결과를 출력해주기로 했다. 그래서 Apache가 Python을 실행할 수 있어야 했고, 그러기 위해선 Python 파일을 서버에 올려야 했다. Python 파일을 올린 디렉토리는 따로 만들어야 했기에, Apache의 root 경로를 Python 파일이 있는 디렉토리로 변경해줘야 했다. Apache 웹 서버의 루트 경로를 설정하는 곳은 `000-default.conf`인데 해당 파일은 아래의 주소에 위치한다. `vi` 명령어를 이용해서 000-default.conf 파일을 수정해주자! (리눅스 명령어도 몰라서 한참 헤맸다 ^^..)
~~~
$ /etc/apache2/sites-available
~~~
![change_default_conf](/img/191110/191110_img_5.png)
그러면 아래와 같은 파일이 나타날 것이다. 기본 root 경로는 /var/www/html로 설정되어 있다. 이 부분을 주석처리 해주고, 원하는 경로로 변경해준다.
![change_default_conf2](/img/191110/191110_img_6.png)

### + 리눅스에서 파일 안의 텍스트를 수정하는 방법  
**i 입력**: 아래에 `INSERT`라고 뜰 것이다. 수정이 가능한 상태이다.  
**esc 클릭**: 수정을 완료했다는 것이다. 다시 수정하고 싶다면 i를 입력한다.  
**:wq 입력**: 수정 한 내용을 저장하고 마친다. (**:q** :저장 없이 마치는 것, **:!q** :강제 종료)  

### 만약, readOnly 경고가 뜬다면!  
파일에 쓰기 권한이 없는 것이다. 일단 파일을 빠져나오고, 아래와 같은 명령어를 통해 쓰기 권한을 부여해준다. (나는 그냥 모든 권한을 부여했다.) 그러면 파일 수정이 가능해질 것이다. 각각의 파일에 어떤 권한이 부여되어있는지 알고싶다면, `ls -al`를 입력하면 된다.
~~~
sudo chmod 777 000-default.conf
~~~

## 실행 파일의 디렉토리 추가
나는 /home/ubuntu로 root 경로를 지정해 줬으니, Python파일이 있어야 할 디렉토리 경로 또한 생성해 줘야 한다. 그것은 `apache2.conf`에 정의되어 있다. 경로는 아래와 같다.
~~~
$ /etc/apache2/apache2.conf
~~~
마찬 가지로 vi 명령어를 이용해서 수정을 해준다. 디렉토리 태그를 추가하고 경로를 맞춰준다. 나는 /home/ubuntu에 searchmong 이라는 디렉토리 안에 Python 파일을 업로드 할 것이기 때문에 아래와 같이 설정해줬다.
![add_directory](/img/191110/191110_img_7.png)
여기까지 완료했으면, searchmong 이라는 디렉토리를 실제 생성해주자! cd /home/ubuntu 로 해당 경로로 이동한 뒤, mkdir 명령어를 통해 폴더를 만들어 준다.
~~~
mkdir searchmong
~~~
그리고 확인해보자!
![add_directory2](/img/191110/191110_img_8.png)


여기까지가 AWS 구축과 Apache 설치 및 디렉토리 변경 방법이다. 지금와서 과정을 보면 쉽고 간단해보이지만, 서버를 처음하는 입장에선 모든게 낯설고 어려웠다.😵😵😵 심지어 리눅스 명령어도 몰랐기 때문에 conf 파일을 수정할 때 꽤나 애먹었다.....😥 서버 개발자님들 진심으로 존경스럽다..💓  
다음 포스팅에서는 scp를 이용해서 서버에 파일을 업로드 하는 것과, Python으로 CGI를 사용하기를 정리해보겠다.
