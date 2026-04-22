---
title: 【snpe+qnn】汇总
updated: 2024-12-04 07:26:17Z
created: 2024-11-27 07:04:58Z
latitude: 40.69467810
longitude: 116.56985710
altitude: 0.0000
---

| 标题 | 链接 | 备注 |
| -- | -- | -- |
|SNPE-Benchmark | https://blog.csdn.net/Thinkobj/article/details/128387271 </br> https://zhuanlan.zhihu.com/p/447804052| 包含一些常见bug的解决方法 | 

# 1. SNPE
## 1.1. snpe benchmark使用注意事项
通过在本机运行脚本，会自动把相关数据、模型等传输到开发板上，然后自动在开发板上运行。
### 1.1.1. 环境配置
1. 运行snpe自带的环境检查工具：
	~~~
	source snpe-X.Y.Z/bin/dependencies.sh   # 系统依赖,会自动安装
	source snpe-X.Y.Z/bin/check_python_depends.sh   # python依赖
	~~~
2.  设置环境变量
	~~~
	export SNPE_ROOT=/home/dell/workspace/hanfy/snpe-1.64.0.3605
	export PATH=$SNPE_ROOT/bin/x86_64-linux-clang:$PATH
	export PYTHONPATH=$SNPE_ROOT/lib/python:$PYTHONPATH
	export LD_LIBRARY_PATH=$SNPE_ROOT/lib/x86_64-linux-clang:$LD_LIBRARY_PATH
	~~~
3.  snpe_bench.py运行时需要的配置文件 **.json
   raw数据、image_list.txt、模型会传到手机设备的/data/local/tmp/snpebm/模型名/ 文件夹中。其中会在文件夹下创建raws文件夹，单独保存raw图。所以image_list.txt文件中的图片路径格式：raws/img.raw
	~~~json
	{
	    "Name":"eyelandmark_model",
	    "HostRootPath": "/home/dell/workspace/hanfy/models/benchmark_rst/eyelandmark_model/results", // 本机上保存结果的路径
	    "HostResultsDir":"/home/dell/workspace/hanfy/models/benchmark_rst/eyelandmark_model/results",
	    "DevicePath":"/data/local/tmp/snpebm", // 设备上放置所有benchmark有关的数据、库、模型等的文件夹
	    "Devices":["898a04b"],  // 设备的编号
	    "HostName": "localhost",
	    "Runs":2,
	
	    "Model": {
	        "Name": "eyelandmark_model",
	        "Dlc": "/home/dell/workspace/hanfy/models/exp_13EyeLandmark_FP32_FP32T_1x112x112x1_RGB-IR_V0.0.9.dlc",
	        "InputList": "/home/dell/workspace/hanfy/raws/image_list.txt",
	        "Data": [
	            "/home/dell/workspace/hanfy/raws"
	        ]
	    },
	
	    "Runtimes":["GPU", "CPU"],
	    "Measurements": ["timing", "mem"]
	
	 }
	~~~
5.  指标说明
	1.  **初始化指标度量** (主要是模型推理初始化的各项指标的度量，会根据设置的profiling level的不同，测量的内容会不一样)
		| 指标 | 描述 | 备注 |
		| -- | -- | -- |
		|load|加载模型需要的时间| | 
		|deserialize|反序列化加载的模型需要的时间| | 
		|create|创建snpe网络和初始化给定模型的所有层所花费的时间||
		|profiling| 设置为detailed | 会更详细的分解为下列内容||
		|init | 这个时间包括测量加载、反序列化和创建的时间| |
		|de-init |卸载snpe的时间||
		|create network(s)|创建所有网络所需的总时间|如果网络被分段，会创建多个网络|
		|rpc init time | rpc初始化和加速器启动时间 | 仅适用于dsp和aip|
		|snpe accelerator init time | 准备数据的总时间 | 仅适用于dsp和aip |
		|accelerator init time |仅启动加速器硬件的总时间 |仅适用于dsp和aip|
	
	2. **模型推理度量 **（是模型的一次推理时间的平均度量）
		|指标 | 描述 | 备注 |
		| -- | -- | -- |
		|Total Inference Time |一次推理的全部时间 | 包括输入输出的准备和处理|
		|Forward Propagate | 排除硬件加速器上的overhead |比如仅在GPU内核上的执行时间 |
		|RPC Execute | SNPE调用RPC和加速器上花费的全部时间 |仅适用于DSP和AIP|
		|Snpe Accelerator |SNPE调用硬件加速推理的时间 |仅适用于DSP和AIP |
		|Accelerator |仅在硬件加速器上花费的时间 |仅适用于DSP和AIP|
		|Misc Accelerator |推理中加速器优化带来的时间，比如DSP为了实现最佳性能而增加的附加层所花费的时间。|仅适用于DSP和AIP|

	3. **模型每一层的度量** (每一行度量了模型中每一层的执行时间)
		|avg 平均值 |max 最大值	| min 最小值	| runtime 加速单元|
		|-- | -- | -- | -- |
  		- 对于GPU，DSP，如果设定了 "CpuFallback":true, 那么可能有些不支持的层会运行在CPU上，然后runtime显示是CPU。比如deeplabV3中的最后一层layer_108 (Name:ArgMax Type:argmax).
		- 对于DSP，AIP，有些层的数值是0，那么表示这层和下一层在执行是融合为一个层。比如Conv+ReLu.
		- 对于888和778等新一代平台HTP(DSP)，每一层的数值不在是us，而是cycles，简单来说就是DSP跑了多少个周期。（Time = Cycles/ Frequency）
