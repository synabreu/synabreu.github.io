---
title: "[실습] NVIDIA 고성능 딥러닝 추론 최적화 및 실행 엔진, TensorRT"
date: 2024-04-03
tags: [메타, llama, FSDP, PyTorch, ZeRO optimizer, Mixed Precision, Model Parallelism, Pipeline Parallelism, DeepSpeed-Inference, DDP, Distributed Data Parallel]
typora-root-url: ../
toc: true
categories: [NVIDIA,TensorRT]
---



**TensorRT**는 NVIDIA가 개발한 **고성능 딥러닝 추론 최적화 및 실행 엔진**이다. 주로 **GPU를 활용한 딥러닝 모델의 추론 속도를 높이기 위해 사용**되며, 실시간 추론과 같은 지연 시간이 중요한 환경에서 강력한 성능을 발휘한다. TensorRT에 대해 다음과 같이 정리해 보자면?



## 1. 핵심 요약

| 항목                | 설명                                            |
| ------------------- | ----------------------------------------------- |
| **개발사**          | NVIDIA                                          |
| **주요 용도**       | 딥러닝 모델 추론 속도 최적화 (GPU 기반)         |
| **지원 프레임워크** | TensorFlow, PyTorch, ONNX 등                    |
| **출력 대상**       | NVIDIA GPU (Ampere, Hopper, Xavier 등)          |
| **주요 기능**       | FP16/INT8 정밀도 변환, 레이어 통합, 배치 최적화 |



## 2. TensorRT의 주요 특징

* **정밀도 최적화 (Precision Calibration)**: FP32 → FP16 또는 INT8로 자동 변환해 속도 및 메모리 효율 극대화.
* **그래프 최적화**: 불필요한 연산 제거, 레이어 병합 등을 통해 연산량 축소.

* **레이턴시 감소**: CUDA를 활용해 연산 스케줄링을 최적화하고 레이턴시를 줄임.

* **배치 최적화**: 배치 크기 설정에 따라 다이나믹 최적화 지원 (Dynamic Shape).

* **ONNX 호환**: ONNX 형식으로 export된 모델을 쉽게 import하고 최적화 가능.

```bash
## 사용 예시 (PyTorch → ONNX → TensorRT)

# PyTorch 모델을 ONNX로 변환
torch.onnx.export(model, input_tensor, "model.onnx")

# ONNX 모델을 TensorRT 엔진으로 변환 (trtexec 사용)
trtexec --onnx=model.onnx --saveEngine=model.trt --fp16
```



## 3. 예제 시나리오

* 모델: HuggingFace에서 변환한 **LLaMA 또는 GPT2 모델 (ONNX 형식)**
* 환경: NVIDIA GPU가 설치된 Linux, Python 3.8+, CUDA & TensorRT 설치
* 입력: 토큰화된 텍스트 → ONNX → TensorRT 추론
* 출력: 토큰 결과 → 디코딩



## 4. 사전 준비

```bash
# ONNX → TensorRT 엔진 변환 (fp16 최적화)
trtexec --onnx=gpt2.onnx --saveEngine=gpt2_fp16.trt --fp16
```



## 5. Python 추론 (TensorRT + ONNX Runtime)

```python
import tensorrt as trt
import pycuda.driver as cuda
import pycuda.autoinit
import numpy as np
from transformers import GPT2Tokenizer

# Load tokenizer
tokenizer = GPT2Tokenizer.from_pretrained("gpt2")

# 입력 문장을 토큰화
input_text = "The future of AI is"
input_ids = tokenizer.encode(input_text, return_tensors="np").astype(np.int32)

# TensorRT runtime 초기화
TRT_LOGGER = trt.Logger(trt.Logger.WARNING)
with open("gpt2_fp16.trt", "rb") as f, trt.Runtime(TRT_LOGGER) as runtime:
    engine = runtime.deserialize_cuda_engine(f.read())
    context = engine.create_execution_context()

# 메모리 할당
input_shape = input_ids.shape
output_shape = (input_shape[0], 32, 50257)  # 예시: (batch_size, seq_len, vocab_size)
input_nbytes = input_ids.nbytes
output_nbytes = np.prod(output_shape) * np.float16().nbytes

# GPU 메모리 할당
d_input = cuda.mem_alloc(input_nbytes)
d_output = cuda.mem_alloc(output_nbytes)

# Host → Device 복사
cuda.memcpy_htod(d_input, input_ids)

# 추론 실행
bindings = [int(d_input), int(d_output)]
context.execute_v2(bindings)

# 결과 가져오기
output = np.empty(output_shape, dtype=np.float16)
cuda.memcpy_dtoh(output, d_output)

# 결과 디코딩 (가장 높은 확률 토큰)
generated_ids = np.argmax(output[0], axis=-1)
generated_text = tokenizer.decode(generated_ids)

print("Generated Text:", generated_text)
```



## 6. 주의 사항

* 이 코드는 **GPT2 기준의 ONNX → TensorRT 변환** 기반이며, 구조에 따라 shape, output handling은 조정 필요
* LLaMA 계열 모델을 TensorRT로 추론하려면 `NVIDIA TensorRT-LLM` 패키지나 `TensorRT-LLM` Python wrapper를 사용하는 것이 더 일반적
* NVIDIA TensorRT 공식 홈페이지: https://developer.nvidia.com/tensorrt



