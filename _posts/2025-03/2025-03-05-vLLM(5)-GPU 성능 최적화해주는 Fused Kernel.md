---
title: "vLLM(5)-GPU 성능 최적화 해 주는 Fused Kernel"
date: 2025-03-04
tags: [LLM, vLLM, OpenSource, 오픈소스, FlashAttetnion]
typora-root-url: ../
toc: true
categories: [vLLM, OpenSource]
---

vLLM 를 공부하면서 꼬리에 꼬리를 묻는 용어와 개념들이 많다. 그만큼 어느 날 톡 튀어나온 것은 아니고 기존의 프레임워크를 바탕으로 나왔다. **"Fused kernel"**은 GPU 프로그래밍에서 **성능 최적화의 핵심 기법 중 하나**로, 여러 개의 연산을 **하나의 GPU 커널**로 묶어 실행하는 방식이라고 앞 써 FlashAttention에 대해 얘기하면서 정리한 바 있다. 이 노트에서는 조금 더 Fused Kernel 에 대해 개념과 연산 그리고 왜 필요한 지에 대해 알아보도록 하겠다. 



## **1. Fused Kernel이란?**

* 여러 개의 연산(예: matmul → softmax → dropout)을 하나의 **통합된 CUDA 커널(kernel)**로 묶어서 **한 번에 실행**하는 방식

* Fused Kernel 필요성

  | 일반 커널 방식 (비 fused)                                  | Fused 커널 방식                                              |
  | ---------------------------------------------------------- | ------------------------------------------------------------ |
  | 연산마다 **개별 커널 실행** (예: A → B → C 각각 따로 실행) | 연산들을 하나로 묶어서 **한 번에 실행**                      |
  | **매 연산마다 메모리 접근** 발생 → 느림                    | 중간 결과를 **메모리에 저장하지 않고** 레지스터나 shared memory에 유지 |
  | GPU scheduling, kernel launch 비용 발생                    | **메모리 접근 줄이고** 속도 향상 (최대 수배 빠름)            |



## **2. 예시: Attention 연산**

* Transformer 모델에서 Self-Attention은 다음과 같이 구성됨

  ```
  QK^T → Softmax → Dropout → Weighted Sum (V)
  ```

* 일반(전통) 방식과 Fused 방식 비교

  | 일반 커널 방식 (비 fused)                                    | Fused 커널 방식                                              |
  | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | 각각의 연산을 따로 커널로 실행<br/> → QK^T 결과를 메모리에 저장<br/> → 다시 Softmax, Dropout을 각자 실행<br/> → **GPU 메모리 접근 3~4회** | **위 모든 연산을 하나의 커널로 실행**<br/> → 중간결과 저장 생략<br/> → **메모리는 한 번만 접근, 연산은 연속으로 수행** |



## **3. 성능 효과**

| 항목              | 전통 커널 | Fused 커널     |
| ----------------- | --------- | -------------- |
| 커널 실행 횟수    | 3~5회     | 1회            |
| 메모리 접근 횟수  | 많음      | 최소화         |
| 속도              | 느림      | **2~4배 빠름** |
| GPU memory 사용량 | 높음      | 낮음           |



## **4. 어디서 사용되나?**

* **FlashAttention** (fused softmax + matmul)

* **LayerNorm Fusion** - LayerNorm 연산 전체를 하나의 GPU 커널로 통합(fusion)하여 연산 속도를 높이고 메모리 접근을 줄이는 최적화 기법

* **Transformer block fusion** (multi-head + MLP)  - 

  * MLP(Multilayer Perceptron): 여러 개의 완전연결층(Dense layer)으로 구성된 가장 기본적인 인공 신경망. 입력과 출력을 비선형 변환으로 연결하는 다층 신경망

* NVIDIA TensorRT, PyTorch 2.x, JAX XLA, DeepSpeed 등에서 핵심 최적화 기술로 사용

  

## **5. 요약 정리**

| 항목      | 설명                                    |
| --------- | --------------------------------------- |
| 정의      | 여러 GPU 연산을 하나의 커널로 묶어 실행 |
| 장점      | 속도 ↑, 메모리 접근 ↓, 커널 런칭 비용 ↓ |
| 사용 예   | Attention, LayerNorm, Conv+ReLU 등      |
| 관련 기술 | CUDA, Triton, XLA, FlashAttention 등    |