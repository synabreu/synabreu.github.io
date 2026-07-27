---
title: "쿠버네티스 기반 LLM용 vLLM 분산 추론 llm-d"
date: 2026-07-27
tags: [llm-d, vllm. OpenSource, 분산추론, 쿠버네티스, kubernetes,  ]
typora-root-url: ../
toc: true
categories: [OpenSource]
---
**llm-d**는 대규모 모델 분산 추론(Large Language Model Distributed Inferenc)를 의미하며, 대규모 언어 모델(LLM)을 쿠버네티스(Kubernetes) 환경에서 대규모 서비스 형태로 운영하기 위한 오픈소스 분산 추론(Inference) 플랫폼이다. 쉽게 말하면 "vLLM을 기반으로 한 LLM용 쿠버네티스 추론 플랫폼()Kubernetes Inference Platform)이다.

기존 vLLM이 "하나의 GPU 서버에서 LLM 추론을 빠르게 수행하는 엔진"이라면, llm-d는 여러 GPU 노드와 여러 모델 인스턴스를 효율적으로 관리하는 운영 계층(Serving Infrastructure)에 가깝다.

#### 1. 왜 llm-d가 필요한가?

ChatGPT 같은 서비스를 운영한다고 생각하면 문제가 커진다. 예를 들어, Llama 3.1 405B,DeepSeek V3/R1 671B, GPT-OSS 120B, Solar Open 250B MoE 등 같은 모델은 하나의 GPU에 올라가지 않는다.

![llmd-01]({{ '/images/2026-07/llmd-01.jpg' | relative_url }})

<img src="{{ '../images/2026-07/llmd-01.jpg' | relative_url }}" alt="llm-d 이미지" style="max-width:100%; height:auto; display:block; margin:0 auto;" />


![LLM-D Architecture]({{ "https://synabreu.github.io/images/2026-07/llmd-01.jpg" | relative_url }})


여기서 발생하는 문제가:

* 어떤 GPU 서버로 요청을 보낼 것인가?
* 현재 KV Cache가 많이 남아있는 서버는 어디인가?
* Prefill과 Decode를 분리할 것인가?
* GPU 사용률을 어떻게 최적화할 것인가?
* 사용자별 latency SLA를 어떻게 보장할 것인가?

이다. 따라서, llm-d는 이런 **LLM 특화 스케줄링 문제**를 해결하기 위해 등장했다.

#### 2. vLLM과 llm-d 차이

| 구분      | vLLM                                | llm-d                          |
| --------- | ----------------------------------- | ------------------------------ |
| 역할      | LLM 추론 엔진                       | LLM 추론 플랫폼                |
| 주요 기능 | Token 생성 최적화                   | Cluster 운영 최적화            |
| 위치      | Worker Layer                        | Platform Layer                 |
| 핵심 기술 | PagedAttention, Continuous Batching | Kubernetes Scheduling, Routing |
| 관리 대상 | 하나의 모델 서버                    | 여러 모델 서버/GPU Pool        |
| 사용자    | AI 개발자                           | AI Platform Engineer           |

쉽게 이해하기 위해 비유하면, CUDA = GPU 연산 계층, PyTorch = 모델 개발 프레임워크, vLLM = 고성능 LLM 런타임, llm-d = 쿠버네티스 기반 LLM 클라우드 플랫폼이다.

#### 3. llm-d 주요 기술 구성

##### 1) vLLM 기반 런타임

llm-d는 vLLM을 핵심 실행 엔진으로 사용한다.

![1785131296889](image/2026-07-27-쿠버네티스기반LLM용vLLM분산추론llm-d/1785131296889.jpg)

vLLM의 장점인 PagedAttention, 연속 배칭(Continuous Batching), KV 캐시 관리, 텐서 병렬(Tensor Parallel), 양자화(Quantization) 등을 그대로 활용한다.

##### 2) 지능적 라우팅(Intelligent Routing)

일반 쿠버네티스 로드 밸런서(Kubernetes Load Balancer)는 단순 라인딩 로빈 방식으로 동작한다.

![1785131397022](image/2026-07-27-쿠버네티스기반LLM용vLLM분산추론llm-d/1785131397022.jpg)

문제는 LLM에서는 아래와 같이 요청이 발생하면,

GPU1:
KV Cache 90% 사용
긴 Context 요청 처리 중

GPU2:
KV Cache 여유
Idle

일반 LB:
GPU1 선택

이렇게 되면, 결과는 TTFT 증가하고 GPU 메모리 부족 및 처리량(Throughput) 감소.
그러나 llm-d는,

![1785131479630](image/2026-07-27-쿠버네티스기반LLM용vLLM분산추론llm-d/1785131479630.jpg)

#### 4. Prefill / Decode 분리(Disaggregation)

1) LLM 추론에서는 하나의 요청을 처리할 때 두 가지 단계가 있음

![1785131631492](image/2026-07-27-쿠버네티스기반LLM용vLLM분산추론llm-d/1785131631492.jpg)

2) 전통적인 방식은 하나의 GPU 서버가 두 작업을 모두 수행함

기존 방식 (Coupled / 통합 구조)

![1785131789027](image/2026-07-27-쿠버네티스기반LLM용vLLM분산추론llm-d/1785131789027.jpg)

  즉, Prefill 과 Decode 단계가 같은 GPU 자원에 결합(coupled) 되어 있다.

3) Disaggregation을 적용하면, Prefill 전용 GPU와 Decode 전용 GPU를 분리한다.

![1785131889749](image/2026-07-27-쿠버네티스기반LLM용vLLM분산추론llm-d/1785131889749.jpg)

## 5. GPU 추론 분리 이유

| 구분      | Prefill            | Decode             |
| --------- | ------------------ | ------------------ |
| 주요 작업 | Prompt 처리        | Token 생성         |
| 병목      | Compute            | Memory bandwidth   |
| GPU 요구  | Tensor Core 연산량 | KV Cache 처리      |
| 특징      | 짧고 강한 연산     | 길고 반복적인 연산 |

두 단계의 GPU 요구사항이 다르기 때문. 예를 들면, Prefill은 긴 문서 RAG 요청:

입력:
100,000 tokens 문서

→ 한번에 큰 Matrix 계산

GPU 연산(Compute) 사용률 증가

Decode 시, 답변 생성:

token 1
token 2
token 3
...
token 500

반복적으로 KV 캐시를 읽음. 메모리 대역폭이 중요.

## 6. 쿠버네티스와 관계

llm-d는 쿠버네티스 네이티브(Kubernetes Native)이다. 지원 환경은 NVIDIA GPU, AMD GPU, TPU, XPU, CPU 등 다양한 accelerator 환경을 목표로 한다.

![1785131997994](image/2026-07-27-쿠버네티스기반LLM용vLLM분산추론llm-d/1785131997994.jpg)

## 7. KServe와 관계

기존:

![1785132087187](image/2026-07-27-쿠버네티스기반LLM용vLLM분산추론llm-d/1785132087187.jpg)

llm-d 적용:

![1785132167053](image/2026-07-27-쿠버네티스기반LLM용vLLM분산추론llm-d/1785132167053.jpg)

정리하자면, KServe는 모델 서빙 프레임워크이며, llm-d는 LLM 특화 분산 추론 계층이며,vLLM은 모델 실행 엔진이다.

## 8. NVIDIA Dynamo와 비교

|            | llm-d                       | NVIDIA Dynamo    |
| ---------- | --------------------------- | ---------------- |
| 회사       | Red Hat / Google / IBM 중심 | NVIDIA           |
| 기반       | Kubernetes + vLLM           | NVIDIA AI Stack  |
| GPU 종속성 | 낮음                        | NVIDIA 최적화    |
| Scheduler  | AI-aware routing            | NVIDIA optimized |
| 목표       | Open ecosystem              | NVIDIA ecosystem |

## 9. 기업 AI 인프라 관점

  ![1785132245907](image/2026-07-27-쿠버네티스기반LLM용vLLM분산추론llm-d/1785132245907.jpg)

이런 구성을 하면, GPU 활용도 증가 및 비용 감소, 멀티 모달 서빙 가능, SLA 관리 가능 및 클라우드 네이티브 운영 가능함.

## 10. 결론

vLLM이 "LLM을 빠르게 실행하는 엔진"이라면, llm-d는 "수십~수백 GPU에서 vLLM 기반 LLM 서비스를 운영하기 위한 쿠버네티스-네이티브 분산 추론 플랫폼"이다. 현재 AI 데이터센터 관점에서는, NVIDIA NIM/Dynamo ↔ Red Hat llm-d ↔ KServe/vLLM 생태계가 차세대 LLM 서빙 아키텍처 경쟁 구도이다.
