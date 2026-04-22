---
title: >-
  【三维重建】gipuma-Massively Parallel Multiview Stereopsis by Surface Normal
  Diffusion
updated: 2023-07-13 02:56:34Z
created: 2023-07-13 02:42:29Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

github：https://github.com/kysucix/gipuma
# 1. 配置
需要调用CUDA，所以如果没有cudatoolkit，则需要安装。
安装的过程中，可能会提示nvidia的驱动已经存在，那么需要把驱动卸载掉，然后重新安装cudatoolkit，此时会把nvidia驱动再次安装。
## nvidia驱动卸载步骤：
当您在安装 NVIDIA 驱动程序时遇到 "An NVIDIA kernel module 'nvidia-drm' appears to already be loaded in your kernel" 错误时，这通常表示内核模块已经加载，可能是与当前安装的 NVIDIA 驱动程序冲突。要解决此问题，请按照以下步骤操作：
1. 禁用并停止图形界面
在 Ubuntu 系统中，您可以通过 Ctrl+Alt+F1（在某些情况下可能是 Ctrl+Alt+F2 或 Ctrl+Alt+F3）进入文本模式。接下来，运行以下命令停止图形界面：
	~~~
	bash
	sudo systemctl isolate multi-user.target
	~~~
	在 CentOS 或 RHEL 系统中，使用以下命令停止图形界面：
	~~~
	bash
	sudo systemctl stop lightdm
	~~~
2. 卸载现有的 NVIDIA 驱动程序
	卸载已安装的 NVIDIA 驱动程序：
	~~~
	bash
	sudo apt-get purge nvidia*
	~~~
	对于 CentOS 或 RHEL 系统：
	~~~
	bash
	sudo yum remove nvidia*
	~~~
3. 卸载 NVIDIA 内核模块
	使用以下命令卸载 nvidia-drm 内核模块：
	~~~
	bash
	sudo modprobe -r nvidia-drm
	~~~
	确保其他 NVIDIA 相关模块也被卸载（如果已加载）：
	~~~
	bash
	sudo modprobe -r nvidia
	sudo modprobe -r nvidia-uvm
	~~~
4. 安装新的 NVIDIA 驱动程序
	现在您可以按照 NVIDIA 官方的安装指南重新安装 NVIDIA 驱动程序。确保您安装了适合您 GPU 型号的驱动程序。

5. 重新启动计算机
	安装完毕后，请使用以下命令重启计算机：
	~~~
	bash
	sudo reboot
	~~~
完成重启后，新的 NVIDIA 驱动程序应已成功安装且不再显示错误。如果问题仍然存在，请提供更多关于错误的详细信息，以便我们更好地帮助您解决问题。

# 2. 源码编译
1. CMakeLists.txt中，对下面代码进行确认，确保cuda和gpu的版本对应：
~~~
set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS};-O3 --use_fast_math --ptxas-options=-v --compiler-options -Wall -gencode arch=compute_50,code=sm_50 -gencode arch=compute_52,code=sm_52 -gencode arch=compute_53,code=sm_53 -gencode arch=compute_60,code=sm_60 -gencode arch=compute_61,code=sm_61 -gencode arch=compute_62,code=sm_62 -gencode arch=compute_70,code=sm_70 -gencode arch=compute_72,code=sm_72 -gencode arch=compute_75,code=sm_75)
~~~
