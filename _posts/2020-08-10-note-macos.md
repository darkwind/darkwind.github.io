---
title: Mac Note
author: DarkwinD
date: 2021-01-01 00:00:00 +0000
categories: [Note]
tags: [mac]
---

# Delete .DS_Store

맥에서 원하는 위치 및 하위 디렉토리에서 .DS_Store 를 삭제하는 방법

``` bash
find . -type f -name .DS_Store -print -delete
```
