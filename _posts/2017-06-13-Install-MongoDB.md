---
layout:     post
title:      MongoDB
date:       2017-06-13 22:55:00
summary:    MongoDB 설치하기
---

```
$ brew update
$ brew install mongodb
```

`homebrew`를 이용해서 설치한다.

```
$ mkdir -p /data/db
$ sudo chown -R `id -un` /data/db
$ > # Enter your password
```

몽고db 데이터가 있을 폴더를 생성한다.

```
$ brew tap homebrew/services
```

우선 `brew services`를 설치한 후,

```
$ brew services start mongodb
$ brew services list
$ mongod --version
```

MongoDB service를 시작한다. 그리고 제대로 설치되었는지 버전확인을 해본다.

- Run the Mongo daemo
```
$ mongod
```

- Run the Mongo Shell
```
$ mongo
```

`MongoDB v3.4`에서 실행했을 때 아래와 같은 경고가 뜬다.
```
** WARNING: Access control is not enabled for the database.
**          Read and write access to data and configuration is unrestricted.
```

이때 차례대로 `admin`을 지정해준다.


1) Start MongoDB without access control.
```
$ mongod --port 27017 --dbpath /data/db
```
2) Connect to the instance.
```
$ mongo --port 27017
```
3) Create the user administrator.
```
use admin
db.createUser(
  {
    user: "myUserAdmin",
    pwd: "abc123",
    roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
  }
)
```
4) Re-start the MongoDB instance with access control.
```
$ mongod --auth --port 27017 --dbpath /data/db
```

5) Connect and authenticate as the user administrator
```
mongo --port 27017 -u "myUserAdmin" -p "abc123" --authenticationDatabase "admin"
```

6) Afeter connect
```
$ mongo --port 27017

$ use admin
$ db.auth("myUserAdmin", "abc123" )
```

[참고페이지](https://docs.mongodb.com/master/tutorial/enable-authentication/)