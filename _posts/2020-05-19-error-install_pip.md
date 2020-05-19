---
layout: post
title: Error pip install
tags: [error, pyenv, pip]
---

```bash
$ pip install --upgrade pip
WARNING: pip is configured with locations that require TLS/SSL, however the ssl module in Python is not available.
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError("Can't connect to HTTPS URL because the SSL module is not available.",)': /simple/pip/
```

다음과 같은 오류가 발생하는 경우가 있다. 이때는 패키지 의존성이 깨져서 발생하는 것이므로 해당 python version을 다시 설치해주면 된다.

```bash
$ pyenv install 3.7.2
```

다음과 같이 해당 버전을 다시 설치한 후 패키지를 설치하면 정상적으로 되는 것을 확인할 수 있다.

## 참조

- [https://devlog.jwgo.kr/2020/02/29/pip-error-configured-with-locations-that-require-tls-ssl/](https://devlog.jwgo.kr/2020/02/29/pip-error-configured-with-locations-that-require-tls-ssl/)