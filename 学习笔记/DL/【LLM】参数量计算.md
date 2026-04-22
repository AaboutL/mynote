---
title: 【LLM】参数量计算
updated: 2024-03-15 09:31:05Z
created: 2024-03-13 07:27:54Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
tags:
  - llm
---

# 1. 代码
config: /store3/maowy/models/starcoder2-15b/config.json
```json 
{
  "_name_or_path": "/aot/checkpoints/15b/config.json",
  "architectures": [
    "Starcoder2ForCausalLM"
  ],
  "attention_dropout": 0.0,
  "bos_token_id": 0,
  "eos_token_id": 0,
  "hidden_act": "gelu_pytorch_tanh",
  "hidden_size": 6144,
  "initializer_range": 0.01275,
  "intermediate_size": 24576,
  "max_position_embeddings": 16384,
  "mlp_type": "default",
  "model_type": "starcoder2",
  "norm_epsilon": 1e-05,
  "norm_type": "layer_norm",
  "num_attention_heads": 48,
  "num_hidden_layers": 40,
  "num_key_value_heads": 4,
  "rms_norm_eps": 1e-05,
  "rope_theta": 100000,
  "sliding_window": 4096,
  "tie_word_embeddings": false,
  "torch_dtype": "float32",
  "transformers_version": "4.37.0.dev0",
  "use_bias": true,
  "use_cache": true,
  "vocab_size": 49152
}
```

```python
import os
import json

ckpt_path = "/store3/maowy/models/starcoder2-15b"
# ckpt_path = "/store3/hanfy/workspace/models/ts/3B/linyizuo-sft_3B_pruned-48960_stage2_v011x2_stage3_v012x3_nocat_s4k_714step_hf"

config_path = os.path.join(ckpt_path, "config.json")
config = json.load(open(config_path))

hidden_size = config['hidden_size']
intermediate_size = config['intermediate_size']
num_attention_heads = config['num_attention_heads']
num_hidden_layers = config['num_hidden_layers']
num_key_value_heads = config['num_key_value_heads']
vocab_size = config['vocab_size']
max_position_embeddings = config['max_position_embeddings']
hidden_act = config['hidden_act']
use_bias = config.get('use_bias', False)

padded_vocab_size = vocab_size
make_vocab_size_divisible_by = 128
tensor_model_parallel_size = 1

token_emb_size = vocab_size * hidden_size
position_emb_size = max_position_embeddings * hidden_size

attention_norm_weight_size1 = hidden_size
attention_norm_bias_size1 = hidden_size if use_bias else 0

attention_qkv_weight_size = hidden_size * hidden_size + 2 * hidden_size * hidden_size / num_attention_heads * num_key_value_heads
attention_qkv_bias_size = (hidden_size + 2 * hidden_size / num_attention_heads * num_key_value_heads) if use_bias else 0

attention_dense_weight_size = hidden_size * hidden_size
attention_dense_bias_size = hidden_size  if use_bias else 0

attention_norm_weight_size2 = hidden_size
attention_norm_bias_size2 = hidden_size if use_bias else 0

if hidden_act in ['swiglu', 'silu']:
    attention_mlp_dense_size = hidden_size * (intermediate_size * 2) + intermediate_size * hidden_size
else:
    attention_mlp_dense_size = hidden_size * intermediate_size * 2
attention_mlp_bias_size = (intermediate_size + hidden_size) if use_bias else 0

transformer_size = (attention_norm_weight_size1 + attention_norm_bias_size1 +
                    attention_qkv_weight_size + attention_qkv_bias_size +
                    attention_dense_weight_size + attention_dense_bias_size + 
                    attention_norm_weight_size2 + attention_norm_bias_size2 +
                    attention_mlp_dense_size + attention_mlp_bias_size) * num_hidden_layers


final_norm_weight_size = hidden_size
final_norm_bias_size = hidden_size if use_bias else 0
final_norm_size = final_norm_weight_size + final_norm_bias_size

total_size = token_emb_size * 2 + transformer_size + final_norm_size
print(f"total_size:                {total_size / 1e9}B ({total_size})")
print(f"transformer size:          {transformer_size / 1e9}B ({transformer_size})")
print(f"embedding + lm_head size:  {(token_emb_size * 2 + final_norm_size) / 1e9}B ({token_emb_size * 2 + final_norm_size})")
```

# 2. 链接
https://zhuanlan.zhihu.com/p/624740065