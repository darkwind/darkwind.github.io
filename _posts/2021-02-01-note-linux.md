---
title: Linux Note
author: DarkwinD
date: 2021-02-01 00:00:00 +0000
categories: [Note]
tags: [linux, ubuntu]
---

# Ubuntu Command

설치된 패키지 확인
---
``` bash
dpkg -l
```

Ubuntu의 패키지 관리 유틸리티인 dpkg를 이용하면 패키지를 제거할 수 있다.

옵션은 –remove, –purge의 2가지가 있는데 remove는 binary만 제거하고 설정 등의 데이터는 그대로 남겨둔다.
remove를 할 경우에는 crontab의 설정이나 /etc/rc?.d 등의 설정이 그대로 남기 때문에 에러 로그가 쌓이는 등의 부작용이 있다.

깔끔하게 삭제할 경우에는 purge를 이용하면 된다.

dpkg –list 명령의 결과를 보면
제일 앞의 컬럼에 'ii', 'rc' 등을 볼 수 있는데
'rc'는 remove되었고 purge 되지는 않았다는 것을 뜻한다.

rc 된 항목만 찾아서 지우기
---
``` bash
dpkg -l | grep "^rc" | cut -d " " -f 3 | xargs sudo dpkg --purge
```

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