---
title:  "Ubuntu에 Python 3.7.0 설치하는 방법"
date:   2018-11-08 15:44:23
categories: [ubuntu, python]
tags: [install]
---

Ubuntu에 Python 3.7.0 설치하는 방법
===

``` bash
$ sudo apt-get install build-essential checkinstall
$ sudo apt-get install \
python-dev \
python-setuptools \
python-pip \
python-smbus \
libreadline-gplv2-dev \
libncursesw5-dev \
libgdbm-dev \
libc6-dev \
libbz2-dev \
zlib1g-dev \
libsqlite3-dev \
tk-dev \
libssl-dev \
openssl \
libffi-dev

$ cd /usr/local/src
$ sudo wget https://www.python.org/ftp/python/3.7.0/Python-3.7.0.tgz
$ sudo tar xzf Python-3.7.0.tgz
$ cd Python-3.7.0
$ sudo ./configure --enable-optimizations
$ sudo make altinstall

$ python3.7 -V
```
