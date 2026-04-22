---
title: 【转载】Docker容器显示图形到宿主机屏幕
updated: 2022-08-16 15:20:09Z
created: 2022-08-10 01:04:39Z
latitude: 40.21610000
longitude: 116.23470000
altitude: 0.0000
---

Docker本身的工作模式是命令行的，但是如果运行在docker中的应用需要显示图形界面如何能实现呢？
可以通过在宿主机安装xserver，将docker容器视为客户端，通过网络或挂载的方式就可以实现将需要显示的图像显示在宿主机显示器。

# 方法一: 网络方式（此方式也可以用于两主机间）
```
A.在宿主机
查看宿主机IP
$ ifconfig                          ##假设为xxx.xxx.xxx.xx
查看当前显示的环境变量值
$ echo $DISPLAY   (要在显示屏查看，其他ssh终端不行)  ##假设为:0
或通过socket文件分析：
$ ll /tmp/.X11-unix/                            ##假设为X0= ---> :0

安装xserver
$ sudo apt install x11-xserver-utils
$ sudo vim /etc/lightdm/lightdm.conf 
增加许可网络连接
[SeatDefaults]
xserver-allow-tcp=true
重启xserver
$ sudo systemctl restart lightdm
许可所有用户都可访问xserver
xhost +


B.在docker 容器内
# export DISPLAY=xxx.xxx.xxx.xx:0
注意：环境变量设置需要每次进docker设置，可以写在：/etc/bash.bashrc 文件中，避免每次进终端时设置

```

# 方法二: 挂载方式
挂载方式是在使用image创建docker容器时，通过-v参数设置docker内外路径挂载，使显示xserver设备的socket文件在docker内也可以访问。并通过-e参数设置docker内的DISPLAY参数和宿主机一致。
```
A.在宿主机
查看宿主机IP
$ ifconfig                          ##假设为xxx.xxx.xxx.xx
查看当前显示的环境变量值
$ echo $DISPLAY   (要在显示屏查看，其他ssh终端不行)  ##假设为:0
或通过socket文件分析：
$ ll /tmp/.X11-unix/                            ##假设为X0= ---> :0

安装xserver
$ sudo apt install x11-xserver-utils
许可所有用户都可访问xserver
xhost +


B.在docker 容器创建时
-v /tmp/.X11-unix:/tmp/.X11-unix
-e DISPLAY=:0

例如：
docker run -itd --name 容器名 -h 容器主机名 --privileged \
           -v /tmp/.X11-unix:/tmp/.X11-unix  \
           -e DISPLAY=:0 镜像名或id /bin/bash

```

# 3.验证：
使用带有界面功能的时钟软件尝试
```
在docker容器中：
$ sudo apt-get install xarclock
$ xarclock
应该可以看到xserver端显示器显示时钟界面。

```

参考文章: https://blog.csdn.net/wzw_mzm/article/details/70916202