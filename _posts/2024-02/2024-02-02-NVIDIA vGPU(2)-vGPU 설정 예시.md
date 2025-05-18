---
title: "NVIDIA vGPU(2)-vGPU 설정 예시"
date: 2024-02-02
tags: [NVIDIA, GPU, RDMA, vGPU, HyperVisor]
typora-root-url: ../
toc: true
categories: [NVIDIA, Accelerated Computing, HPC, GPU, vGPU]
---

NVIDIA의 vGPU에 대해 어떻게 구성하는 지 다양한 시나리오를 작성해 보겠다. NVIDIA H100 GPU 4장을 사용하는 환경에서 vGPU를 할당하는 방식은 워크로드의 특성과 요구사항에 따라 다양하게 구성할 수 있다. H100은 고성능 컴퓨팅 및 AI 워크로드에 최적화된 GPU로, vGPU 기능을 통해 하나의 물리 GPU를 여러 가상 머신(VM)에서 공유할 수 있다.



## 1. 전체 시나리오 구성 고려사항

* **라이선스:** vGPU 기능을 사용하려면 NVIDIA의 vGPU 소프트웨어 라이선스가 필요함
* **하이퍼바이저 지원:** VMware vSphere, Citrix Hypervisor 등에서 vGPU 기능을 지원함
* **MIG(Multi-Instance GPU)**: H100은 MIG 기능을 통해 GPU를 논리적으로 분할하여 사용할 수 있음. 이는 vGPU와는 다른 기술로, 특정 워크로드에 더 적합할 수 있음.



## 2. 대규모 AI 학습 워크로드를 위한 단일 VM에 다수의 vGPU 할당

* 환경: H100 GPU 4장
* 구성
  * 4개의 H100 GPU를 단일 VM에 할당 (GPU 패스스루 또는 vGPU 할당)
  * 각 GPU는 전체 메모리(80GB)를 단일 VM에서 사용
* 장점:
  * 대규모 데이터셋 처리 및 복잡한 모델 학습에 적합
  * 최대의 GPU 성능 활용



## 3. AI 추론 워크로드를 위한 다수의 vGPU 할당

* 환경: H100 GPU 4장 (각각 80GB HBM3 메모리)
* 목표: AI 추론 서비스를 위한 다수의 경량 VM 구성
* 구성:
  - 4개의 H100 GPU를 단일 VM에 할당 (GPU 패스스루 또는 vGPU 할당)
  - 각 GPU는 전체 메모리(80GB)를 단일 VM에서 사용
* 장점:
  - 대규모 데이터셋 처리 및 복잡한 모델 학습에 적합
  - 최대의 GPU 성능 활용



## 4. 혼합 워크로드를 위한 다양한 vGPU 프로파일 할당

* 환경: H100 GPU 4장

* 목표: AI 추론 및 학습 워크로드를 동시에 처리

* 구성:

  - 각 H100 GPU에서 다양한 vGPU 프로파일 생성
    - 예: 2개의 vGPU는 40GB 메모리 할당 (학습용)
    - 4개의 vGPU는 10GB 메모리 할당 (추론용)

  - 총 24개의 vGPU 인스턴스를 통해 다양한 VM에 할당

* 장점:

  - 워크로드에 따라 유연한 리소스 할당
  - GPU 자원의 최적화된 활용



