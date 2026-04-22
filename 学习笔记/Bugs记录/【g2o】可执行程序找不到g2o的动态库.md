---
title: 【g2o】可执行程序找不到g2o的动态库
updated: 2022-10-09 07:29:11Z
created: 2022-10-09 07:22:49Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

可执行程序已经通过cmake链接了g2o的动态库，但是在运行的时候，提示找不到对应的动态库，如下：
```
./main: error while loading shared libraries: libg2o_opengl_helper.so: cannot open shared object file: No such file or directory
```

**解决方法**：
1. CMakeLists.txt中通过target_link_libraries()显式的添加.so
2. 命令行中通过export LD_LIBRARY_PATH=$dir;
3. 如果上面的方法都解决不了，则
```
1. sudo vim /etc/ld.so.conf
2. 把.so所在的文件夹路径添加进去，保存退出
3. sudo ldconfig
```