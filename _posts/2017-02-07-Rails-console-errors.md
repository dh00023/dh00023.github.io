---
layout:     post
title:      Rails console error
date:       2017-02-07 02:58:00
summary:    rails console error 해결방법
categories: rails
tags: [rails]
---

```
$ rails c
Running via Spring preloader in process 34212
Loading development environment (Rails 5.0.1)
Cannot read termcap database;
using dumb terminal settings.
irb(main):001:0> ^D
```
이러한 error가 발생하는 경우에는 임시방편으로 `$DISABLE_SPRING=true rails c`를 해도 실행이된다.
하지만 제대로된 방법은 **`rake rails:update:bin`**을 해주면 제대로 실행이된다.