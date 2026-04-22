---
title: 【LLM】显存优化
updated: 2024-07-18 12:30:16Z
created: 2024-07-18 12:26:38Z
latitude: 39.91014100
longitude: 116.35732000
altitude: 0.0000
---

# 通过参数设置提高显存的利用率，显存不足的
1. 设置huggingface transformers框架中的 --gradient_checkpointing=True
2. 设置环境变量：export PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True