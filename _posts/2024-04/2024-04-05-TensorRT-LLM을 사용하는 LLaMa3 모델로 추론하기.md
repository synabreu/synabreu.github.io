---
title: "TensorRT-LLM을 사용하는 LLaMa3 모델로 추론하기"
date: 2024-04-05
tags: [엔비디아, NVIDIA, TensorRT, TensorRT-LLM, Transformer, 트랜스포머, Multi-GPU, 추론, Inference, 텐서 병렬, Tensor Parallelism, 스트리밍 디코딩]
typora-root-url: ../
toc: true
---

NVIDIA의 [TensorRT-LLM](https://github.com/NVIDIA/TensorRT-LLM)을 사용하여 **LLaMA 3 모델**을 추론하는 **Python 예제 코드**를 작성해 보자! 이 코드는 TensorRT-LLM Python API 기반이며, LLM 구조에 최적화된 방식으로 작성했다. 

## 1. 전제 조건  

* NVIDIA GPU (Ampere 이상, A100/H100 추천)
* TensorRT-LLM 설치 가이드: [https://github.com/NVIDIA/TensorRT-LLM#installation](https://github.com/NVIDIA/TensorRT-LLM#installation)
* HuggingFace에서 변환된 LLaMA 3 모델 (ex: `meta-llama/Llama-3-8B-Instruct`)
* `convert_checkpoint.py`로 TensorRT-LLM 변환된 모델
* 추론 전 TensorRT 엔진 생성 완료 (`trtllm.build_engine()`)

## 2. 디렉토리 구성  

* NVIDIA GPU (Ampere 이상, A100/H100 추천)

```text
llama3-tensorrtllm/
├── engine/
│   └── llama3-8b-plan.plan
├── tokenizer/
│   └── tokenizer.model
├── infer.py
```

## 3. 추론 소스 코드 (llama3-inference.py)  

```python
from tensorrt_llm.runtime import ModelRunner
from transformers import LlamaTokenizer
import torch

# 1. Tokenizer 준비
tokenizer = LlamaTokenizer.from_pretrained("./tokenizer")

# 2. 입력 문장 토큰화
prompt = "대한민국의 수도는"
input_ids = tokenizer.encode(prompt, return_tensors="pt")

# 3. TensorRT-LLM 모델 로딩
runner = ModelRunner(engine_dir="./engine", tokenizer=tokenizer)

# 4. 추론 실행
output = runner.generate(
    input_ids=input_ids,
    max_new_tokens=64,
    temperature=0.7,
    top_p=0.9,
)

# 5. 디코딩
output_text = tokenizer.decode(output[0], skip_special_tokens=True)
print("Generated:", output_text)

```

## 4. TensorRT-LLM 엔진 빌드 명령

```bash
trtllm-build \
  --checkpoint_dir ./converted-llama3 \
  --tokenizer_dir ./tokenizer \
  --output_dir ./engine \
  --dtype float16 \
  --max_batch_size 1 \
  --max_input_len 1024 \
  --max_output_len 128 \
  --enable_fp16

```

## 5. 요약

* TensorRT-LLM은 일반적인 `torch`나 `onnx`와는 달리 **KV 캐시, 텐서 병렬** 등에 최적화되어 있어 초거대 모델 추론 성능이 매우 뛰어나다.

* Multi-GPU 추론, 텐서 병렬(Tensor Parallelism), 스트리밍 디코딩도 지원

  

