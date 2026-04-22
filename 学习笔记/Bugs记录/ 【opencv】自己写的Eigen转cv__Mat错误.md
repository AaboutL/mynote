---
title: "\_【opencv】自己写的Eigen转cv::Mat错误"
updated: 2022-11-13 15:03:54Z
created: 2022-11-13 14:56:38Z
---

1. 问题
把相机的内参矩阵K从Eigen转到cv::Mat时，写了如下代码：
```
static void MatEigentoCv(const Eigen::Matrix3d& m, cv::Mat& mat){
        mat = cv::Mat::eye(3, 3, CV_32F);
        mat.at<double>(0, 0) = m(0, 0);
        mat.at<double>(0, 2) = m(0, 2);
        mat.at<double>(1, 1) = m(1, 1);
        mat.at<double>(1, 2) = m(1, 2);
    }
```
但是在运行时，fx, fy和cy的输出是正确的，cx的输出是非常大的值。
改成如下代码：
```
static void MatEigentoCv(const Eigen::Matrix3d& m, cv::Mat& mat){
        mat = cv::Mat::zeros(3, 3, CV_32F);
        for(int i = 0; i < 3; i++){
            for(int j = 0; j < 3; j++){
                mat.at<double>(i, j) = m(i, j);
            }
        }
    }
```
运行时，所有的值都出错。

目前还不清楚原因。
使用opencv内置的函数得到的结果正常：#include <opencv2/core/eigen.hpp>
所以，opencv和eigen之间的转换还是用opencv内置的函数