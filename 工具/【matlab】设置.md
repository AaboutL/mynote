---
title: 【matlab】设置
updated: 2024-10-17 07:27:34Z
created: 2023-06-29 03:57:13Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

# 1. 界面缩放调整
在matlab 的 command window中输入下面命令（1.5是缩放比例），然后重启matlab
~~~
s = settings;
s.matlab.desktop.DisplayScaleFactor;
s.matlab.desktop.DisplayScaleFactor.PersonalValue = 1.5
~~~

# 2. Linux下，添加matlab的按钮
~~~
[Desktop Entry]
Type=Application
Name=Matlab
GenericName=Matlab R2020b
Comment=Matlab:The Language of Technical Computing
Exec=/usr/local/MATLAB/R2020b/bin/matlab
Icon=/usr/local/MATLAB/R2020b/bin/glnxa64/cef_resources/matlab_icon.png
Categories=Development;Matlab;
~~~