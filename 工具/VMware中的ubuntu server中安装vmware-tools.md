---
title: VMware中的ubuntu server中安装vmware-tools
updated: 2022-07-30 07:28:46Z
created: 2022-07-30 07:21:34Z
latitude: 39.91014100
longitude: 116.35732000
altitude: 0.0000
---

1. 在vmware中点击启动键，然后选择状态栏中VM->Reinstall VMware Tools，在弹出的选项框中，选择OK。
2. ubuntu server系统启动后，运行如下命令
```
mkdir /mnt/vmtools
mount /dev/cdrom /mnt/vmtools
cp /mnt/vmtools/VMwareTools-....tar.gz /tmp
tar -xvf VMwareTools-....tar.gz
cd vmware-tools-distrib
sudo .vmware-install.pl
```
上面的mount可能执行不了，需要在虚拟机的设置中，把CD/DVD(SATA)中的Connected和Connect at power on两个选中。

但是安装好vmware tools后，界面仍然无法调大。