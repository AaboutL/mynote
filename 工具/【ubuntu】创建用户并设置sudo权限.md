---
title: 【ubuntu】创建用户并设置sudo权限
updated: 2024-08-15 11:05:25Z
created: 2023-08-09 06:35:31Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

1. 创建用户
	~~~
	sudo adduser username
	~~~
2. 设置sudo权限
	~~~
	sudo usermod -aG sudo username
	~~~
3. 给与用户root权限
	~~~
	sudo vim /etc/sudoers
	添加：
	root ALL=(ALL) ALL
	username ALL=(ALL) ALL
	~~~