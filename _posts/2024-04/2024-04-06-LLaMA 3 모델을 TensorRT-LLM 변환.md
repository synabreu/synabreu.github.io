---
title: "LLaMA 3 모델을 TensorRT-LLM 변환"
date: 2024-04-06
tags: [엔비디아, NVIDIA, TensorRT, TensorRT-LLM, Transformer, 트랜스포머, Multi-GPU, 추론, Inference, 텐서 병렬, Tensor Parallelism, 스트리밍 디코딩]
typora-root-url: ../
toc: true
categories: [NVIDIA, TensorRT-LLM, Meta, LLama 3]
---



HuggingFace에서 받은 **LLaMA 3 모델**을 **TensorRT-LLM에서 사용할 수 있도록 변환**하는 코드에 대해 좀더 알아보자. 이 코드는 NVIDIA의 `TensorRT-LLM`에서는 제공된 `convert_checkpoint.py`를 사용하여 모델 가중치를 변환한다. 

우선 전체 과정에 대해 요약하자면, HuggingFace 에서 Safetensors 또는 HF 포맷으로 다운로드한 다음, convert_checkpoint.py 파일 형태로 TensorRT-LLM 포맷으로 변환한다. 그 다음 trtllm-build 인 TensorRT 엔진 빌드하면 된다.



## 1. 디렉토리 구조

```bash
llama3/
├── huggingface_model/       ← HuggingFace 다운로드 완료 (e.g., meta-llama/Llama-3-8B-Instruct)
├── converted_llama3/        ← 변환 후 저장 디렉토리
├── convert_checkpoint.py    ← 변환 스크립트
```



## 2. 모델 가중치 변환을 convert_checkpoint.py 작성

```python
import os
import shutil
from pathlib import Path
from transformers import AutoTokenizer, AutoModelForCausalLM

from tensorrt_llm.models.llama.convert import convert_hf_llama_to_trt_llm

# 1. 설정
huggingface_model_path = "meta-llama/Llama-3-8B-Instruct"     # huggingface model
output_path = "converted_llama3"
dtype = "float16"  # 또는 "bfloat16"

# 2. 출력 디렉토리 생성
os.makedirs(output_path, exist_ok=True)

# 3. 모델 로드 (로컬 저장 또는 HF에서 직접 로딩)
model = AutoModelForCausalLM.from_pretrained(huggingface_model_path, torch_dtype="auto")
tokenizer = AutoTokenizer.from_pretrained(huggingface_model_path)

# 4. 모델 변환 실행
convert_hf_llama_to_trt_llm(
    model=model,
    output_dir=output_path,
    dtype=dtype,
    tokenizer=tokenizer,
    tp_rank=0,  # 텐서 병렬을 쓸 경우 조정
    world_size=1,
)

# 5. tokenizer도 복사
tokenizer.save_pretrained(os.path.join(output_path, "tokenizer"))

print("변환 완료:", output_path)

```



## 3. 필요 패키지 설치

```python
!pip install git+https://github.com/NVIDIA/TensorRT-LLM.git
!pip install transformers accelerate
```



## 4. Hugging Face에서 모델 다운로드

```python
from transformers import AutoTokenizer, AutoModelForCausalLM
model = AutoModelForCausalLM.from_pretrained("meta-llama/Llama-3-8B-Instruct")
tokenizer = AutoTokenizer.from_pretrained("meta-llama/Llama-3-8B-Instruct")
```

* 또는 `transformers-cli login` 후 `huggingface-cli download`를 사용함



## 5. 변환 후 출력 디렉토리

```bash
converted_llama3/
├── model.nemo
├── config.json
├── checkpoint/
│   ├── layer_00.bin
│   ├── ...
├── tokenizer/
│   ├── tokenizer.model
│   └── tokenizer_config.json
```

* Tokenizer 디렉토리 준비됨 (보통 `tokenizer/`)
* GPU: A100, H100 또는 L40 필요 (Ampere 이상)



## 6. 단일 GPU일 때 trtllm-build 명령

```bash
trtllm-build \
  --checkpoint_dir ./converted_llama3 \
  --tokenizer_dir ./tokenizer \
  --output_dir ./engine \
  --dtype float16 \
  --max_batch_size 1 \
  --max_input_len 1024 \
  --max_output_len 128 \
  --enable_fp16
```



## 7. trtllm-build 주요 옵션

| 옵션               | 설명                                                         |
| ------------------ | ------------------------------------------------------------ |
| `--checkpoint_dir` | 변환된 모델 가중치 폴더                                      |
| `--tokenizer_dir`  | tokenizer.model, tokenizer_config.json 등                    |
| `--output_dir`     | TensorRT 엔진(plan 파일) 저장 경로                           |
| `--dtype`          | 추론 정밀도 (`float16`, `bfloat16`, `float32`, `fp8` 중 선택) |
| `--max_input_len`  | 허용할 최대 입력 토큰 수                                     |
| `--max_output_len` | 생성할 최대 출력 토큰 수                                     |
| `--enable_fp16`    | FP16 엔진 최적화 활성화                                      |