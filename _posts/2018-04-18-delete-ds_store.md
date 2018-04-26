---
title: "맥에서 .DS_Store 삭제하는 방법"
date: 2018-04-18 16:01:00
layout: post
category: [macos]
tags: [DS_Store, command]
---

맥에서 원하는 위치 및 하위 디렉토리에서 .DS_Store 를 삭제하는 방법

``` bash
find . -type f -name .DS_Store -print -delete
```
