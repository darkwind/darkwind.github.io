---
layout: default
title: Ubuntu Command
parent: Linux
nav_order: 1
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
