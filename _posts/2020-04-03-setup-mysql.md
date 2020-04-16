---
layout: post
title: mac에서 mysql 비밀번호 초기화하기
tags: [mysql, mac]
---

```sh
$ mysql -u root
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
```
이때는 비밀번호 설정이 안된경우로 다음과 같이 비밀번호를 같이 입력해주면된다.

```sh
$  mysql -u 사용자 -p 비번
```

### 비밀번호 재설정

```sh
$ mysql -u root
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
```
이때는 비밀번호가 틀린 경우이다.

Mac에서 mysql 비밀번호를 잃어버린 경우 다음과 같이 명령어를 입력한다.
이떄 **mysql 서버가 중단된 상태**여야한다.(확인 필요)

```sh
$ brew services stop mysql                        
Stopping `mysql`... (might take a while)
==> Successfully stopped `mysql` (label: homebrew.mxcl.mysql)
```

```sh
$ sudo mysqld_safe --skip-grant-tables
```

근데 이때 mysqld_safe 오류가 발생할 수 있다.

#### mysqld_safe 오류

```sh
2020-04-03T04:59:31.6NZ mysqld_safe Logging to '/usr/local/var/mysql/jeongdahyeui-iMac.local.err'.
2020-04-03T04:59:31.6NZ mysqld_safe A mysqld process already exists
```

이때는 mysqld와 관련된 프로세스를 종료시켜주면된다.

```sh
$ killall mysqld mysqld_safe
```
killall을 해준 후 10초뒤면 shutdown이 된다. 

```sh
$ ps aux | grep mysqld
 ps aux | grep mysqld                                 ✔  7160  13:59:32
jeongdaye        40605   0.0  0.0  4418712    752 s002  S+    1:59PM   0:00.00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn mysqld
jeongdaye        40585   0.0  1.5  5235688 517676   ??  S     1:59PM   0:04.52 /usr/local/opt/mysql/bin/mysqld --basedir=/usr/local/opt/mysql --datadir=/usr/local/var/mysql --plugin-dir=/usr/local/opt/mysql/lib/plugin --log-error=jeongdahyeui-iMac.local.err --pid-file=jeongdahyeui-iMac.local.pid
jeongdaye        40486   0.0  0.0  4271832   1132   ??  S     1:59PM   0:00.02 /bin/sh /usr/local/opt/mysql/bin/mysqld_safe --datadir=/usr/local/var/mysql
```

남아있는 프로세스가 있는지 확인을 해보고 남아있다면 아래 명령어로 종료시켜준다.

```sh
$ killall -9 mysqld mysqld_safe
```

그 후에 다시 명령어를 실행하면 된다.

```sh
$ sudo mysqld_safe --skip-grant-tables
2020-04-03T06:27:31.6NZ mysqld_safe Logging to '/usr/local/var/mysql/jeongdahyeui-iMac.local.err'.
2020-04-03T06:27:31.6NZ mysqld_safe Starting mysqld daemon with databases from /usr/local/var/mysql
2020-04-03T06:27:31.6NZ mysqld_safe mysqld from pid file /usr/local/var/mysql/jeongdahyeui-iMac.local.pid ended
```



## 참고

- [https://hoststud.com/resources/resolved-error-mysqld_safe-a-mysqld-process-already-exists.141/](https://hoststud.com/resources/resolved-error-mysqld_safe-a-mysqld-process-already-exists.141/)
- [성장하는 호랑이](https://babytiger.tistory.com/entry/mysql에-로그인이-안-될-경우)