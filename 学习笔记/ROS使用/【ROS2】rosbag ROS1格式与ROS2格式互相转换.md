---
title: 【ROS2】rosbag ROS1格式与ROS2格式互相转换
updated: 2022-10-16 07:35:07Z
created: 2022-10-16 07:25:19Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

ROS1和ROS2中rosbag的格式不一致，不能交叉使用，但是目前公开数据集的rosbag都是ROS1格式的，所以在ROS2中使用，需要自己进行转换。
# 转换工具——rosbags
rosbags使用手册链接：https://ternaris.gitlab.io/rosbags/topics/convert.html
源码链接：https://gitlab.com/ternaris/rosbags
rosbags是一个纯python工具包，功能：
1. 读写rosbag2
2. 读写rosbag1
3. 支持序列化和反序列化的类型系统
4. **支持rosbag1和rosbag2之间高效的转换**
5. 等等

**rosbags完全不依赖ROS，可以单独使用，也可以和ROS一起使用**