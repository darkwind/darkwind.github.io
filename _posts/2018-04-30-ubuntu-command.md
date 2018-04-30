---
title: "우분투 커맨드"
date: 2018-04-30 12:00:00
layout: post
categories: [linux]
tags: [dpkg]
---

설치된 패키지 확인
===
``` bash
dpkg -l
```

rc 된 항목만 찾아서 지우기
===
``` bash
dpkg -l | grep "^rc" | cut -d " " -f 3 | xargs sudo dpkg --purge
```
