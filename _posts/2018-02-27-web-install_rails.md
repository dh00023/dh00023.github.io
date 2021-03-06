---
layout: post
title: Mac에 Rails 설치하기
tags: [ruby-on-rails, mac, homebrew]
---

## Homebrew 설치하기 [[페이지 바로가기](https://brew.sh/index_ko.html)]
```
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

설치가 되어있다면 최신 버전으로 update해준다.
```
$ brew update
```


주로 **rbenv** 또는 **rvm**으로 설치한다. 둘중에 한가지를 이용해서 설치하면된다.

## rbenv 설치

```
$ brew install rbenv && brew upgrade ruby-build
```

설치할 수 있는 루비버전 확인하기[[현재 안정 버전의 루비 버전 확인하기](https://www.ruby-lang.org/ko/downloads/)]
```
# 경로(path)확인
$ env | grep PATH

# 경로 설정해주기
$ export PATH="$HOME/.rbenv/bin:$PATH"
$ eval "$(rbenv init -)"
```

```
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
$ echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
```

```
$ rbenv install -l
```
설치할 수 있는 버전 확인하기

```
$ rbenv install 2.5.0 && rbenv rehash
$ rbenv global 2.5.0
```
이때 rehash는 새로운 환경을 재설정하기 위한 것이다.

설치된 루비버전 확인해보기
```
$ ruby -v
```

#### bundler 설치
```
$ gem install bundler
$ rbenv rehash
```

#### Rails 설치
```
$ gem install rails --version 5.0.6 --no-ri --no-rdoc
```
이때 레일즈 버전을 확인하고 설치하자.

### Ruby 버전별 레일즈 설치하기

```
$ % RBENV_VERSION=2.4.0 rbenv exec gem install rails --version 5.0.6 --no-ri --no-rdoc
```

루비 버전 2.4.0에 5.0.6 레일즈를 설치한 것이다.

```
$ rbenv local 2.4.0
```


## rvm 설치 [[페이지 바로가기](https://rvm.io/)]

```
$ gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
$ \curl -sSL https://get.rvm.io | bash -s stable
$ source ~/.profile
```

#### ruby 설치
```
$ rvm install ruby 2.5.0
$ rvm use 2.5.0 --default
$ ruby -v
```

#### bundler 설치
```
$ gem install bundler
```

#### Rails 설치
```
$ gem install rails --version 5.0.6 --no-ri --no-rdoc
$ rails -v
```

