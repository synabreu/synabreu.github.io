---
title: "DeepSpeed 와 ZeRO는 다른 기술인가"
date: 2024-01-06
tags: [DeepSpeed, 딥스피드, Microsoft, 마이크로소프트, PyTorch, ZeRO optimizer, Mixed Precision, Model Parallelism, Pipeline Parallelism, DeepSpeed-Inference, DDP, Distributed Data Parallel]
typora-root-url: ../
toc: true
---



DeepSpeed와 ZeRO는 별개의 기술이 아니며, ZeRO는 DeepSpeed 프레임워크 안에 포함된 핵심 기술 중 하나이다. 좀 더 알아보면 다음과 같다.



## 1. DeepSpeed vs. Zero

| 기술명                               | 설명                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| **DeepSpeed**                        | Microsoft가 만든 **PyTorch 기반 분산 학습 프레임워크** 전체. 여러 최적화 기술을 포함 |
| **ZeRO (Zero Redundancy Optimizer)** | DeepSpeed 내부의 **대표적인 메모리 최적화 기능** 중 하나. 대규모 모델 학습을 위한 핵심 기술 |



## 2. DeepSpeed 구성 요소

| 모듈                       | 설명                                                     |
| -------------------------- | -------------------------------------------------------- |
| ✅ ZeRO                     | 메모리 최적화를 위한 파라미터/그래디언트/옵티마이저 분산 |
| ✅ FP16 / bf16 지원         | 자동 혼합 정밀도 학습 (AMP)                              |
| ✅ DeepSpeed Inference      | 추론 속도 최적화 (CUDA 커널 최적화)                      |
| ✅ DeepSpeed MoE            | Mixture of Experts 모델 병렬 학습 지원                   |
| ✅ Pipeline Parallelism     | 모델을 여러 GPU에 레이어 단위로 나눔                     |
| ✅ Activation Checkpointing | GPU 메모리를 줄이기 위해 일부 activation만 저장          |



## 3. 결론

| 질문                         | 답변                                            |
| ---------------------------- | ----------------------------------------------- |
| DeepSpeed와 ZeRO는 같은가요? | ❌ 기술적으로 다름                               |
| 어떤 관계인가요?             | ✅ ZeRO는 DeepSpeed의 핵심 기능 중 하나.         |
| ZeRO만 단독 사용 가능한가요? | ❌ 아니다. ZeRO는 DeepSpeed 엔진 안에서만 동작함 |
