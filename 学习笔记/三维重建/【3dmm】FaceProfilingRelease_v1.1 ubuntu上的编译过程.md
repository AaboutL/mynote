---
title: 【3dmm】FaceProfilingRelease_v1.1 ubuntu上的编译过程
updated: 2023-06-29 08:05:35Z
created: 2023-06-29 07:56:20Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

1. 编译安装opencv，最好安装到系统目录，省的麻烦
2. 分别编译出 ZBufferC.mexa64、ZBufferTriC.mexa64、Mex_FaceFrontalizationMappingC.mexa64、Mex_FaceFrontalizationMappingBigTriC.mexa64、Mex_FaceFrontalizationFillingC.mexa64
其中，ZBufferC和ZBufferTriC在本项目中没有，需要从3DDFA_Release项目中拷贝过来
编译后，执行main.m文件时，会出现libopencv.so.407找不到的问题，主要是安装opencv后，没有对一些系统路径进行配置。按照“【opencv】完整编译安装配置过程” 中的过程配置opencv

3. 然后执行main.m文件时，出现下列错误
	~~~
	/usr/local/MATLAB/R2018b/bin/glnxa64/../../sys/os/glnxa64/libstdc++.so.6: version `GLIBCXX_3.4.26' not found
	~~~
	这个路径是matlab的安装路径，这个路径下的libstdc++.so.6中的GLIBCXX版本太低。所以把系统中的libstdc++.so.6软连接到当前目录就可以解决。