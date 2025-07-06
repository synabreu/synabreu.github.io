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