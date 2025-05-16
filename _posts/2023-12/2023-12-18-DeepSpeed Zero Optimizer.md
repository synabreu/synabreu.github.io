---
title: "DeepSpeed Zero Optimizer"
date: 2023-12-18
tags: [DeepSpeed, 딥스피드, Microsoft, 마이크로소프트, PyTorch, ZeRO optimizer, Mixed Precision, Model Parallelism, Pipeline Parallelism, DeepSpeed-Inference]
typora-root-url: ../
toc: true
categories: [Microsoft, PyTorch, DeepSpeed, GPU, Distributed Training, Transformer]
---

**ZeRO (Zero Redundancy Optimizer)**는 DeepSpeed 의 핵심 기술로서, **초대규모 모델 학습을 GPU 여러 개로 확장할 수 있게 해주는 기술**이다. 기존의 DataParallel 방식은 각 GPU가 전체 모델과 옵티마이저 상태를 복사해서 쓰기 때문에 메모리 낭비가 심하다. 따라서,  ZeRO는 이 상태 정보를 분산 저장하고 통신을 최소화하여 **메모리 효율성**과 **확장성**을 크게 향상시킨다. 



## 1. ZeRO 단계별 차이

| Stage | 분산 저장 대상                       | 설명                                 |
| ----- | ------------------------------------ | ------------------------------------ |
| **0** | 없음                                 | 일반 DataParallel                    |
| **1** | Optimizer 상태 분산 저장             | 메모리 절약 시작                     |
| **2** | + Gradient 분산 저장                 | 더 많은 절약 가능                    |
| **3** | + 모델 파라미터까지 분산 저장 (Full) | 초대규모 모델 지원 가능, 초기화 복잡 |



## 2. ZeRO Stage 2/3 코드 구현

* `ds_config_zero2.json` (ZeRO Stage 2)

```json
{
  "train_batch_size": 32,
  "gradient_accumulation_steps": 1,
  "fp16": {
    "enabled": true
  },
  "zero_optimization": {
    "stage": 2,
    "allgather_partitions": true,
    "reduce_scatter": true,
    "contiguous_gradients": true,
    "overlap_comm": true
  }
}
```

* `ds_config_zero3.json` (ZeRO Stage 3) - 일반적으로 `offload_optimizer`나 `offload_param`을 `"cpu"`나 `"nvme"`로 설정해 GPU 메모리를 극적으로 줄임

```json
{
  "train_batch_size": 32,
  "gradient_accumulation_steps": 1,
  "fp16": {
    "enabled": true
  },
  "zero_optimization": {
    "stage": 3,
    "offload_optimizer": {
      "device": "none"
    },
    "offload_param": {
      "device": "none"
    },
    "overlap_comm": true,
    "contiguous_gradients": true
  }
}

```

* `train_zero.py` (ZeRO Stage 2/3 공통 코드)

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import deepspeed
import argparse

class SimpleModel(nn.Module):
    def __init__(self, hidden_dim=128):
        super().__init__()
        self.fc1 = nn.Linear(784, hidden_dim)
        self.fc2 = nn.Linear(hidden_dim, 10)

    def forward(self, x):
        x = F.relu(self.fc1(x))
        return self.fc2(x)

def get_data():
    x = torch.randn(32, 784)
    y = torch.randint(0, 10, (32,))
    return x, y

def main(args):
    model = SimpleModel()
    parameters = filter(lambda p: p.requires_grad, model.parameters())

    model_engine, optimizer, _, _ = deepspeed.initialize(
        model=model,
        model_parameters=parameters,
        config=args.deepspeed_config
    )

    for step in range(100):
        x, y = get_data()
        x = x.to(model_engine.device)
        y = y.to(model_engine.device)

        outputs = model_engine(x)
        loss = F.cross_entropy(outputs, y)

        model_engine.backward(loss)
        model_engine.step()

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument('--deepspeed_config', type=str, required=True)
    args = parser.parse_args()
    main(args)

```



## 3. 실행 방법

* ZeRO Stage 2 실행: 

```bash
deepspeed train_zero.py --deepspeed_config ds_config_zero2.json
```

* ZeRO Stage 3 실행: 

```bash
deepspeed --num_gpus=4 train_zero.py --deepspeed_config ds_config_zero3.json
```



## 4. 요약

* ZeRO-2는 옵티마이저 상태 + 그래디언트를 분산 저장

* ZeRO-3는 모델 파라미터까지 분산해 초대규모 LLM 학습 가능

* DeepSpeed 설정 파일만 바꾸면 코드 수정 없이 스테이지 전환 가능

  
