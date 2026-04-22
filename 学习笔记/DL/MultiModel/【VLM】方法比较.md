---
title: 【VLM】方法比较
updated: 2023-10-30 14:16:13Z
created: 2023-10-29 02:18:49Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

| 方法 | 语言模型 | 视觉模型 | 对齐方式 | 公共数据 | 自建数据 | 训练策略 | 训练资源 | 
| -- | -- | -- | -- | -- | -- | -- | -- |
|LLaVA | Vicuna | CLIP ViT-L/14 | one linear projection layer |1. 预训练：CC3M->595K; </br> | 1. 使用ChatGPT/GPT-4标注，prompt的形式：（1）从多个角度对图片的描述；（2）bbox：空间位置和物体属性 </br> 2. 数据：COCO中挑选的数据 </br> 3. 数据类型：三种类型：（1）对话；（2）详细描述；（3）复杂推理 </br> 4. 数据量：总共158K：对话：58K，详细描述：23K，复杂推理：77K|Stage1: 固定visual encoder和LLM的权重，训练linear projection layer；</br> Stage2：固定visual encode，优化linear projectin layer和LLM的权重 | |
|LLaVA-1.5 | Vicuna-7B/13B | CLIP-ViT-L-336px | MLP projection | 1. 预训练：LCS-558K；</br> 2. 指令调优：open-knowledge（VQAOKVQA、A-OKVQA）、OCR（OCRVQA、TextCaps）、region-level VQA（Visual Genome、RefCOCO）、GQA、ShareGPT data||| 8 A100，1 day |
|MiniGPT-4 | Vicuna | EVA-CLIP ViT-G/14 + Q-Former | a single projection layer |1. 预训练：LAION、Conceptual Captions、SBU；</br> 2. 指令调优：3,500detailed image description pairs | ###Human: <Img><ImageFeature></Img>Describe this image in detail. Give as many details as possible. Say everything you see. ###Assistant: | 固定Visual encoder 和 LLM的权重，只训练投影层| Stage1: 4 A100，20k steps, batchsize=256, 8 hours </br> Stage2: 1 A100, 400 steps, batchsize=12, 7 minutes|
|MiniGPT-V2| LLaMA2-chat (7B) | EVA-CLIP ViT-G/14 |a linear projection layer | 1. weakly-labeled datasets：LAION、CC3M、SBU、GRIT-20M from Kosmos v2（REC、REG）</br> 2. fine-grained datasets: COCO caption、Text Captions、RefCOCO、RefCOCO+、RefCOCOg(REC)、restructured the data from ReferCOCO and its variants（REG)；VQA datasets：GQA、VQA-v2、OCR-VQA、OK-VQA、AOK-VQA。</br> 3. instruction datasets：LLaVA instruction data（23K+58K)、Flicker 30k| 1. Mixing multi-task dataset：混合不同任务的数据创建了多轮对话数据集 </br> 2. Unnatural instruction：helping recover the language generation ability| 1. 448x448(2x2->1) </br> 2.三阶段训练：weakly-labeled dataset、fine-grained image-text dataset、multi-modal指令数据 </br> 3. 整个训练过程固定visual encoder </br> 4. 增加任务标识符 </br> 5. 增加空间位置数据 </br> 6. finetune LLM with LoRA| Stage1: 8 A100，400K steps, bs=96, lr=1e-4, 90 hours </br> Stage2: 4 A100, 50K steps, bs=64, lr=1e-5, 20 hours </br> Stage3: 4 A100, 35K steps, bs=24, lr=1e-5, 7 hours |
|BLIP | | | | | | | |
|BLIP2| | | | | | | |