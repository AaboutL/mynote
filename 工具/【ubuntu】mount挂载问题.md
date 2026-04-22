---
title: 【ubuntu】mount挂载问题
updated: 2024-09-05 09:22:20Z
created: 2023-10-25 06:32:56Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

## server上确保安装nfs-common、nfs-kernel-server
## nfs-common.service is masked解决方法
1. 查看： file /lib/systemd/system/nfs-common.service
2. 删除：sudo rm /lib/systemd/system/nfs-common.service
3. 重启daemon：sudo systemctl daemon-reload
4. 重启：sudo systemctl start nfs-common

## server的/etc/exports中添加
```
/home/用户名/nfs_rootfs *(rw,sync,no_root_squash,no_subtree_check)
```
/home/用户名/nfs_rootfs ：nfs客户端挂载目录  ：允许所有的网段访问，也可以使用具体的IP
rw ：挂接此目录的客户端对该共享目录具有读写权限
sync ：资料同步写入内存和硬盘
no_root_squash ：root用户具有对根目录的完全管理访问权限。
no_subtree_check：不检查父目录的权限。

## server服务上需要安装nfsd，并启动
```
sudo apt install nfs-kernel-server
sudo systemctl start nfs-kernel-server
sudo systemctl status nfs-kernel-server 查看状态

```