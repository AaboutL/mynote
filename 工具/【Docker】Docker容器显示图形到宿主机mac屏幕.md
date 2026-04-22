---
title: 【Docker】Docker容器显示图形到宿主机mac屏幕
updated: 2024-02-19 06:37:44Z
created: 2022-08-16 15:20:18Z
---

1.  安装XQuartz
    
2.  配置XQuartz：偏好设置-》安全性-》允许从网络客户端连接
    
3.  mac终端执行：
    
    ```
    xhost+
    ```
    
4.  运行容器
    
    ```
    docker exec -it -e DISPLAY=[mac的ip]:0 [容器名] /bin/bash
    ```
    
5.  安装socat：从XQuartz 2.7.9开始，不再监听tcp 6000端口了，只能把unix socket转换为tcp socket才能正常运行x11应用 https://github.com/moby/moby/issues/8710#issuecomment-72669844
    
    ```
    brew install socat
    socat TCP-LISTEN:6001,reuseaddr,fork UNIX-CLIENT:\"$DISPLAY\"
    ```
    
6.  启动XQuartz，在docker容器中运行程序，可以显示opencv图形；
    
7.  通过Clion运行并显示图形，需要对可执行程序的Environment variables配置DISPLAY，如下图
    <img src="../_resources/1ae4510a300e1767d0f762e7b42e9b4e.png" alt="1ae4510a300e1767d0f762e7b42e9b4e.png" width="358" height="195">
    <img src="../_resources/db62ff5b631647857410dc79cdf74e66.png" alt="db62ff5b631647857410dc79cdf74e66.png" width="692" height="244">
    

# 使用中遇到的问题

1.  使用一段时间后，突然没办法显示图像了。尝试过重启xQuartz、export DISPLAY=192.168.2.43:0、重新执行“ socat TCP-LISTEN:6001,reuseaddr,fork UNIX-CLIENT:"$DISPLAY" “，都不好使。
    **解决方法**：在mac终端中执行” xhost + “。