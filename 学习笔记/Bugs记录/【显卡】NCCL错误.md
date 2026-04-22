---
title: 【显卡】NCCL错误
updated: 2024-06-11 06:17:48Z
created: 2024-03-26 11:56:30Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

## 1. Bootstrap : no socket interface found 解决
```
torch.distributed.DistBackendError: NCCL error in: ../torch/csrc/distributed/c10d/ProcessGroupNCCL.cpp:1251, internal error - please report this issue to the NCCL developers, NCCL version 2.18.6
ncclInternalError: Internal check failed.
Last error:
Bootstrap : no socket interface found
```

### 解决方法：
```bash
export NCCL_SOCKET_IFNAME=en,eth,em,bond
```
---

## 2. Some NCCL operations have failed or timed out. Due to the asynchronous nature of CUDA kernels, subsequent GPU operations might run on corrupted/incomplete data.
```
[rank7]:[E ProcessGroupNCCL.cpp:1537] [PG 0 Rank 7] Timeout at NCCL work: 1, last enqueued NCCL work: 1, last completed NCCL work: 3452013982.
[rank7]:[E ProcessGroupNCCL.cpp:577] [Rank 7] Some NCCL operations have failed or timed out. Due to the asynchronous nature of CUDA kernels, subsequent GPU operations might run on corrupted/incomplete data.
[rank7]:[E ProcessGroupNCCL.cpp:583] [Rank 7] To avoid data inconsistency, we are taking the entire process down.
[rank7]:[E ProcessGroupNCCL.cpp:1414] [PG 0 Rank 7] Process group watchdog thread terminated with exception: [Rank 7] Watchdog caught collective operation timeout: WorkNCCL(SeqNum=1, OpType=BROADCAST, NumelIn=1, NumelOut=1, Timeout(ms)=600000) ran for 600017 milliseconds before timing out.
Exception raised from checkTimeout at ../torch/csrc/distributed/c10d/ProcessGroupNCCL.cpp:565 (most recent call first):
frame #0: c10::Error::Error(c10::SourceLocation, std::string) + 0x57 (0x7f94ddb56897 in /home/hanfy/anaconda3/envs/mmyolo/lib/python3.10/site-packages/torch/lib/libc10.so)
```
### 原因和解决方法：
原因：数据不一致，有可能是数据增强时，某一个样本的label没有了；尺寸无法对齐等
解决方法：检查配置文件的命名拼写、配置文件中数据集的路径拼写、检查数据，关闭数据增强，更换其他数据实验