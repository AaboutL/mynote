---
title: slam基础-判断IMU是否充分激励
updated: 2022-11-18 03:32:17Z
created: 2022-11-18 02:51:06Z
---

# 什么是IMU的激励
单目slam存在的一个问题是没有绝对尺度信息，即获得的位置信息无法反应到现实世界。使用IMU的一个目的就是获取绝对尺度信息。最简单的IMU由一个加速度计和一个陀螺仪组成。加速度计可以测量IMU的加速度。
假设F1、F2、F3为三个连续的图像帧，P1、P2、P3为通过单目获得的位置，P1、P2、P3是没有绝对尺度的。假设P1、P2、P3与真实的带有尺度（单位:米）的位置相比，相差尺度s。现在对P1、P2、P3进行插值使其连续并微分，可以得到速度v1、v2、v3，速度与真实速度同样相差尺度s。而IMU可以测量出v1与v2、v2与v3之间真实的速度变化，然后对齐之后就可以得到尺度：
$$
s(v2 - v1) = at, t是F1、F2之间的时间间隔
$$
所以需要IMU可以测量出加速度，进而要保证F1、F2、F3之间不能是匀速运动，这就是IMU需要的激励

## 1. VINS-mono的方法
考察线加速度的变化，计算滑窗中加速度的变化，只要标准差大于0.25，表示已经充分激励，足够初始化
```
    //check imu observibility
    {
        map<double, ImageFrame>::iterator frame_it;
        Vector3d sum_g;
        for (frame_it = all_image_frame.begin(), frame_it++; frame_it != all_image_frame.end(); frame_it++)
        {
            double dt = frame_it->second.pre_integration->sum_dt;
            Vector3d tmp_g = frame_it->second.pre_integration->delta_v / dt;
            sum_g += tmp_g;
        }
        Vector3d aver_g;
        aver_g = sum_g * 1.0 / ((int)all_image_frame.size() - 1);
        double var = 0;
        for (frame_it = all_image_frame.begin(), frame_it++; frame_it != all_image_frame.end(); frame_it++)
        {
            double dt = frame_it->second.pre_integration->sum_dt;
            Vector3d tmp_g = frame_it->second.pre_integration->delta_v / dt;
            var += (tmp_g - aver_g).transpose() * (tmp_g - aver_g);
            //cout << "frame g " << tmp_g.transpose() << endl;
        }
        var = sqrt(var / ((int)all_image_frame.size() - 1));
        //ROS_WARN("IMU variation %f!", var);
        if(var < 0.25)
        {
            ROS_INFO("IMU excitation not enouth!");
            //return false;
        }
    }
```