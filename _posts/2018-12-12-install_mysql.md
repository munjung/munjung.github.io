---
layout: post
title: 맥북에 mysql 설치하기
subtitle: Homebrew 이용하기
tags: [mysql]
---
설치하는 방법은 두가지가 있다.  

1. [공식페이지](https://dev.mysql.com/downloads/mysql/)에서 dmg 파일 다운로드
2. 터미널에서 brew로 설치  

나는 `brew`를 이용하여 설치하려고 한다.  
만약 [Homebrew](https://brew.sh/index_ko) 미설치 시 터미널에서 아래 명령어를 입력하면 된다.

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

> **MySQL 설치하기**  


```
$ brew install mysql
```

> **MySQL 설정하기**


```text
$ mysql.server start
$ mysql_secure_installation
```

여기까지 하고나면 몇가지 질문들이 나오는데 자신에게 맞도록 적절히 설정하면 된다.

- 비밀번호를 복잡하게 설정 할 것인가? `no`  

```
VALIDATE PASSWORD COMPONENT can be used to test passwords and improve security.
It checks the strength of password and allows the users to set only those passwords which are secure enough.    
Would you like to setup VALIDATE PASSWORD component?  

Press y|Y for Yes, any other key for No:
```

- 비밀번호 입력하기  

```
Please set the password for root here.
New password:
Re-enter new password:
```
- 익명사용자를 삭제할 것인가? `yes`  

`yes`하면 `-u` 가 필수, `no` 하면 `$ mysql`만으로도 접속 가능  


```
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for testing, and to make the installation go a bit smoother. You should remove them before moving into a production environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) :
```

- 원격로그인을 허용하지 않을 것인가? `no`  

```
Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) :
```
- 기본으로 있는 test DB를 삭제할 것인가? `no`  

```
By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,and should be removed before moving into a production environment.

Remove test database and access to it? (Press y|Y for Yes, any other key for No) :
```
- 수정사항이 있는가?  `no`  

```
Reload privilege tables now? (Press y|Y for Yes, any other key for No) :
```

>**MySQL 시작하기**  

```text
$ mysql -u root -p
```
