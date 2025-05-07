---
title: "NVIDIA DGX SuperPOD"
date: 2023-12-20
tags: [NVIDIA, GPU, RDMA, DGX, SuperPod, InfiniBand, Spine-Leaf, NDR, NCCL]
typora-root-url: ../
---

NVIDIA DGX SuperPOD는 DGX POD의 확장형 버전으로, 대규모 LLM 훈련, 시뮬레이션, HPC, GenAI 등에 사용되는 엔터프라이즈급 AI 슈퍼컴퓨터 아키텍처이다. 따라서,  수십~수백 대의 **DGX 시스템**을 **초고속 스토리지**, **InfiniBand 기반 스파인-리프 네트워크**, **NVIDIA AI 소프트웨어 스택**과 함께 구성한 **대규모 AI 슈퍼컴퓨팅 플랫폼**으로 구성한다.

단순히 서버 수를 늘린 것이 아니라, **서버 + 네트워크 + 스토리지 + 운영 소프트웨어**가 하나의 통합 플랫폼으로 설계되어 있고, 세계 최고 수준의 **AI 훈련 속도, 확장성, 신뢰성** 확보한다.



## 1. 실제 구성 예시

* H100 GPU와 함께 DGX SuperPOD 구성

| 구성 항목       | 세부 내용                                                  |
| --------------- | ---------------------------------------------------------- |
| DGX 시스템 수   | **256대** (DGX H100)                                       |
| GPU 수          | 256 × 8 = **2048개의 H100 GPU**                            |
| 네트워크        | **InfiniBand NDR 400Gbps**, Spine-Leaf full CLOS           |
| 스토리지        | **DDN A³I**, **WekaIO**, **NetApp AI 스토리지** (GDS 지원) |
| 성능            | **1엑사플롭스 AI 성능 이상** (FP8/FP16 기준)               |
| 운영 소프트웨어 | **NVIDIA Base Command**, Slurm, Kubernetes, NGC 컨테이너   |
| AI 사용 사례    | GPT-4급 LLM, GenAI 파이프라인, 재무/의료 모델 학습         |



## 2. 네트워크 구성 (Spine-Leaf)

* **수십~수백 개의 Leaf 스위치** (DGX와 연결)
* **10~40개의 Spine 스위치** (Leaf 스위치를 cross 연결)
* 전체는 **non-blocking**, **CLOS 토폴로지** 기반
* **InfiniBand NDR** 또는 **HDR** 사용 (400~200Gbps)
* 모든 DGX는 최소 2~4개의 NIC (ConnectX-7, BlueField)로 고속 연결
* RDMA + NCCL + SHARP(NVIDIA Collective Accelerator) 활용



## 3. 스토리지 구성

* **GPU Direct Storage (GDS)**: GPU ↔ NVMe 스토리지 직접 연결로 CPU 개입 없이 속도 향상
* **스토리지 노드도 InfiniBand로 직접 연결**
* 사용 솔루션 예: **DDN A³I**, **WekaFS**, **VAST Data**, **NetApp EF600**



## 4. 운영 소프트웨어: NVIDIA Base Command Platform

| 구성 요소                 | 역할                                        |
| ------------------------- | ------------------------------------------- |
| **Slurm**                 | 분산 훈련 및 스케줄링                       |
| **Kubernetes**            | AI inference 및 파이프라인 실행             |
| **NGC**                   | PyTorch/TensorFlow/JAX 컨테이너 이미지 제공 |
| **Base Command Manager**  | 노드 관리, 모니터링, job orchestration      |
| **NVIDIA Fabric Manager** | NVLink/NVSwitch 모니터링                    |

*  NVIDIA DGX POD 와 SuperPod에는 Base Command Platform이 기본 탑재되지 않고, 옵션으로 추가 구매 및 배포가 가능함.



## 5. 실제 사례

| 조직/기업                          | 사용 목적                                   |
| ---------------------------------- | ------------------------------------------- |
| **Meta**                           | LLaMA 3 훈련용 SuperCluster                 |
| **Microsoft**                      | OpenAI 모델 훈련용 Azure 기반 SuperPOD      |
| **NVIDIA 자체**                    | DGX GH200 훈련용 (Grace Hopper 기반)        |
| **국가 연구소 (KISTI, Jülich 등)** | 기후 시뮬레이션, 신약 개발, LLM 훈련        |
| **Oracle OCI**                     | 고객용 AI Foundation Model 훈련 인프라 제공 |



## 6. 비교: DGX POD vs SuperPOD

| 항목      | DGX POD               | DGX SuperPOD                         |
| --------- | --------------------- | ------------------------------------ |
| 규모      | 4~32 DGX              | 64~512+ DGX                          |
| GPU 수    | 수십~수백 개          | 수천 개                              |
| 네트워크  | HDR 200G              | NDR 400G Spine-Leaf                  |
| 대상      | 중간 규모 AI 워크로드 | 초대규모 LLM, Exascale AI            |
| 대표 예시 | 기업 AI팀, 연구소     | OpenAI, Meta, Microsoft, 국가 슈퍼컴 |
