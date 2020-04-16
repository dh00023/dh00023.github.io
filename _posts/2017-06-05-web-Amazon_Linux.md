---
layout: post
title: AWS EC2 Amazon Linux deploy하기
tags: [ruby-on-rails, depoly, aws, ec2, linux]
---

### AWS instance 생성

1. EC2 - launch instance
2. Amazon Linux 선택
3. 프리티어(무료 - t2.micro) 선택 후 review and launch
4. Security Group - Edit security group - Add Rule - HTTP선택 - Review and launch
|Type|Portocal|Port Range|
|-----|-----|-----|
|HTTP|TCP|80|

5. create key pair - 이름입력 후 Download key pair (이 키가 있어야 서버에 접속할 수 있다.)
6. 서버에 이름 지어주기

### 서버 연결하기

1. chmod 400 (다운로드 한 pem key 위치)
```
chmod 400 <이름>.pem
chmod 400 test.pem // 예제
```

2. ssh –i [키파일] [user_name@host_name]
```
ssh -i test.pem ec2-user@52.75.323.321
```

### git 설치하기

```
$ sudo yum update
$ sudo yum install git-all.noarch
```

### 내 프로젝트 clone해오기
```
$ git clone <gitrepository주소>
```

### 루비 설치하기

#### rvm

루비를 쉽게 설치하는 준비작업이다.

- ([https://rvm.io/](https://rvm.io/))에서 install rvm하기

```
$ gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
$ \curl -sSL https://get.rvm.io | bash -s stable
$ source ~/.profile

$ rvm install ruby 2.3.3
```
```
$ gem install bundler
```

주석처리 되어있는 `$ gem therubyracer` 를 활성화시켜주세요.
Rubyracer은 Javascript 코드를 루비에서 사용할 수 있게 해주는 gem이다.

```
$ bundle install
```

#### rbenv

- rbenv 설치

```
$ git clone git://github.com/sstephenson/rbenv.git .rbenv
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
$ echo 'eval "$(rbenv init -)"' >> ~/.bashrc
$ source ~/.bashrc
```

- ruby-build 설치

```
$ git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
$ echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc
$ source ~/.bashrc
```

- ruby 설치

```
$ rbenv install 2.3.3
$ rbenv global 2.3.3
$ rbenv rehash
$ ruby -v
```

```
$ gem install bundler
$ gem install rails
$ rbenv rehash
$ rails -v
```

주석처리 되어있는 `$ gem therubyracer` 를 활성화시켜주세요.
Rubyracer은 Javascript 코드를 루비에서 사용할 수 있게 해주는 gem이다.

```
$ bundle install
```

### passenger 설치

Nginx 서버를 사용하기 위해 필요한 gem이다.

Nginx는 웹 어플리케이션을 계속 서버에서 돌게 해주는 웹 서버이다. Passenger는 Rails에서 Nginx를 관리할 수 있게 해주는 모듈이다.

```
$ gem install passenger
```
```
$ sudo passwd
// 초기 1회 비밀번호 설정
```
```
$ su // 명령어를 통해 root권한 획득
```

### Nginx 설치

```
$ passenger-install-nginx-module
$ exit
```
설치가 끝나면 exit를 통해 `su`권한을 끝낸다.

### 환경설정

```
$ sudo vi /opt/nginx/conf/nginx.conf
```

```conf
// /opt/nginx/conf/nginx.conf
server {

    listen 80;

    server_name likelion.com;

    passenger_enabled on;

    root /home/ec2-user/Myproject/public;
}
// 여기 위에 추가
server{
 ...
}
```

기존에 존재하는 server위에 코드를 추가해준다.

### Figaro

```
# Gemfile
gem 'figaro'
```

```
$ bundle install
$ bundle exec figaro install
```

`application.yml` 파일에 `rake secret`을 쳐서 나온 키 값을 입력한다.

```
production:
	SECRET_KEY_BASE : ’암호화된 메시지’
```

```
production:
	secret_key_base: <%=ENV["SECRET_KEY_BASE"]%>
```

### migration 실행

```
$ rake db:migrate RAILS_ENV=production
```

### Ngnix 서버 실행

```
$ sudo /opt/nginx/sbin/nginx
```

### 이미지 경로 설정

```
<%= image_path ‘파일명’ %>
```

### Asset pipeline

Javascript, Stylesheet(css), images 등 을 모두 합쳐버려서 사용자들이 웹페이지에 보다 빠르게 접속할 수 있게해준다.

```
$ RAILS_ENV=production rake assets:precompile
```

### 서버 재부팅 설정

```
$ touch tmp/restart.txt
```