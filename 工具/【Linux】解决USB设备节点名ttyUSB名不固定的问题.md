---
title: 【Linux】解决USB设备节点名ttyUSB名不固定的问题
updated: 2023-06-07 08:20:23Z
created: 2023-06-07 08:19:23Z
latitude: 39.91014100
longitude: 116.35732000
altitude: 0.0000
---

链接：http://linuxdown.net/install/faq/20211022_how_linux_38430.html

1. Linux下USB设备节点名不固定问题经常会遇到

以USB转串口设备为例，通常设备节点名为ttyUSBx（x为0~n）,Linux内核会根据设备插入的先后顺序进行编号的分配，比如第一个插入的设备编号为ttyUSB0，然后依此加1，变为ttyUSB1，ttyUSB2……

如果仅仅以设备节点ttyUSBx来区别具体是哪个设备，因为末位的编号是随时会变的，所以就会造成混乱。无法保证A设备就是ttyUSB0，B设备就是ttyUSB1。在设备文件/dev目录下并没有提供固定显示ttyUSB的方法，但是，其实，每个USB端口都有唯一的端口号，相当于每个门店的门牌号。只要我们依据端口号来进行设备的区分，那么问题就迎刃而解了。简单点来说就是找到端口号，然后根据端口号找到挂载在这个端口号上面的USB设备是ttyUSB0还是 ttyUSB1….(这个是变化的，前面讲到了)。



2. 关于端口号的查看方法,连接好两个USB转串口设备之后:

执行命令：ls -l /sys/class/tty/ttyUSB*

lrwxrwxrwx root     root              2017-08-01 13:40 ttyUSB0 -> ../../devices/ff540000.usb/usb3/3-1/3-1.1/3-1.1:1.0/ttyUSB0/tty/ttyUSB0

lrwxrwxrwx root     root              2017-08-01 13:43 ttyUSB1 -> ../../devices/ff540000.usb/usb3/3-1/3-1.2/3-1.2:1.0/ttyUSB1/tty/ttyUSB1

可知：其中ttyUSB0所在的端口号为3-1.1，而ttyUSB1所在的端口号为3-1.2，可以看出，这里的3-1.1端口上比3-1.2上提前插上USB设备，所以会以这种方式命名。如果插入设备的顺序相反，那么端口号3-1.1上对应的设备应该是ttyUSB1，而3-1.2上对应的设备应该是ttyUSB0。但是如果在实际过程中我们只需要采集端口3-1.1传来的数据，我们该如何通过ttyUSBx不定的设备节点，获取到固定端口的数据呢？



3. 实际问题，实际解决：

实际工程中，碰到的一个问题是：硬件上连接有两个USB转串口设备，硬件只要一上电，两个USB设备几乎是同时上电的，这将导致ttyUSB0或者ttyUSB1无法每次固定的对应到上一次的那个相同端口，上层软件需要通过串口设备节点/dev/ttyUSBx来打开一个串口，并从串口获取数据，但是这个ttyUSBx设备并不是一直都指向固定的一个usb端口号，这直接导致我们无法往下操作了。

解决办法：这里使用bash语言加Python正则表达式的相关知识解决这个问题：解决问题的思路是：

1. 第一次上电的时候，我们需要确定哪个端口上的数据是我们所需要的：

ls -l /sys/class/tty/ttyUSB*

假设是3-1.1这个端口是我们的data端口。

2. 以后每次上电，我们要找到3-1.1这个端口后面挂载的ttyUSB设备是ttyUSB0还是ttyUSB1，并建立一个软链接将当时获取到的ttyUSBx生成一个软链接，名字固定为ttydata,那么以后每次打开/dev/ttydata就能找到正确3-1.1这个端口，并获取数据了。

3. 建立一个文件夹getUSB，该文件夹下面包含：


其中cmd.sh是利用bash脚本获取/sys/class/tty/ttyUSB*的一些信息保存在device_usb.txt中

getUSB.py是通过device_usb.txt中的信息，获取到当前挂着在端口3-1.1上的是ttyUSB0还是ttyUSB1并保存在usbdev 中

cmd.sh如下：

#!/bin/bash

declare -i a=0

declare -i b=0 

while [[ ! -e "/sys/class/tty/ttyUSB0" ]]

do

sudo sleep 0.01s

a=a+1

if [ $a -eq 300 ];then  #等待一段时间没有检测ttyUSB0设备到会自动跳出while

break

fi

done



while [[ ! -e "/sys/class/tty/ttyUSB1" ]]

do

sudo sleep 0.01s

b=b+1

if [[ $b -eq 300||$a -ne 0 ]];then  #if USB0 been detected ,also get out of while

break

fi

done





if [[ ! -e /sys/class/tty/ttyUSB0&&! -e /sys/class/tty/ttyUSB1 ]]; then #如果不存在ttyUSB设备

echo "Not have ttyUSB0 or not have ttyUSB1"

else                                   #如果完美检测到了两个ttyUSB设备，则将信息log到device_usb.txt当中

tty1=$(ls -l /sys/class/tty/ttyUSB0) 

tty2=$(ls -l /sys/class/tty/ttyUSB1)



sudo ls -l /sys/class/tty/ttyUSB0 /sys/class/tty/ttyUSB1 > ./device_usb.txt

fi



if [ ! -n "$tty1" ] ;then   # "! -n" shows blank var  #非空检测

echo "tty1 is empty"

fi

#delay 0.01s to make sure the device_usb.txt complete

sudo sleep 0.01s

#remove the old USB device shortcut



if [ ! -e "/dev/ttydata" ] ;then # 如果/dev/ttydata本身不存在

echo "-------------/dev/ttydata not found"

else                                     #如果存在，则需删除之，然后重新创建之

echo "/dev/ttydata is exist"

sudo rm /dev/ttydata

fi



#exct Python language to get the rignt USB interface

./getUSB.py    #调用当前路径下的getUSB.py这个Python语言，明确此次是哪个,ttyUSB0,或者ttyUSB1挂载在端口3-1.1上



usbdev=$(cat ./usbdev) #获取到这个设备

echo "the device is : "

echo $usbdev



sudo ln -s /dev/$usbdev /dev/ttydata #将这个设备软连接到/dev/ttydata以后每次打开这个ttydata即可



getUSB.py:

#!/usr/bin/python

#coding:utf-8

import re  #正在表达式

sss = open("./device_usb.txt","rb") #打开device_usb.txt设备，并读取内容

www = open("./usbdev","wb")  #当前路径下创建usbdev文件，后续会写入内容

s_read = sss.read()  usb3/3-1/3-1.1/

r = r"usb3/3-1/3-1\.1.+(ttyUSB[0-9])"

#正则中“.”需要转义，所以使用“\.”表示“.”

#这个规则是找到usb3/3-1/3-1.1/这个字符串后面紧跟的是此次上电生成的ttyUSB0或者ttyUSB1

output = re.findall(r,s_read)

(output[0]) #将结果写到usbdev中

()

sss.close()



完成之后设置开机项目，将文件夹当道一个位置，然后设置开机启动cmd.sh（在/etc/rc.local中设置）则每次开机之后，会从/dev/ttydata获取到固定端口的数据



这两天在ubuntu中开发跟串口有关程序时，发现来回拔插串口线或者插多个串口线时总是出现串口号ttyUSB*不固定的问题，给应用程序带来不少麻烦，遂google解决。

linux中设备号一般按先后顺序一次向后增大，udev规则文件可以解决这个蛋疼的问题。udev是一种Linux2.6内核采用的/dev 目录的管理系统（可以把它认为是windows中的设备管理器），它通过从sysfs获得的信息，可以提供对特定设备的固定的设备名。sysfs是linux 2.6内核的一种新型文件系统，它提供了当前设备的基本信息。

udev的重要功能就是为为设备提供固定的设备名， 根据Wirting udev rules的详细介绍， udev有如下功能：

 

Rename a device node from the default name to something else

Provide an alternative/persistent name for a device node by creating a symbolic link to the default device node

Name a device node based on the output of a program

Change permissions and ownership of a device node

Launch a script when a device node is created or deleted (typically when a device is attached or unplugged)

Rename network interfaces 简单阅读之后创建文件/etc/udev/rules.d/10-local.rule, 内容如下

 

 

[html] view plain 

<</span>span style="white-space:pre">  </</span>span>KERNEL=="ttyUSB*", ATTRS{idVendor}=="067b", ATTRS{idProduct}=="2303", MODE:="0777", SYMLINK+="user_uart"  

<</span>span style="white-space:pre">  </</span>span>KERNEL=="ttyUSB*", ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="7523", MODE:="0777", SYMLINK+="mcu_uart"  

意思就是匹配sys中内核名为ttyUSB*的设备，属性匹配依据生产商编号idVendor和产品号idProduct, 设定读写权限为0777, 符号链接名为user_uart-----PL2303串口转USB， mcu_uart----CH340串口转USB。
 

idVendor和idProduct由 lsusb  -vvv 命令查看。

保存退出后udev规则就生效了，重新拔插两个串口设备，就可以看到/dev/user_uart指向/dev/ttyUSB0,  /dev/mcu_uart指向/dev/ttyUSB1. 这样以来，我只要在程序里打开/dev/user_uart或/dev/mcu_uart就可以一直准确的打开指定的串口设备了。

PS：如果您的两个串口用的都是PL2303或者都是CH340/CP2102,那就真的无能为力了。。。

常用的匹配类型：

* BUS：匹配总路类型，比如PCI USB等  

  * KERNEL：- 匹配Kernel设备名，比如hda hdb. 

* DRIVER ：- 匹配Kernel的驱动程序名   

* SUBSYSTEM： - 匹配子系统名。   

* ID ：- 匹配总路系统的ID （e.g. PCI bus ID）。

* PLACE ：- 匹配物理位置  （对USB很有用）

资料链接：

