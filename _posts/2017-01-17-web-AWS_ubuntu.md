---
title: AWS EC2 Ubuntu deploy하기
categories:
- web
tags:
- aws
- ec2
- deploy
- ruby
---

이 글은 AWS EC2에 rails 서비스를 deploy하는 방법이다.

## 1.AWS 인스턴스 생성
1. Services - EC2
2. launch instance
3. Ubuntu select
4. t2.micro review and launch
5. edit security group
6. Add Rule -> HTTP -> review and launch
7. launch
8. create a new key pair - key pair name 입력후 다운로드
9. launch instance
10. view instance
11. instance에 이름 정해주기

## 2. pem키 설정

AWS가 내가 가진 키가 유효한지 확인하기 위해서는 권한을 변경해줘야한다.

`chmod` : 리눅스 접근 권한 변경 명령어 
* chmod 400 Downloads/konkuk.pem
> * chmod 400 pem키 위치
> 
> * `400` : 절대모드로 읽기만 가능

	[참조사이트 바로가기](http://blog.naver.com/kdi0373/220513288265)

`SSH`(=secure shell)

**다른 컴퓨터**에 **원격**으로 **안전**하게 **접속**하는 **프로그램**
* ssh -i pem키 위치 ubuntu@Public IP
> 
> * `-i` identity-file의 약자로 옵션으로 id파일을 넘긴다는 뜻이다. 그래서 key파일을 써주는 것이다.
> * 새로 생성한 EC2인스턴스에 접속하기위해선 그 컴퓨터의 주소(IP)가 필요하다.
> * 즉, ssh 방식을 통해 인터넷 어딘가 중에서 IP주소 에 있는 컴퓨터에 ubuntu user로 접속할 것이다. -i 옵션으로 konkuk.pem 열쇠를 넘길거라는 뜻이다.


## 3. rvm 설치([rvm.io](http://rvm.io)접속)
 1. `sudo apt-get update`
 > sudo(substitute user do):관리자(root)가 아닌 계정일 때 입력하는 명령어 줄을 관리자 권한으로 실행
 2. `gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3`
 
 3. `\curl -sSL https://get.rvm.io | bash -s stable`
 
 4. `source ~/.rvm/scripts/rvm`
 
 5. `rvm requirements` 

```
rvmsudo /usr/bin/apt-get install build-essential openssl libreadline6 libreadline6-dev curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev automake libtool bison subversion
```

## 4. git 설치 후 clone

우분투는 git이 설치되어있다.
 
 `git clone https://github.com/...`

## 4. ruby 설치
* `rvm install 2.3.3`
* `rvm use 2.3.3 --default`
* `gem install bundler`
 > bundle install 명령어를 가능하게 해주는 gem

## 5. ruby gem 설치
`rvm rubygems current`

## 6. Rails 설치
`gem install rails`


이때 clone한 repository로 이동.

## 7. passenger 설치
 `gem install passenger`
>* Nginx서버를 사용하기 위해 필요한 gem
>* Nginx : 웹 어플리케이션을 계속 서버에서 돌게해ㅐ주는 웹서버 & Rails 앱을 Production모드로 배포할 수 있게 해줌.
>* Passenger : Rails에서 Nginx를 관리할 수 있게 해주는 모듈

## 8. nginx 설치

 1. `rvmsudo passenger-install-nginx-module`
 
 2. `sudo passwd` 후 비밀번호설정
 
 3. `su` 후 비밀번호 입력
 4. `apt-get install libcurl4-openssl-dev`
 
 5. `exit`
 
 6. `rvmsudo passenger-install-nginx-module`
 
 7. enter계속 치다가 1번
 8. 엔터

## 9.환경설정

1. `sudo vi /opt/nginx/conf/nginx.conf`

```
  server {
		listen 80;
		server_name konkuk.com; 
		passenger_enabled on; 
		root /home/ubuntu/ku_stat/public; 
   }
```

   *밑의 server글이 보이면 위의 것 붙여넣기! root는 pwd를 해서 나오는 주소/public;*

```
   server {
       listen       80;
       server_name  localhost;
```

2. `bundle install` 

	* `gem therubyracer` #제거 : javascript코드를 루비에서 사용할 수 있게 하는 gem 
	* `gem 'figaro'` 추가 

3. `bundle exec figaro install` 

4. `rake secret` 을 입력 후 나오는 문자열 복사
5. `config/application.yml`파일에 SECRET_KEY_BASE: '암호화된 메세지' 입력
6. `rake db:migrate RAILS_ENV=production`
7. `sudo /opt/nginx/sbin/nginx`
> nginx서버를 띄우는 것
8. `RAILS_ENV = production rake assets:precompile`
> **Asset pipe line** 을 사용하는 명령어
> Asset = Javascript, Stylesheet(css), images 등을 모두 합쳐 사용자들이 웹페이지를 보다 빠르게 접속할 수 있게 해준다.
9. `mkdir tmp`
> tmp폴더가 없다면! 
10. `touch tmp/restart.txt`
> 수정사항이 있을때마다 서버를 재부팅해준다. (Passenger는 tmp 폴더에 있는 restart.txt라는 파일의 수정시간이 업데이트 될때마다 서버를 스스로 재시작한다.)

11. `<%= image_path '파일명' %>`
> asset pipe line을 쓰면 asset들을 서버에 압축해서 올려주기 때문에 기존의 
> ```
> <img src = "/assets/images/likelion.jpg">
> ``` 
> 는 적용되지 않는다. (파일들 위치와 이름이 마음대로 변경된다.) 대신 파일 이름이 동일하면 그 위치를 알아서 찾아주는 메서드를 사용하면된다.

## 10. Public IP로 접속

참조링크 [ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-rails-and-nginx-with-passenger-on-ubuntu) 


## rbenv 설치 방법

- rbenv 설치

```
$ sudo apt update
$ sudo apt install rbenv
$ sudo apt install ruby-build
$ rbenv rehash
```

- ruby dev설치

```
$ sudo apt install ruby-dev
```

- bundler / rails 설치

```
$ sudo gem install bundler
$ sudo gem install rails -v 5.0.3 --no-ri --no-rdoc
$ rbenv rehash
$ rails -v
```