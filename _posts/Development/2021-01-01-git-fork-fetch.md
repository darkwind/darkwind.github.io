---
title: Fork Repository 동기화
author: DarkwinD
date: 2021-01-01 00:00:00 +0000
categories: [Development, Git]
tags: [git]
---

## Github 원본 repository 에서 업데이트 내용을 받아오기 위해 사용

현재 remote repository 확인
---
``` bash
$ git remote -v
origin  https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)
origin  https://github.com/YOUR_USERNAME/YOUR_FORK.git (push)
```

원본 repository 추가
---
``` bash
$ git remote add upstream https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git

$ git remote -v
origin    https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)
origin    https://github.com/YOUR_USERNAME/YOUR_FORK.git (push)
upstream  https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git (fetch)
upstream  https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git (push)
```

Fetch 명령어로 upstream repository 의 최신 정보를 받아온다
---
``` bash
$ git fetch upstream
remote: Counting objects: 75, done.
remote: Compressing objects: 100% (53/53), done.
remote: Total 62 (delta 27), reused 44 (delta 9)
Unpacking objects: 100% (62/62), done.
From https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY
 * [new branch]      master     -> upstream/master
```

upstream repository 에서 나의 local master branch 로 merge 한다.
---
``` bash
$ git checkout master
Switched to branch 'master'
​
$ git merge upstream/master
Updating a422352..5fdff0f
Fast-forward
 README                    |    9 -------
 README.md                 |    7 ++++++
 2 files changed, 7 insertions(+), 9 deletions(-)
 delete mode 100644 README
 create mode 100644 README.md
```

변경된 내용을 origin에 push 한다.
---
``` bash
$ git push origin master
```