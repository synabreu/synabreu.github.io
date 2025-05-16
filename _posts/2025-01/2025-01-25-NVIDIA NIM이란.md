---
title: "NVIDIA NIM(2): NVIDIA NIM 이란?"
date: 2025-01-25
tags: [NVIDIA, NIM,GPU, CUDA, microservice, inference, AWS, Azure, GCP, Kubernetes, k8s, NVIDIA Cloud, NVIDIA AI Enterprise, 쿠버네티스, 마이크로서비스, 추론]
typora-root-url: ../
toc: true
categories: [NVIDIA, NIM]
---

NVIDIA NIM 이란 풀어서 적으면 **'NVIDIA Inference Microservice'** 이다. 한마디로 말해서, 온-프레미스, 데이터 센터나 퍼블릭 클라우드에서 파운데이션 모델의 배포를 가속화하고 데이터를 안전하게 유지할 수 있도록 지원하는 사용하기 쉬운 마이크로서비스 집합체로 구성되어 있다.

또한, NIM 마이크로서비스는 프로덕션 수준의 런타임 환경을 제공하며, 지속적인 보안 업데이트를 포함한다. 산업계의 표준으로 기업들이 안정적으로 사용할 수 있도록 지속적인 API 유지 서비스하면서 비즈니스 애플리케이션과 결합해 실행할 수 있도록 품질을 제공하고 있는 것이 큰 특징이다.

원래 NVIDIA NIM 는 NVIDIA AI Enterprise 플랫폼의 일부분으로 시작되었다. 최신 버전인 NVIDIA AI Enterprise 5.0에는 생성형 AI 개발 및 배포를 가속화하기 위한 사용하기 쉬운 마이크로서비스가 포함되어 있다. 이 서비스는 NVIDIA AI 파운데이션 모델과 사용자 정의 모델을 포함한 다양한 AI 모델을 지원하며, 업계 표준 API를 활용하여 데이터 센터, 클라우드, 워크스테이션 전반에 걸쳐 LLM 추론을 확장할 수 있도록 돕는다.



![그림 1 - NVIDIA NIM 아키텍처](../images/2025-01/NIM01-1.png)

<div align="center">[그림 1 - NVIDIA NIM 아키텍처]</div>

​									

[그림 1]에서 보듯이, 크게 3 가지 계층으로 나눠져 있다. 맨 아래 계층 부터 설명하자면, NVIDIA CUDA 계층으로 NVIDIA가 직접 운영하는 자체 서비스로써 수억 개의 GPU CUDA 기반이다. 그 다음이 두번째 계층인 쿠버네티스(Kubernetes)이고, 최종적으로 애플리케이션, API 및 모델 등등 집합체로 이뤄져 있다. 한편, NIM은 사전 빌드된 컨테이너와 Helm 차트를 기반으로 구축되어 있고, 이를 통해 Microsoft Azure, AWS, Google Cloud를 포함한 다양한 환경에서 인프라가 견고하고, 테스트되었으며, 검증되도록 보장한다.



* NVIDIA CUDA 계층에 대해 좀더 설명하자면, **NVIDIA CUDA 는 'Compute Unified Device Architecture'의 줄임말**로 NVIDIA 그래픽 처리 장치(GPU, Graphical Processing Unit)에서 병렬 컴퓨팅을 수행할 수 있도록 설계된 병렬 컴퓨팅 플랫폼 및 프로그래밍 모델로 뜻한다. 따라서, CUDA는 개발자가 GPU의 고성능 연산 능력을 활용하여 애플리케이션의 속도를 크게 향상시킬 수 있도록 지원한다.

* 쿠버네티스(Kubernetes, K8s)는 컨테이너화된 애플리케이션의 배포, 스케일링, 관리를 자동화하는 오픈소스 플랫폼이다. 구글에서 처음 개발했으며, 현재는 클라우드 네이티브 컴퓨팅 재단(CNCF, Cloud Native Computing Foundation)에서 관리하고 있다. 기술적으로 말하자면, 쿠버네티스는 대규모 컨테이너 오케스트레이션을 효율적으로 수행하도록 설계되었다.

* 애플리케이션, API 및 모델 등등. 이 패키지는 대표적으로 사전 구축된 컨테이너와 헬름 챠트(Helm chart), 각종 산업에 표준화된 API, 산업 관련 해당 도메인에 특화된 소스 코드, 최적화된 추론 엔진 등등과 마치 종합 선물처럼 하나의 패키지 형태 집합으로 이뤄져 한데 모아 두었다. 따라서, 그러한 NIM 패키지는 여러분의 원하는 형태의 애플리케이션을 단 몇 줄의 코드만으로 기업형 AI 애플리케이션을 빠르게 빌드하고 배포할 수 있다.



그러므로, NVIDIA NIM은 시스템 간 원활한 통합을 가능하게 하기 위해 업계 표준 API와 NVIDIA Cloud 표준을 채택하고 있다. 특화된 산업 분야에서 일하는 사용자들을 위해, NIM은 대규모 언어 모델(LLM), 비전 언어 모델(VLM), 비디오 처리, 헬스케어 등 다양한 분야에 맞춘 도메인별 소스 코드를 제공한다. 이를 통해 고객은 특정 사례에 필요한 도구를 활용할 수 있다.

또한, NVIDIA는 고객의 맞춤화의 중요성을 잘 알고 있다. 그래서 사용자들이 특정 사례에 맞춰 개발한 사용자 정의 모델도 지원한다. 이를 통해 고객은 고유한 요구 사항에 맞게 NVIDIA NIM 기술을 조정할 수 있다. 끝으로, **추론 엔진(Inference Engine)**은 각 모델과 하드웨어 SKU에 맞게 최적화되어 있어, 애플리케이션에 상관없이 뛰어난 성능을 제공한다.