---
title: Mac OS Update 오류(Xcode)
categories:
- setting
tag:
- Xcode
- git
---

Mac OS를 update이후에 git을 사용시 Xcode CommandLineTools 오류가 발생했다.

### 오류

```bash
xcrun: error: invalid active developer path
 (/Library/Developer/CommandLineTools), missing xcrun at:
 /Library/Developer/CommandLineTools/usr/bin/xcrun
```

### 해결방법

Mac에서 git은 Xcode의 command line tools에 의존한다. 그러므로 Xcode의 Command Line Tools를 설치(update)해주면된다.

```bash
$ xcode-select --install
```
