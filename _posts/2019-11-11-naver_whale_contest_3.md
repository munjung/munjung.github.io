---
layout: post
title: 403 Forbidden, CORS 정책 해결하기
subtitle: 빠져나올 수 없는 서버의 늪3
tags: [Server]
---

저번 포스팅까지는 서버를 구축하고, Python을 서버에 업로드 하는 등 서버를 개발하는 단계 위주로 작성을 하였다. 하지만 이번 포스팅에서는 그보다 더 하드하고 눈물겨운 오류에 대해 정리를 하고자 한다. 총 5일의 서버 개발 기간 중, 3일을 차지한 어마어마한 녀석들이다. 서버 개발을 하면서 느낀점은 같은 문제가 생겨도 원인을 알아내고 해결하는 것이 훨씬 힘들다는 것이다. 프론트는 UI나 JS에 문제가 생기면 눈에 보인다. 하지만 서버는 눈에 보이지 않는다. 다만 로그를 뱉어낼 뿐이다. 하나의 로그를 통해 문제의 원인을 밝혀야 한다.😱😱😱 다시한번 말하지만 서버 개발자님들 정말 존경스럽다.

## - 403 Forbidden
![spc_result](/img/191110/191110_img_10.png)

정말 자주 보였던 에러였다. 403 Forbidden는 보안과 관련된 에러로, 디렉토리나 파일에 접근 권한이 없기 때문에 발생하는 오류이다. 그렇기 때문에 권한 설정이 제대로 되어 있는지 확인을 해봐야 한다.

- apache2.conf

디렉토리 태그 안에 권한 설정이 `Deny from all`인 경우가 있다. 말 그대로 모두 제한한다는 뜻이니 접근이 될 리가 없다. 아래와 같이 수정한다.
![directory_allow](/img/191110/191110_img_7.png)
`Allow from all`, `AllowOverride all`,`Order Allow, Deny`라고 설정해주면 정상적으로 접근이 되는 것을 볼 수 있다.
**만약** 특정 IP를 차단하고 싶다면, `Deny from IP`라고 작성하면 된다.

- Directory Permission

apache2.conf에 문제가 없다면 다음으로 확인해봐야 하는 부분이다. 웹서버가 실행될 때 해당 디렉토리를 읽어오는데, 만약 디렉토리에 접근이 되지 않는다면 당연히 문제가 발생할 것이다. `ls -al`명령어를 통해 디렉토리의 권한을 확인하고(하위 디렉토리도 포함) 읽기(r)권한이 없다면 `chmod` 명령어로 권한을 부여해줘야 한다. 그럼에도 불구하고 문제가 해결되지 않는다면, StackOverflow에서 더 검색해보자...😁


## - CORS Policy
이 문제에 대해서 말하기 전에 먼저, 왜 서버를 추가했는지에 대해 먼저 말하려고 한다. 프로젝트에 써야했던 API는 자그마치 3개였다. 그래서 먼저 프론트에서 Naver API를 요청했는데, 그때 발생한 이슈가 바로 크로스 도메인 정책이었다. JS에는 `동일 출처 정책(Same Origin Policy)`이라는 것이 있다. 쉽게 말하면 보안과 관련된 정책으로, 다른 도메인의 서버 URL을 요청하면 보안 문제로 간주하고 이를 차단하는 것이다. 예를 들면 www.aaa.co.kr에서 호출된 AJAX는 www.aaa.co.kr에 있는 URL만을 호출할 수 있다. 그말인 즉, `www.aaa.co.kr도메인에서는 www.bbb.com의 URL을 호출할 수 없다`는 것이다.  

그래서 따로 서버를 분리해 서버에서 API에 대한 요청을 처리하도록 하고, 프론트에서는 서버에 요청해 데이터를 받기로 했다. (그때까지만 해도 나는 크로스 도메인 이슈를 해결한 줄만 알았다...) API 완성 후, Postman에서 테스트를 했을 때 아래와 같이 결과가 매우 잘 나왔다.
![cors_1s](/img/191110/191110_img_11.png)
그래서 팀원들에게 자신만만하게 서버부부분이 완성되었음을 알리고 AJAX 테스트를 해봤는데, 크로스 도메인 정책 위반이 또 나와버렸다.🤑🤑🤑 크로스 도메인을 피하려고 서버를 만든거라 굉장히 당황스러웠다. 그래서 구글링을 해보니, CORS 문제를 해결하기 위해서는 서버 측에서 다른 도메인에서도 요청이 가능하도록 풀어줘야 한다고 했다. 그제서야 왜 서버가 있어야하고 필요한지 이해가 됐다. Apache cross domain allow만 쳐봐도 꽤나 많은 정보들이 나온다. 가장 도움을 많이 받았던 곳은 바로 [이 블로그](https://poanchen.github.io/blog/2016/11/20/how-to-enable-cross-origin-resource-sharing-on-an-apache-server)이다. 크로스 도메인 허용은 아래와 같은 코드로 해결할 수 있다.
~~~
<IfModule mod_headers.c>
	Header always set Access-Control-Allow-Origin "*"
</IfModule>
~~~
그나저나, 문제는 이 코드를 어디에 추가해야 하느냐는 거였다. 수많은 블로그를 찾아봤지만 어느 위치에 추가를 해야하는지는 명확하게 알려주는 곳을 찾기가 힘들었다. 자신의 디렉토리 태그 안에 위의 Header~ 를 넣으라는 글도 있었고, VirtualHost 부분에 넣으라는 글도 있었다. 그래서 저 코드를 이곳저곳 추가하는 상황이 생겼고, 더 큰 문제로 번졌다~~~🙆‍♀🙆‍♀🙆‍♀ 아래의 경로에 들어가서 apache2.conf 파일을 편집해야 한다.
~~~
$ /etc/apache2/apache2.conf
~~~
그리고 위에 썼던 Header 부분을 추가하면 된다. 아래와 같이 하면 된다!
~~~
<IfModule mod_headers.c>
	Header always set Access-Control-Allow-Origin "*"
</IfModule>

<Directory /home/ubuntu/searchmong/>
    	Options +ExecCGI
    	AddHandler cgi-script .cgi .py
    	Order Allow,Deny
    	Allow from all
    	AllowOverride all
    	Require all granted
    	#Header always set Access-Control-Allow-Origin "*"
</Directory>
~~~
처음에 어느 위치에 넣어야 하는지 몰라서 000-default.conf, default-ssl.conf에도 넣어줬다가 Header가 너무 많다는 오류가 나왔다... 그러니 apache2.conf에만 추가하도록!
~~~
$ sudo a2enmod headers
~~~
그 다음 헤더를 활성화 하고, 서버를 restart 하면 cross domain이 allow 된 것을 확인할 수 있다.

## - 그 외
그 외에 있었던 자잘한 문제들도 정리를 하려고 한다.  

#### `"/var/tmp/apache2.conf.swp" already exists!`  
리눅스 명령어를 몰라 텍스트 파일을 편집하지 못했을 때 자주 봤던 오류이다. INSERT하는 방법을 몰라 그냥 난타를 하고 터미널을 꺼버렸고, 다시 그 파일을 열려고 했을 때 저런 오류가 발생했다. 파일을 한번 열고 제대로 저장도 안한채로 종료했기 때문에, swap 파일이 생긴 것이었다. 저 파일을 삭제할 것인지 마저 수정할 것인지 물어보는데, 터미널에 나와있는 단축키를 눌러 본인이 원하는대로 파일을 수정하면 된다.
![remove_swap_file](/img/191110/191110_img_12.png)
아니면, 위의 명령어를 통해 swp 파일을 삭제하는 방법도 있다.

#### `unknown_open raise URLError('unknown url type: %s' % type)`
Python에서 urlretrieve을 했을 때 발생한 것으로, 9시간을 헤매고 분해서 울면서 잠든(지금 생각하면 어이없는) 에러다. 자꾸 urlOpen을 할 때 문제가 생긴다고해서 인코딩 문제인 줄 알았다. 그래서 아파치에서 utf-8로 인코딩을 해줬는데 해결되지 않았다. 그 다음으로, url로 들어오는 String에 문제가 있나 싶어 출력해봤는데 출력값에도 이상이 없었다. 도대체 뭐가 문제일까 하고 엄청 구글링을 했는데 해결 방법을 찾을 수가 없었다.😫 우선 나와 같은 문제를 겪는 사람들이 적어서 정보를 찾기가 쉽지 않았다. 다음날 다시 문제를 봤을 때, 의외의 곳에서 원인을 발견했다.
![postman_error](/img/191110/191110_img_14.png)
파이썬 문제가 아니라, 포스트맨에서 내가 작성한 value가 문제였다. String으로 이미지 url을 전달받을 것이었기 때문에 value에 "image_url"을 넣었는데, image_url이 아니라 "image_url"로 값이 들어와서 파이썬에서 '"image_url"'이 되는 것이었다. 이러한 이유로 String 파싱이 제대로 되지 않아 위와 같은 오류가 발생한 것이었다. 포스트맨에서 value에 쌍따옴표를 제거해주니 결과가 깔끔하게 나왔다...😨 정말정말정말 황당하고 힘들었던 에러였다.

#### `AWS-EC2 백그라운드 실행`
문제라기 보다는 알아둬야할 것 같아서 정리를 한다. AWS와 연동이 끊어지면 서버가 죽게되고 그렇게 되면 API 호출을 하지 못하게 된다. 하지만 1년 내내 노트북을 켜고 다닐 수는 없기에 AWS-EC2를 백그라운드에서 실행 시켜야 한다. [이 블로그](https://medium.com/@omicro03/190110-aws-ec2-%EB%B0%B1%EA%B7%B8%EB%9D%BC%EC%9A%B4%EB%93%9C-%EC%8B%A4%ED%96%89-a84381a35787)에서 잘 정리가 되어있다. `nohup` 명령어를 이용하면 간단하게 해결할 수 있다.  

#### `nginx run python cgi script`  
이것도 문제라기 보다는 알아두고 나중에 해보고 싶은 것이다. [이전 포스팅][이전 포스팅](/2019-11-10-naver_whale_contest_1/)에서 nginx를 설치했으나 여러가지 이슈로 Apache를 선택했다고 했는데 그 이유에 관한 것이다. 물론 nginx도 cgi script를 실행할 수 있었다. 그런데, 그러기 위해서는 Python에서 flask라는 라이브러리를 사용해야 한다고 했다. 그런데 이 flask는 PyCharm의 Professional 버전에서 제공해준다고 한다. 하지만 나는  Community 버전이다.(이미 프로페셔널 버전은 작년에 씀...) 그래서 Python을 개발할 수 있는 다른 툴을 사용해볼까 했지만 이미 PyCharm의 편의성에 빠진 상태였기 때문에 nginx를 버리고 Apache를 선택하게 되었다.😊 다음에 서버 개발할 수 있는 기회가 생긴다면, flask도 한번 사용해보고 싶다!

여기까지가 빠져나올 수 없는 서버의 늪 시리즈 최종이다. 짧은 시간이었지만 정말 많은 것을 배웠다. 지금까지 나한테 서버는 어렵게만 느껴지는 알 수 없는 미지의 영역이었는데 이번 기회로 경험할 수 있어서 뜻깊었다. 7일의 개발 기간 중, 5일은 서버에 쓰고 나머지 2일은 프론트를 손봤다. 다음 포스팅부터는 프론트에 대해서 간단하게 정리하려고 한다.
