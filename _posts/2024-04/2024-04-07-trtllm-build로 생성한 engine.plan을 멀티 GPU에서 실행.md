---
title: "2024-0trtllm-build로 생성한 engine.plan을 멀티 GPU에서 실행"
date: 2024-04-07
tags: [엔비디아, NVIDIA, TensorRT, TensorRT-LLM, Transformer, 트랜스포머, Multi-GPU, 추론, Inference, 텐서 병렬, Tensor Parallelism, 스트리밍 디코딩]
typora-root-url: ../
---



계속해서 TensorRT-LLM에 대해 알아보고 있는 데, trtllm-build`로 생성한 `engine.plan`을 멀티 GPU에서 병렬로 실행하는 방법에 대해 좀 더 알아보자! 참고로 이 방법은 NVIDIA TensorRT-LLM의 텐서 병렬 (Tensor Parallelism) 기능을 사용해야 한다.



## 1. Tensor Parallel용 `engine.plan` 생성

```bash
trtllm-build \
  --checkpoint_dir ./converted_llama3 \
  --tokenizer_dir ./tokenizer \
  --output_dir ./engine \
  --dtype float16 \
  --max_batch_size 1 \
  --max_input_len 1024 \
  --max_output_len 128 \
  --enable_fp16 \
  --tp_size 2
```

* `trtllm-build` 시 `--tp_size` 옵션을 통해 GPU 수에 맞는 텐서 병렬 설정을 해야 함
* `--tp_size 2` → 2개의 GPU에서 병렬 실행을 위한 plan 생성



## 2. TensorRT-LLM Runtime API을 이용한 멀티 GPU 추론하기(multigpu-infer.py)

```python
import torch
from tensorrt_llm.runtime import ModelRunnerBuilder
from transformers import LlamaTokenizer

# 디바이스 개수 확인
num_gpus = torch.cuda.device_count()
assert num_gpus >= 2, "두 개 이상의 GPU가 필요합니다."

# Tokenizer 로드
tokenizer = LlamaTokenizer.from_pretrained("./tokenizer")
prompt = "Explain quantum computing in simple terms."
input_ids = tokenizer.encode(prompt, return_tensors="pt")

# 텐서 병렬을 위한 RunnerBuilder 생성
builder = ModelRunnerBuilder()
builder.load("engine", rank=0, world_size=2)  # rank는 아래에서 fork로 실행됨

# 추론 실행 (TP 병렬)
with builder.build() as runner:
    outputs = runner.generate(
        input_ids=input_ids,
        max_new_tokens=64,
        temperature=0.7,
        top_p=0.9,
    )
    result = tokenizer.decode(outputs[0], skip_special_tokens=True)
    print("Generated:", result)
```



## 3. 멀티 GPU 실행 (예: 2GPU 기준)

```bash
torchrun --nproc_per_node=2 multigpu-infer.py
```

* TensorRT-LLM은 내부적으로 `torch.distributed` 기반 MPI 방식으로 rank마다 실행해야 하므로, `torchrun`을 사용함



## 4. 주의 사항

| 항목        | 설명                                                         |
| ----------- | ------------------------------------------------------------ |
| `--tp_size` | `torchrun`에서 사용하는 `--nproc_per_node`와 반드시 동일해야 함 |
| `rank`      | `builder.load("engine", rank=RANK, world_size=WORLD_SIZE)` 형태로 각 프로세스별 지정 필요 |
| 토크나이저  | 병렬로 접근 가능하도록 사전에 공유 디렉토리에 저장           |
| GPU 메모리  | 각 GPU가 8B 또는 13B 이상 모델을 처리할 수 있는 충분한 메모리가 필요 |



## 5. 요점 정리

| 단계      | 명령 / 설명                                       |
| --------- | ------------------------------------------------- |
| 모델 변환 | `convert_checkpoint.py`                           |
| 엔진 빌드 | `trtllm-build --tp_size=N`                        |
| 실행      | `torchrun --nproc_per_node=N infer.py`            |
| 병렬 설정 | `builder.load(..., rank=R, world_size=N)` 로 구성 |
