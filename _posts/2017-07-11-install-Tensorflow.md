---
layout:     post
title:      Tesorflow 설치하기
date:       2017-07-11 19:08:00
summary:    Tensorflow 설치하기
---


# Tensorflow 설치하기

1. pip and virtualenv 설치
```
$ sudo easy_install pip
$ sudo pip install --upgrade virtualenv
```

2. a virtualenv environment 생성
```
$ virtualenv --system-site-packages -p python3 targetDirectory # for Python 3.n
```

3. virtualenv environment 활성화
```
$ source ~/tensorflow/bin/activate      # If using bash, sh, ksh, or zsh
```

4. pip3 버전 업그레이드
```
$ pip3 install --upgrade pip
```

5. Tensorflow와 패키지 설치
```
$ pip3 install --upgrade tensorflow     # for Python 3.n
```

6. Tensorflow설치후 다시 활성화해보면 `(tensorflow)`가 뜬다.
```
$ source ~/tensorflow/bin/activate      # bash, sh, ksh, or zsh
(tensorflow) dahyeui-MacBook-Pro:~ dh0023$
```

7. Tensorflow 종료
```
$ deactivate 
```

8. Tensorflow 삭제
```
$ rm -r ~/tensorflow
```
