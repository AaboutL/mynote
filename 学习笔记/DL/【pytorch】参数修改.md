---
title: 【pytorch】参数修改
updated: 2024-03-22 12:06:02Z
created: 2024-03-13 17:04:54Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

# 参数修改：
链接： https://www.cnblogs.com/orion-orion/p/16293822.html
| RuntimeError: a view of a leaf Variable that requires grad is being used in an in-place operation.
原因是，如果一个变量要计算梯度，就不要修改它。
你要修改就把requires_grad设置为false。
等修改好了在设置requires_grad为true。
这样不会就不会影响计算梯度。
```python
if mpu.get_data_parallel_rank() == 0:
		for layer in model:
			for name, param in layer.named_parameters():
				param.requires_grad = False
				layer_id = ''
				if len(name.split('.')) > 5:
					layer_id = name.split('.')[5]

				if layer_id in ['4', '6', '8']:
					if 'dense_4h_to_h' in name:
						param[:] = 0
					param.requires_grad = True

				print(f'{name}, {param.size()}, {param.requires_grad}')
				print(param)
```