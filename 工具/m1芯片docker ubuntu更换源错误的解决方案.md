---
title: m1芯片docker ubuntu更换源错误的解决方案
updated: 2022-08-09 08:52:14Z
created: 2022-08-09 08:38:09Z
---

# 解决方法
## m1芯片上更换成国内阿里源应该是：
deb http://mirrors.aliyun.com/ubuntu-ports/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu-ports/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu-ports/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu-ports/ focal-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu-ports/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu-ports/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu-ports/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu-ports/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu-ports/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu-ports/ focal-backports main restricted universe multiverse

## m1芯片ubuntu官方的源为：
deb http://ports.ubuntu.com/ubuntu-ports/ bionic main restricted
deb http://ports.ubuntu.com/ubuntu-ports/ bionic-updates main restricted
deb http://ports.ubuntu.com/ubuntu-ports/ bionic universe
deb http://ports.ubuntu.com/ubuntu-ports/ bionic-updates universe
deb http://ports.ubuntu.com/ubuntu-ports/ bionic multiverse
deb http://ports.ubuntu.com/ubuntu-ports/ bionic-updates multiverse
deb http://ports.ubuntu.com/ubuntu-ports/ bionic-backports main restricted universe multiverse
deb http://ports.ubuntu.com/ubuntu-ports/ bionic-security main restricted
deb http://ports.ubuntu.com/ubuntu-ports/ bionic-security universe
deb http://ports.ubuntu.com/ubuntu-ports/ bionic-security multiverse

**仔细看可以发现，arm64芯片上的源，地址最后为ubuntu-ports**
看下x86_64架构下的源：
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
**可以看出，这个架构下的源，地址最后不带ports**

**x86_64/amd64**架构下，docker中ubuntu的官方源的格式如下：
deb http://archive.ubuntu.com/ubuntu/ focal-backports main restricted universe multiverse
**与m1芯片的官方源对比，区别在与ports和archive**