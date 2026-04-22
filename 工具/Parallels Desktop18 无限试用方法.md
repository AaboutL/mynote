---
title: Parallels Desktop18 无限试用方法
updated: 2022-10-27 03:48:34Z
created: 2022-10-23 12:41:31Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

# 方法一：
修改系统时间

1.  首先关闭软件
2.  把系统时间改成过去的时间
    
    ```
    sudo systemsetup -setusingnetworktime Off && sudo systemsetup -setdate 10:01:2022
    ```
    
3.  重新打开软件
4.  把系统时间改回来
    
    ```
    sudo systemsetup -setusingnetworktime On
    ```
    

* * *

上面的方法不好使了，似乎得一直保持系统时间是过去才行，只要修改回当前的时间，试用日期就变了。

上面的方法，改回当前时间后，虽然试用日期变了，但是只要没有退出app，就能够继续使用几分钟，应该是PD每隔几分钟就会检查一下。

所以，如果想一直试用，就需要保持宿主机的机器时间一直为过去的某个时间。

# 方法二：
https://git.icrack.day/somebasj/ParallelsDesktopCrack/src/branch/main