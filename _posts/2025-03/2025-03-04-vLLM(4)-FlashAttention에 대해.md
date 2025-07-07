---
title: "vLLM(4)-FlashAttention에 대해"
date: 2025-03-04
tags: [LLM, vLLM, OpenSource, 오픈소스, FlashAttetnion]
typora-root-url: ../
toc: true
categories: [vLLM, OpenSource]
---

vLLM 의 핵심 중 하나인 FlashAttention에 대해 알아보겠다. FlashAttention 레이어는 추론에서 내부 동작 원리도 중요하지만 인공지능 개발자가 코딩할 때 옵션을 어떻게 주는 가에 대해 이해할 필요가 있다. FlashAttention에 대해 한 번 노트를 정리해 보겠다.



## **1. FlashAttention의 중요성**

* **FlashAttention**은 대규모 언어 모델(LLM)에서 **트랜스포머(Transformer)**의 핵심 구성요소인 **Self-Attention** 연산을 **더 빠르고 효율적으로 수행하도록 최적화된 알고리즘**으로 CUDA 커널에서 구현함.
* 문제점: 기존의 self-attention은 **메모리 사용량이 많고 속도가 느림** → 특히 GPU 메모리를 많이 차지하여 **큰 모델의 추론/학습이 어려움**
* 해결책: 메모리 사용량을 대폭 감소, 속도 개선 (최대 2~4배 빠름) 및 정확도 손실 없이 기존 attention과 동일한 결과를 계산



## **2. FlashAttention 동작 원리**

* 전통적인 attention은 `QK^T`를 계산한 후 softmax → `softmax(QK^T)V`를 계산함. 이때 중간값(`QK^T`)을 GPU 메모리에 저장해야 해서 메모리 소모가 큼
* FlashAttention은,
  * 이 중간 결과를 메모리에 저장하지 않고 **스트리밍 방식**으로 처리
  * **캐시 친화적인 방식**으로 softmax를 병렬로 계산
    * **GPU의 L1/L2 캐시 또는 shared memory**와 같은 **하드웨어 레벨 메모리 계층**
    * **행렬 곱셈과 softmax 계산 시**, 중간 데이터를 **GPU의 캐시에 머무르게 하여 DRAM 접근을 줄이는 방식**으로 최적화됨
    *  **attention 계산을 fused kernel로 한번에 처리하여 메모리와 연산량을 줄인다**
      * fused kernel 은 여러 개의 연산을 **하나의 GPU 커널**로 묶어 실행하는 방식으로 GPU 프로그래밍에서 **성능 최적화의 핵심 기법 중 하나**
  * **FP16/BF16** 등 혼합 정밀도도 지원
* 동작 핵심:  attention 계산을 fused kernel로 한번에 처리하여 메모리와 연산량을 줄인다.



## **3. FlashAttention v1 vs v2 비교**

| 항목                         | FlashAttention v1               | FlashAttention v2                                   |
| ---------------------------- | ------------------------------- | --------------------------------------------------- |
| **출시 시기**                | 2022년                          | 2023년                                              |
| **연산 최적화 수준**         | 기존보다 빠르지만 일부 한계     | 완전히 새롭게 설계된 커널로 **훨씬 더 빠름**        |
| **메모리 효율성**            | 개선됨                          | **v1보다 더 적은 GPU 메모리 사용**                  |
| **지원 정밀도**              | FP16, BF16                      | FP16, BF16, **지원 안정성 향상**                    |
| **패딩 지원**                | 일부 제약 있음                  | **패딩 자동 처리 지원** (variable-length 시퀀스)    |
| **head size 제한**           | 비교적 유연                     | **제한 있음** (예: 64, 128, 256 등만 지원)          |
| **멀티쿼리 attention (MQA)** | 지원 안 함                      | **지원**                                            |
| **성능**                     | 2–3배 속도 향상 (vs 기본 torch) | **4배 이상 향상 가능**                              |
| **인풋 형태 제약**           | 존재                            | **더 유연** (예: variable batch size)               |
| **하드웨어 지원**            | A100, H100, 일부 RTX 카드       | **Hopper (H100), Ada (RTX 40 시리즈)에서 최적화됨** |
| **사용 예**                  | early vLLM, GPT-NeoX 등         | 최신 vLLM, Mistral, LLaMA 2, LLaMA 3 등             |

* FlashAttention v1과 v2는 모두 트랜스포머 모델의 **Self-Attention** 연산을 최적화한 고속 알고리즘이지만, 성능, 유연성, 지원 기능 면에서 차이가 있음



## **4. FlashAttention 설치**

* cmd 또는 powershell 에서 설치

  ```
  pip install flash-attn --no-build-isolation
  ```

* 모델 설정에서 FlashAttention을 켜는 방식

  ```python
  os.environ["VLLM_USE_FLASH_ATTENTION"] = "1"
  ```

  

## **5. FlashAttention 사용 시 주의점**

* Head size가 제한됨: FlashAttention v2는 head size가 64, 128, 256 등 **정해진 값**만 허용
* **CUDA 버전**에 따라 다르게 동작하며, **특정 GPU 아키텍처**에서만 효과적 (예: A100, H100 등)
  * FlashAttention은 **Ampere (A100)** 이후 등장한 **SM 구조 최적화**, **shared memory 용량 증가**, **속도 중심 커널 실행** 등을 활용함
  * FlashAttention v2는 **Hopper (H100)** 이상에서 **최대 성능**을 발휘하도록 설계됨 → **B200도 자연스럽게 포함**
    * Streaming Multiprecessor(SM): 
      * GPU에서 연산을 실제로 수행하는 핵심 연산 유닛 블록. 
      * NVIDIA GPU에서 SM은 CPU의 코어에 해당하는 개념.
      * 각 SM은 수십 개의 CUDA 코어, 텐서 코어, 레지스터, shared memory 등을 포함. 
      * FlashAttention과 같은 고성능 커널은 **SM 내부 자원 활용 최적화**가 핵심



## **6. head size 에러 (v2에서 자주 발생)**

```bash
ValueError: Head size 80 is not supported by FlashAttention. 
Supported head sizes are: [32, 64, 128, 256, ...]
```

*  해결법: `os.environ["VLLM_USE_V1"] = "0"` 또는 모델을 **지원되는 head size**로 조정

* 선택 기준

  | 상황                                      | 추천 버전                                 |
  | ----------------------------------------- | ----------------------------------------- |
  | 성능 극대화, 최신 GPU 사용                | **FlashAttention v2**                     |
  | 호환성 중요 (비표준 head size 등)         | **FlashAttention v1 또는 비활성화**       |
  | 복잡한 sequence, 패딩 등 자동처리 원할 때 | **v2**                                    |
  | 모델이 `v1`만 지원하는 경우               | **v1 필수** (예: 일부 오래된 vLLM 백엔드) |

* vLLM에서 쉽게 전환하려면.

  ```python
  os.environ["VLLM_USE_V1"] = "0"  # v2 사용
  os.environ["VLLM_USE_FLASH_ATTENTION"] = "1"
  ```

  

## **7. FlashAttention 지원 모델 목록**

| 모델 이름                          | 지원 여부   | 비고                                              |
| ---------------------------------- | ----------- | ------------------------------------------------- |
| **GPT-2, GPT-J, GPT-NeoX**         | ✅           | FlashAttention v1/v2 모두 호환됨                  |
| **LLaMA 1 / 2 / 3 (7B, 13B, 70B)** | ✅           | v2 기준으로 head size 맞아야 함                   |
| **Mistral 7B, Mixtral 12x7B**      | ✅           | v2 최적화 모델 (head size 128)                    |
| **Falcon 7B / 40B**                | ⚠️ 제한적    | 일부 버전에서 작동, head size 확인 필요           |
| **Phi-2 (microsoft/phi-2)**        | ✅           | head size 64로 v2 호환 가능                       |
| **Gemma 2B, 7B**                   | ✅           | `head size=64`, v2에서 잘 작동                    |
| **Command-R / xGen**               | ⚠️ 일부 호환 | HuggingFace 최신 포크에서 확인 필요               |
| **Qwen, DeepSeek, Yi**             | ⚠️ 실험적    | 일부는 커널 변경 필요                             |
| **BLOOM / OPT**                    | ❌           | 대부분 FlashAttention 미지원 또는 비효율적        |
| **T5 / BERT 계열 (encoder-only)**  | ❌           | FlashAttention은 **decoder 기반 모델**에 최적화됨 |



## **7. vLLM에서 FlashAttention 동작 확인**

* FlashAttention이 작동하려면 다음을 만족해야 함

  * **Head size**: v2는 `head_size ∈ {32, 64, 128, 256, ...}` 이어야 함 

    → 예: LLaMA-7B (`head size = 128`), Phi-2 (`64`)는 호환

  * **GPU 아키텍처**:

    * H100, A100, RTX 4090 등에서 가장 효율적
    * CUDA 11.8 이상 필요

  * **배치 구성 / 패딩 여부**: FlashAttention v2는 variable-length padding을 지원하므로 더 유연

    * **padding**: 문장의 길이가 다를 때, 짧은 문장을 길이에 맞추기 위해 덧붙이는 값
      * 문장 A → `[I, love, AI, <pad>, <pad>]`
      * 문장 B → `[Machine, learning, is, fascinating, .]`
      * 여기서 `<pad>`가 **padding token** 임.
    * **FlashAttention v1**은 **고정 길이 입력(batch 내 동일 길이)**만 효율적으로 처리하는 반면, **FlashAttention v2**는 **variable-length padding**, 즉 **서로 다른 길이의 시퀀스를 효율적으로 처리**할 수 있음
    * **FlashAttention v1**은 불필요한 연산 포함해 padding이 많아지면 낭비가 발생하는 반면에, 실제로 유효한 토큰만 계산에 반영하므로 **메모리와 계산 낭비가 줄어듬**

  * **라이브러리 호환성**: `vLLM`, `HuggingFace Transformers`, `nanoGPT`, `Lightning`, `Triton` 등에서 통합 지원

  * vLLM에서 FlashAttention 지원 모델 확인 방법

    * vLLM에서는 대부분의 **Decoder-only 모델**을 대상으로 FlashAttention을 자동 활성화할 수 있음
    * vLLM 모델 지원: [https://github.com/vllm-project/vllm/blob/main/docs/models/supported_models.md](https://github.com/vllm-project/vllm/blob/main/docs/models/supported_models.md)

  * FlashAttention이 잘 작동하지 않는 모델 유형

    | 유형                                 | 이유                                        |
    | ------------------------------------ | ------------------------------------------- |
    | Encoder-only (BERT, T5)              | 구조적으로 FlashAttention과 맞지 않음       |
    | 비표준 head size 사용 모델           | FlashAttention 커널에서 오류 발생           |
    | GPTQ/LoRA 등으로 커스터마이징된 모델 | Quantization된 attention 경로가 다름        |
    | Gated 또는 mixture-of-experts 모델   | 복잡한 attention 구조로 인해 커널 충돌 가능 |

* 테스트 권장 코드 (vLLM 기준)

  ```python
  import os
  os.environ["VLLM_USE_FLASH_ATTENTION"] = "1"
  os.environ["VLLM_USE_V1"] = "0"  # v2 사용
  
  from vllm import LLM
  llm = LLM(model="mistralai/Mistral-7B-Instruct")  # 또는 phi-2 등
  ```

  

## **8. FlashAttention의 캐시 친화성**

* FlashAttention이 말하는 **“캐시”**는 **GPU의 L1/L2 캐시 또는 shared memory**와 같은 **하드웨어 레벨 메모리 계층**을 의미
* FlashAttention은 **행렬 곱셈과 softmax 계산 시**, 중간 데이터를 **GPU의 캐시에 머무르게 하여 DRAM 접근을 줄이는 방식**으로 최적화되어 있음
*  KV 캐시 (Key/Value Cache, 모델 시퀀스 상태 유지용 아님)
* GPU 메모리 계층의 **L1/L2 cache**, **shared memory**를 효율적으로 사용하는 방식

* GPU 캐시와 KV 캐시의 차이점

  | 구분      | GPU 캐시 (FlashAttention)             | KV 캐시 (LLM Inference)                   |
  | --------- | ------------------------------------- | ----------------------------------------- |
  | 의미      | GPU 내부의 L1/L2, shared memory       | LLM의 Key/Value를 저장해 디코딩 속도 향상 |
  | 목적      | 연산 속도 향상 (softmax, matmul 등)   | 반복 추론 시 이전 토큰 재계산 방지        |
  | 크기      | 매우 작음 (KB 단위, GPU 코어에 위치)  | 매우 큼 (GB 단위, 전체 시퀀스 상태 유지)  |
  | 예시      | softmax 시 row 단위 chunk 메모리 활용 | autoregressive decoding 중 context 유지   |
  | 사용 위치 | FlashAttention 내부 커널              | vLLM, Transformers 등 디코더 추론 로직    |



## **9. 결론**

* FlashAttention v2는 대부분의 최신 LLM에서 기본 선택지이지만, head size나 GPU 아키텍처, vLLM 버전에 따라 v1을 사용해야 하는 경우도 많음.

* FlashAttention은 중간값을 GPU 캐시에 유지하고, softmax를 스트리밍 방식으로 계산하며, 연산을 하나의 fused CUDA 커널로 처리함으로써, 모두 GPU 캐시 친화적인 설계를 통해 속도를 극대화함.

* FlashAttention의 "캐시 친화적"이란, **GPU의 L1/L2 캐시나 shared memory를 효율적으로 활용하는 커널 설계를 의미하며, KV 캐시(Key-Value Cache)와는 별개의 개념**

  