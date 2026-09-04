---
title: "[워크삽] AWS 트레니엄 워크삽(2)-AWS AI Chip"
date: 2026-09-04
tags: [aws, 트레니엄, Trainium, vLLM, NeuronX Distributed, NxD, Amazon, EKS]
typora-root-url: ../
toc: true
categories: [aws]
---
이번 AWS 트레니엄 워크삽 글에서는 AWS AI Chip에 대해 알아보자!

# 1. NeuronCore: AWS AI 칩의 기반

NeuronCore-v2는 Trainium과 Inferentia2를 포함한 AWS 2세대 AI 칩을 구동하는 기본 연산 장치다. 이 아키텍처는 AI 가속 성능을 크게 향상하도록 설계됐으며, 대규모 언어 모델을 비롯한 학습과 추론 워크로드를 모두 지원한다.

# 2. NeuronCore-v2 아키텍처 개요

NeuronCore-v2는 다단계 메모리 계층 구조와 머신러닝 연산에 최적화된 전용 연산 장치를 중심으로 설계됐다.

- **텐서 처리 장치(Tensor Processing Units):** 행렬 연산, 합성곱, 신경망 계산을 처리하는 전용 하드웨어다.
- **벡터 처리 장치(Vector Processing Units):** 원소별 연산과 벡터화된 계산을 처리한다.
- **스칼라 처리 장치(Scalar Processing Units):** 제어 흐름, 분기, 스칼라 연산을 담당한다.
- **메모리 계층 구조(Memory Hierarchy):** 고대역폭 메모리 접근을 지원하는 다단계 캐시 시스템이다.
- **NeuronLink 인터페이스:** 코어와 칩 사이에서 데이터를 효율적으로 주고받을 수 있게 한다.

![NeuronCore-v2 아키텍처]({{ '/images/2026-09/neuron-core-v2.png' | relative_url }})
*그림 1. NeuronCore-v2 아키텍처*

# 3. NeuronCore-v2의 주요 설계 특징

NeuronCore-v2는 다음 특성을 중심으로 설계됐다.

- **높은 처리량(High Throughput):** 배치 처리와 병렬 실행에 최적화돼 있다.
- **메모리 효율성(Memory Efficiency):** 압축과 압축 해제를 포함한 고급 메모리 관리 기능을 제공한다.
- **유연성(Flexibility):** 동적 형상(Dynamic Shapes)과 INT8, FP16, BF16, cFP8, TF32, FP32 등 다양한 데이터 형식을 지원한다.
- **확장성(Scalability):** 다중 칩 구성과 분산 추론을 고려해 설계됐다.

# 4. NeuronCore-v2의 성능 특성

NeuronCore-v2 두 개가 제공하는 합산 성능은 다음과 같다.

| 데이터 형식            |   합산 성능 | 주요 용도                           |
| ---------------------- | ----------: | ----------------------------------- |
| INT8                   |    380 TOPS | 양자화 추론 워크로드                |
| FP16, BF16, cFP8, TF32 |  190 TFLOPS | 학습과 추론을 위한 균형 잡힌 정밀도 |
| FP32                   | 47.5 TFLOPS | 높은 정밀도가 필요한 계산           |

> 위 수치는 Trainium 및 Inferentia2 칩에 탑재된 NeuronCore-v2 두 개의 합산 성능을 나타낸다.

이 아키텍처는 AWS AI 칩 전략의 기반을 이룬다. 단일 코어 연산부터 여러 칩을 사용하는 분산 추론 시스템까지 효율적으로 확장할 수 있게 한다.

# 5. NeuronCore-v3: 차세대 AI 연산 아키텍처

NeuronCore-v3는 AWS AI 연산 아키텍처의 최신 발전 단계다. Trainium2 칩의 기반으로 사용되며, 이전 세대보다 성능과 기능이 크게 향상됐다.

## 5-1. NeuronCore-v3 아키텍처 개요

NeuronCore-v3는 향상된 연산 장치와 메모리 관리 기능을 갖춘 더욱 정교한 구조를 도입했다.

- **고급 텐서 처리 장치(Advanced Tensor Processing Units):** 행렬 연산과 신경망 계산의 처리량을 높였다.
- **향상된 벡터 처리 장치(Enhanced Vector Processing Units):** 원소별 연산과 벡터화된 계산을 더욱 효율적으로 처리한다.
- **스칼라 처리 장치(Scalar Processing Units):** 제어 흐름과 분기 처리 기능을 강화했다.
- **고급 메모리 계층 구조(Advanced Memory Hierarchy):** 대역폭과 압축 기능이 향상된 다단계 캐시 시스템이다.
- **NeuronLink-v3 인터페이스:** 칩 사이의 통신 성능을 높인 차세대 인터커넥트다.

![NeuronCore-v3 아키텍처]({{ '/images/2026-09/nc-v3.png' | relative_url }})
*그림 2. NeuronCore-v3 아키텍처*

## 5-2. NeuronCore-v3의 주요 개선 사항

v3 아키텍처는 v2와 비교해 다음과 같이 개선됐다.

- **더 높은 코어 밀도:** Trainium의 NeuronCore-v2 두 개와 비교해 Trainium2에는 NeuronCore-v3 여덟 개가 탑재된다.
- **향상된 메모리 대역폭:** v2의 820GiB/s에서 2.9TB/s로 높아졌다.
- **고급 압축 기능:** 인라인 메모리 압축 및 압축 해제 알고리즘이 개선됐다.
- **향상된 프로그래밍 유연성:** 동적 형상과 사용자 정의 연산자(Custom Operators)를 더욱 폭넓게 지원한다.
- **높은 확장성:** 더 큰 규모의 분산 학습 구성을 지원한다.

## 5-3. NeuronCore-v3의 성능 특성

NeuronCore-v3 코어 한 개가 제공하는 대략적인 성능은 다음과 같다.

| 데이터 형식      |    코어당 성능 | 주요 용도                     |
| ---------------- | -------------: | ----------------------------- |
| FP8              |  약 162 TFLOPS | 처리량이 중요한 추론 워크로드 |
| BF16, FP16, TF32 |   약 83 TFLOPS | 학습 및 추론                  |
| FP32             | 약 22.6 TFLOPS | 높은 정밀도가 필요한 계산     |

Trainium2 칩 하나에는 NeuronCore-v3 코어가 여덟 개 탑재된다. 이 코어들의 합산 성능을 바탕으로 Trainium2는 모든 정밀도 형식에서 1세대 Trainium보다 높은 성능을 제공한다. 특히 대규모 학습 워크로드에 유리하도록 최적화돼 있다.

# 6. AWS Trainium: 대규모 모델을 위해 설계된 전용 가속기

## 6-1. Trainium 1세대

AWS Trainium은 AWS의 1세대 AI 학습 칩으로, 머신러닝 학습 워크로드를 위해 특별히 설계됐다. Trainium 칩을 탑재한 Amazon EC2 Trn1 인스턴스는 대규모 언어 모델의 학습과 추론에 필요한 성능과 비용 효율성을 제공한다.

Trn1 인스턴스 한 대에는 최대 16개의 Trainium 칩이 탑재된다. 각 칩의 주요 사양은 다음과 같다.

- **NeuronCore-v2 두 개:** 두 코어를 합해 INT8 380TOPS, FP16·BF16·cFP8·TF32 190TFLOPS, FP32 47.5TFLOPS를 제공한다.
- **32GiB 고대역폭 메모리(HBM):** 모델 상태를 저장하며 820GiB/s의 메모리 대역폭을 제공한다.
- **1TB/s DMA 대역폭:** 인라인 메모리 압축 및 압축 해제 기능을 사용해 데이터를 효율적으로 이동한다.
- **NeuronLink-v2:** 분산 학습과 분산 추론을 위한 칩 간 인터커넥트다.
- **고급 프로그래밍 기능:** 동적 형상, 제어 흐름, 사용자 정의 연산자를 지원한다.

![Trainium 아키텍처]({{ '/images/2026-09/trainium-neurondevice.png' | relative_url }})
*그림 3. Trainium 아키텍처*

## 6-2. Trainium2: 차세대 성능

AWS Trainium2는 AWS 2세대 AI 학습 칩으로 크게 발전한 제품이다. Amazon EC2 Trn2 인스턴스에 탑재되며, 대규모 머신러닝 학습을 위한 성능을 대폭 높였다.

### 1) Trainium2의 주요 기능

- **NeuronCore-v3 여덟 개:** 합산 기준으로 약 1,300 FP8 TFLOPS, 650~700 BF16·FP16·TF32 TFLOPS, 180 이상의 FP32 TFLOPS를 제공한다.
- **96GiB 고대역폭 메모리(HBM):** 기존보다 세 배 큰 메모리 용량과 2.9TB/s의 메모리 대역폭을 제공한다.
- **3.5TB/s DMA 대역폭:** 인라인 압축 및 압축 해제를 지원해 데이터 이동 성능을 높인다.
- **NeuronLink-v3:** 분산 학습을 위한 향상된 칩 간 인터커넥트다.
- **향상된 집합 통신(Collective Communication):** 대규모 분산 학습 워크로드에 맞게 최적화됐다.
- **논리적 NeuronCore 구성(Logical NeuronCore Configuration, LNC):** 코어를 유연하게 그룹화해 자원 활용률을 최적화한다.

![Trainium2 아키텍처]({{ '/images/2026-09/trainium2.png' | relative_url }})
*그림 4. Trainium2 아키텍처*

### 2) Trainium 대비 성능 개선

- **연산 성능:** 전체 학습 성능이 최대 네 배 향상됐다.
- **메모리:** HBM 용량이 32GiB에서 96GiB로 세 배 증가했으며, 메모리 대역폭은 3.5배 이상 높아졌다.
- **인터커넥트:** NeuronLink-v3를 통해 칩 간 통신 성능이 크게 향상됐다.
- **확장 규모:** 더 큰 규모의 분산 학습 구성을 지원한다.

## 6-3. NeuronCore-v2에서 v3로의 발전

Trainium의 NeuronCore-v2에서 Trainium2의 NeuronCore-v3로 이어지는 변화는 AWS AI 칩 아키텍처가 어떻게 발전했는지를 보여준다.

| 구분          | NeuronCore-v2 및 Trainium           | NeuronCore-v3 및 Trainium2                   |
| ------------- | ----------------------------------- | -------------------------------------------- |
| 코어 구성     | 칩당 2코어                          | 칩당 8코어                                   |
| HBM 용량      | 32GiB                               | 96GiB                                        |
| 메모리 대역폭 | 820GiB/s                            | 2.9TB/s                                      |
| 칩 간 연결    | NeuronLink-v2                       | NeuronLink-v3                                |
| 설계 방향     | 균형 잡힌 성능을 제공하는 기반 확립 | 연산 밀도, 메모리, 인터커넥트 및 확장성 강화 |

NeuronCore-v2는 듀얼 코어 설계와 균형 잡힌 성능을 바탕으로 AWS AI 칩의 기반을 마련했다. NeuronCore-v3는 이를 옥타 코어 구성으로 확장하면서 메모리와 인터커넥트 기능을 강화했다.

이러한 아키텍처 발전을 통해 AWS는 NeuronCore-v2에서 확립한 기본 설계 원칙을 유지하면서 더욱 강력한 AI 학습 및 추론 기능을 제공할 수 있게 됐다.

# 7. 참고자료

* **원문:** [AWS Workshop Studio – AWS AI Chips](https://catalog.us-east-1.prod.workshops.aws/workshops/177cf2c8-d451-405b-a463-eb77d38b8617/en-US/vllm/fundamentals/aws-ai-chips)
