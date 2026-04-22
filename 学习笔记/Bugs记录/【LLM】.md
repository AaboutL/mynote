---
title: 【LLM】
updated: 2025-05-14 03:13:48Z
created: 2025-05-14 02:59:13Z
latitude: 39.91100000
longitude: 116.39500000
altitude: 0.0000
---

# 1. ms_swift+autoawq量化部署问题
1. 量化脚本
	```bash
	ckpt_dir=/data2/tbb/common/hanfy/models/merge_3_1/experiments_0426/learning_2e-05_lora_0_vit_0_num_2_max_1000_2025042620/checkpoint-974
	dataset_file=/data2/tbb/tyt/ts_ms_swift/2025042620.quant.jsonl 
	quant_out=${ckpt_dir}-quant_awq
	
	export PYTHONPATH=$(pwd):$PYTHONPATH
	if [[ -d ${quant_out} ]]; then
	    echo "Directory already exists: ${quant_out}"
	    rm -rf ${quant_out}
	fi
	
	CUDA_VISIBLE_DEVICES=0 swift export \
	    --model ${ckpt_dir} \
	    --max_length 8192 \
	    --quant_batch_size 13 \
	    --system "" \
	    --quant_bits 4 --quant_method awq \
	    --dataset $dataset_file \
	    --output_dir ${quant_out} 
	```
	量化完成，模型部署之后，在进行调用时，出现下面的问题
	```
	Traceback (most recent call last):                                                                         
	  File "/Qwen2.5-VL-32B-Instruct-AWQ/venv/lib/python3.10/site-packages/triton/language/core.py", line 35, in wrapper                                                                     
	    return fn(*args, **kwargs)                                                                             
	  File "/Qwen2.5-VL-32B-Instruct-AWQ/venv/lib/python3.10/site-packages/triton/
	language/core.py", line 1548, in dot                                                                       
	    return semantic.dot(input, other, acc, input_precision, max_num_imprecise_acc, out_dtype, _builder)    
	  File "/Qwen2.5-VL-32B-Instruct-AWQ/venv/lib/python3.10/site-packages/triton/
	language/semantic.py", line 1470, in dot                                                                   
	    assert lhs.dtype == rhs.dtype, f"Both operands must be same dtype. Got {lhs.dtype} and {rhs.dtype}"    
	AssertionError: Both operands must be same dtype. Got fp16 and bf16                                        
	                                                                                                           
	The above exception was the direct cause of the following exception:                                       
	
	  File "/Qwen2.5-VL-32B-Instruct-AWQ/venv/lib/python3.10/site-packages/triton/compiler/compiler.py", line 100, in make_ir
	    return ast_to_ttir(self.fn, self, context=context, options=options, codegen_fns=codegen_fns,
	triton.compiler.errors.CompilationError: at 108:22:
	        masks_s = masks_sk[:, None] & masks_sn[None, :]
	        scales_ptrs = scales_ptr + offsets_s
	        scales = tl.load(scales_ptrs, mask=masks_s)
	        scales = tl.broadcast_to(scales, (BLOCK_SIZE_K, BLOCK_SIZE_N))
	
	        b = (b >> shifts) & 0xF
	        zeros = (zeros >> shifts) & 0xF
	        b = (b - zeros) * scales
	        b = b.to(c_ptr.type.element_ty)
	
	        # Accumulate results.
	        accumulator = tl.dot(a, b, accumulator, out_dtype=accumulator_dtype)
	```

## 解决方法
	```
	pip install git+https://github.com/huggingface/transformers.git@8ee50537fe7613b87881cd043a85971c85e99519
	pip install autoawq==0.2.8 --no-dep
	pip install triton==3.3.0
	```   
这会安装transformer=4.50.0.dev版本。这时需要把不同package的版本对齐。
其中AutoAWQ的0.2.9版本支持Qwen3，但是这个版本的transformers还没有支持。所以需要把AutoAWQ降级到0.2.8。另外，triton也需要升级。
**解决过程**
首先是ms_swift量化脚本无法运行，认为是ms_swift版本太旧，所以重新在新的conda环境中安装了最新的ms_swift，一同安装的transformers版本是4.51.3。这个transformers应该就是部署时的问题来源。