---
layout: post
title: Python CGI 사용, Apache로 파일 업로드하기
subtitle: 빠져나올 수 없는 서버의 늪2
tags: [Server, Python]
---

[이전 포스팅](/2019-11-10-naver_whale_contest_1/)에서는 AWS를 구축하고 Apache를 설치하는 방법에 대해 작성하였다. 이번엔 Apache가 실행할 수 있는 Python 코드를 작성하고 실제 서버에 올리는 것에 대해 정리해보겠다. REST API를 호출하기 위해서는 각 사이트마다 apiKey를 발급받아야 하는데 그 과정은 생략할 것이다. 왜냐하면 앞으로 해야할 것은 시작에 불과하기 때문이다.😁 서버에서 Python을 실행하기 위해서는 CGI를 사용해야 한다고한다. 그러나 CGI가 무엇인지 몰랐기 때문에, 해당 개념에 대해서 먼저 알아봤다.

## - CGI
CGI는 공용 게이트웨이 인터페이스(Common Gateway Interface)로 서버와 클라이언트 사이에서 정보를 주고받는 방법이나 규약을 뜻한다. 쉽게 말하면,  프론트가 서버로 데이터를 요청하면 CGI를 통해서 데이터를 처리하는 것이다. 또한 `서버와 클라이언트 간의 입출력을 정의한 표준`이라고 한다. 그래서 이러한 표준에 의해 만들어진 것을 CGI 스크립트라고 부르고, 어떤 프로그래밍 언어로도 CGI 스크립트를 작성할 수 있다. 더 자세한 설명은 [여기](https://sfeg.tistory.com/196)에서 볼 수 있다.  
> CGI Script를 서버에 업로드하면, Request시 Python이 실행되는구나!

라고 이해했다. 그래서 Python으로 코드를 작성했다. `import cgi`가 가능하다. 역시 파이썬 갓갓갓!  코드는 아래와 같다. 자세한 코드는 생략한다.

~~~
// Python이 설치된 경로를 적어야 한다.
#!/usr/bin/env python3  
#-*- coding: utf-8 -*-

import sys
import codecs
import cgi
import cgitb

// 오류가 발생했을 시, 오류에 대한 정보를 볼 수 있다.
cgitb.enable()

//한글을 출력할 수 있게 한다. 없으면 골치 아프다.
sys.stdout = codecs.getwriter("utf-8")(sys.stdout.detach())

//CGI 스크립트를 읽을 때, HTTP 헤더를 설정해줘야 한다.
print("Content-type: text/html; charset=utf-8\r\n")

import os
import requests
import ssl

params = cgi.FieldStorage()
image_url = params["image_url"].value
image_type = params["image_type"].value

print(output)
~~~
### 필수!
서버에도 Pyhton을 설치해야 한다. 위의 코드는 테스트용으로 PyCharm으로 작성한 것이기 때문에, 서버에서 곧장 실행이 불가능하다.  import한 라이브러 또한 pip을 통해 리눅스에 설치를 해야한다. 나는 이 사실도 모르고 바로 서버에서 실행을 시켰다. PyCharm에서는 결과가 제대로 나오는데 왜 서버에서는 동작이 안되는지 이해가 안되서 3시간을 멘붕에 빠졌었다...😉 (결국, vi 무한 반복으로 리눅스에서 한줄씩 테스트해보는 사태가 발생했다.ㅎ)

## - 서버에 파일 업로드
`scp`를 이용하면 된다. scp는 secure copy (remote file copy program)의 줄임말로 ssh를 이용해 네트워크로 연결된 호스트간에 파일을 주고 받는 명령어로, 서버에 파일을 업로드 할 수 있고 반대로 서버에 있는 파일을 다운로드할 수 있다. 나는 .pem 인증서를 이용해서 서버에 Python 파일을 전송했다. .pem과 Python 파일명을 입력할 때는, 해당 파일이 존재하는 경로도 같이 넣어줘야 한다.

~~~
$ scp -i searchmong.pem ocr.py ubuntu@ec2-15-164-97-135.ap-northeast-2.compute.amazonaws.com: "directory name"
~~~

그렇게 파일을 전송하고 리눅스를 통해 디렉토리에 가보면, ocr.py가 업로드된 것을 확인할 수 있다!
![spc_result](/img/191110/191110_img_9.png)

## - CGI Script 실행
Python 파일을 업로드 했으면, 서버에서 Python의 CGI Scrpit를 실행하도록 설정해야 한다. 저번 포스팅에서 디렉토리 태그를 추가했던 apache2.conf 파일을 수정해줘야 한다.
~~~
$ /etc/apache2/apache2.conf
~~~
Directory 태그안에 아래와 같이 설정해준다. cgi-script를 실행하라는 헤더와 옵션을 추가하면 된다. 그 아래의 코드는 다음에 설명하겠다.
![add_directory](/img/191110/191110_img_7.png)

이번 포스팅은 여기까지다. 다음 포스팅에서는 403 Forbidden error 해결과, CORS 정책 해결(+Postman)에 대해 정리하도록 하겠다. 
