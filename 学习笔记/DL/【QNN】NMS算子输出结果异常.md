---
title: 【QNN】NMS算子输出结果异常
updated: 2024-06-27 02:37:17Z
created: 2024-06-27 02:35:53Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

# 原因：SIMD指令对于argmax、nms等类型的操作不友好

确实，SIMD（Single Instruction, Multiple Data）架构在某些操作上表现非常出色，例如加法、乘法等简单的算术运算，但对于 `argmax` 这样的操作，使用 SIMD 在实现上就显得不太友好。这主要是因为 `argmax` 操作涉及到寻找最大值元素的索引，而这个过程通常包含条件判断和索引操作，这些操作很难在 SIMD 指令集的并行计算模式下高效实现。

### 为什么 SIMD 对 `argmax` 操作不友好

1. **条件判断**：`argmax` 操作需要逐个比较元素，并记录当前最大值的索引。这种条件判断和记录过程不太适合 SIMD 的并行计算模型。
2. **索引操作**：`argmax` 不仅需要找到最大值，还需要记录最大值对应的索引位置。SIMD 擅长对数据进行并行计算，但对记录和索引的管理并不高效。
3. **数据相关性**：`argmax` 操作具有数据相关性，即每一步的比较结果会影响到后续的操作，这与 SIMD 的独立并行计算模式相矛盾。

### 实现 `argmax` 的常用方法

尽管 SIMD 对 `argmax` 操作不太友好，但我们依然可以使用一些技巧来提高 `argmax` 的性能。例如，通过分块处理、利用 SIMD 加速部分计算等。

#### 示例：使用 SIMD 加速部分计算

以下是一个使用 SIMD 加速部分计算的 `argmax` 实现示例，主要思想是先使用 SIMD 找到每个块中的最大值，然后在块之间进行普通比较。

##### 使用 AVX2 实现部分加速的 `argmax`

假设我们使用 AVX2 指令集处理单精度浮点数数组：

```cpp
#include <immintrin.h>
#include <iostream>

int argmax_avx2(float* array, int length) {
    const int simd_width = 8;  // AVX2 处理 8 个单精度浮点数
    __m256 max_values = _mm256_loadu_ps(array);
    __m256i max_indices = _mm256_set_epi32(7, 6, 5, 4, 3, 2, 1, 0);

    for (int i = simd_width; i < length; i += simd_width) {
        __m256 values = _mm256_loadu_ps(&array[i]);
        __m256i indices = _mm256_set_epi32(i + 7, i + 6, i + 5, i + 4, i + 3, i + 2, i + 1, i + 0);

        __m256 mask = _mm256_cmp_ps(values, max_values, _CMP_GT_OS);
        max_values = _mm256_blendv_ps(max_values, values, mask);
        max_indices = _mm256_blendv_epi8(max_indices, indices, _mm256_castps_si256(mask));
    }

    float max_value = array[0];
    int max_index = 0;
    alignas(32) float max_values_array[simd_width];
    alignas(32) int max_indices_array[simd_width];
    _mm256_store_ps(max_values_array, max_values);
    _mm256_store_si256((__m256i*)max_indices_array, max_indices);

    for (int i = 0; i < simd_width; ++i) {
        if (max_values_array[i] > max_value) {
            max_value = max_values_array[i];
            max_index = max_indices_array[i];
        }
    }

    return max_index;
}

int main() {
    float array[] = {0.1f, 1.2f, 2.3f, 3.4f, 4.5f, 3.4f, 2.3f, 1.2f, 5.6f, 4.3f};
    int length = sizeof(array) / sizeof(array[0]);
    int max_index = argmax_avx2(array, length);
    std::cout << "Index of max element: " << max_index << std::endl;
    return 0;
}
```

### 结论

- **数据相关性**：`argmax` 操作具有较强的数据相关性，这与 SIMD 的并行计算模式不匹配。
- **部分加速**：尽管 SIMD 对 `argmax` 操作不太友好，但我们依然可以通过分块处理和部分加速来提高性能。
- **实际应用**：在实际应用中，可以结合使用 SIMD 和常规方法来实现高效的 `argmax` 操作。

通过上述方法，我们可以在一定程度上利用 SIMD 的并行计算能力来加速 `argmax` 操作，但由于其固有的条件判断和索引操作，完全依赖 SIMD 实现 `argmax` 依然具有挑战性。

# SIMD
SIMD（Single Instruction, Multiple Data）是一种计算机架构的指令集，它允许在一个指令周期内对多个数据元素进行并行处理。SIMD架构非常适合于对大量数据进行相同操作的任务，如多媒体处理、图像处理、科学计算和机器学习等。

### SIMD简介

- **单指令**：即单条指令（Single Instruction）被发布。
- **多数据**：多个数据元素（Multiple Data）同时被处理。

SIMD通常用于向量化操作，处理器可以在一个时钟周期内执行相同的操作（如加法、乘法）在多个数据元素上。这种并行处理方式能够显著提高计算密集型应用的性能。

### SIMD指令集

不同处理器架构提供不同的SIMD指令集。例如：

- **Intel**:
  - **MMX**: 最早的SIMD指令集，主要用于多媒体处理。
  - **SSE（Streaming SIMD Extensions）**: 包括SSE、SSE2、SSE3、SSSE3、SSE4.1、SSE4.2等。
  - **AVX（Advanced Vector Extensions）**: 包括AVX、AVX2、AVX-512等。
- **ARM**:
  - **NEON**: ARM架构的SIMD指令集。
- **IBM PowerPC**:
  - **AltiVec**: 或称为VMX，是PowerPC处理器的SIMD指令集。

### 示例

以下是一些常见的SIMD指令集和它们的应用示例：

#### Intel SSE指令集

SSE指令集允许在128位寄存器上进行并行操作。例如，SSE可以在单条指令中对四个单精度浮点数进行加法操作。

```c
#include <xmmintrin.h>  // SSE指令集头文件

void add_sse(float* a, float* b, float* c) {
    __m128 vec_a = _mm_loadu_ps(a);  // 加载4个单精度浮点数到SSE寄存器
    __m128 vec_b = _mm_loadu_ps(b);  // 加载4个单精度浮点数到SSE寄存器
    __m128 vec_c = _mm_add_ps(vec_a, vec_b);  // 逐元素相加
    _mm_storeu_ps(c, vec_c);  // 将结果存储回内存
}
```

#### Intel AVX指令集

AVX指令集扩展了SSE，允许在256位寄存器上进行并行操作。例如，AVX可以在单条指令中对八个单精度浮点数进行加法操作。

```c
#include <immintrin.h>  // AVX指令集头文件

void add_avx(float* a, float* b, float* c) {
    __m256 vec_a = _mm256_loadu_ps(a);  // 加载8个单精度浮点数到AVX寄存器
    __m256 vec_b = _mm256_loadu_ps(b);  // 加载8个单精度浮点数到AVX寄存器
    __m256 vec_c = _mm256_add_ps(vec_a, vec_b);  // 逐元素相加
    _mm256_storeu_ps(c, vec_c);  // 将结果存储回内存
}
```

#### ARM NEON指令集

NEON是ARM处理器上的SIMD指令集，允许在128位寄存器上进行并行操作。

```c
#include <arm_neon.h>  // NEON指令集头文件

void add_neon(float32_t* a, float32_t* b, float32_t* c) {
    float32x4_t vec_a = vld1q_f32(a);  // 加载4个单精度浮点数到NEON寄存器
    float32x4_t vec_b = vld1q_f32(b);  // 加载4个单精度浮点数到NEON寄存器
    float32x4_t vec_c = vaddq_f32(vec_a, vec_b);  // 逐元素相加
    vst1q_f32(c, vec_c);  // 将结果存储回内存
}
```

### SIMD的优势

- **高性能**：SIMD指令可以在单个时钟周期内处理多个数据元素，大幅提高数据并行处理的性能。
- **能效高**：相比于传统的逐元素操作，SIMD能够有效降低能耗，因为它减少了指令发射的次数。
- **广泛应用**：SIMD指令在多媒体处理、图像处理、科学计算、机器学习等领域有着广泛的应用。

### 总结

SIMD是一种强大的计算机架构指令集，允许在单个时钟周期内对多个数据元素进行并行处理。不同的处理器架构提供了不同的SIMD指令集，如Intel的SSE和AVX指令集、ARM的NEON指令集等。通过利用SIMD指令，可以大幅提升数据并行处理任务的性能和能效。