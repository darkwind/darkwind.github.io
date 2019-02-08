---
layout: default
title: VIM Tip
parent: Linux
nav_order: 2
---

# VIM Tip

vi, vim에서 '^M' 제거 하는 방법
---

윈도우의 개행문자가 달라 git에서 받은 파일 내용에 '^M' 이 붙어 있는 경우

[CR,LF 뜻]

라인피드(LF : Line Feed) => 현재 위치에서 바로 아래로 이동

캐리지리턴(CR: Carriage return) => 커서의 위치를 앞으로 이동

``` bash
# ^M는 ^+M 이 아니고 Ctrl + v + m 이다.
:%s/^M//g
```
