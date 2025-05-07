---
title: "2023-12-20-NVIDIA DGX POD.md"
date: 2023-12-20
tags: [NVIDIA, GPU, RDMA, DGX, InfiniBand, Spine-Leaf, NDR, NCCL]
typora-root-url: ../
---

NVIDIA DGX POD는 여러 대의 DGX 시스템 (A100 또는 H100) 을 고속 스토리지, 스파인-리프 네트워크, 그리고 AI 소프트웨어 스택과 함께 구성한 AI 슈퍼컴퓨팅 인프라 유닛을 말한다.  실제 구성 컴포넌트는 DGX 서버인 Computer Layer, 고속 네트워크인 Fabric Layer, 고속 스토리지인 Storage Layer, 소프트웨어 스택 등으로 나뉜다. 



## 1. DGX 서버 (Compute Layer)

* 예: `DGX A100` 또는 `DGX H100` 서버 4~32대
  * 각 서버당:
    * 8× NVIDIA A100/H100 GPU (NVLink로 연결됨)
    * 2× AMD EPYC CPU
    * 1TB RAM 이상
    * 2× 100GbE 또는 2× HDR/200Gbps InfiniBand
* 같은 노드 내 GPU들은 **NVLink**로 직접 연결
* 노드 간은 **NCCL + InfiniBand** 기반 **inter-node** 통신



## 2. 고속 네트워크 (Fabric Layer)

* 스위치 토폴로지: **Spine-Leaf 구조**
* 네트워크 유형:
  - **InfiniBand HDR(200Gbps)** 또는 **NDR(400Gbps)** 스위치
    - 예: **NVIDIA Quantum InfiniBand Switch**
* 노드 당 최소 **2× NIC** → Spine 스위치로 연결
* Leaf 스위치 수 = DGX 수만큼 (보통 Leaf 1대당 DGX 1~2대)
* 모든 Leaf는 모든 Spine에 연결 (**non-blocking CLOS fabric** 구성)
* NCCL은 IB RDMA를 자동 활용 (`NCCL_IB_HCA` 환경변수 등)



## 3. 고속 스토리지 (Storage Layer)

* 예) **DDN A³I**, **NetApp ONTAP AI**, **VAST**, **WekaIO**
* 프로토콜: **NVMe over Fabric**, **RDMA**, **NFS over InfiniBand**
* 기능: 
  * AI 훈련 중 **checkpoints**, **dataset shard** 빠르게 읽기/쓰기
  * GPU Direct Storage (GDS)로 GPU ↔ 스토리지 직접 접근



## 4. 소프트웨어 스택

* 운영체제: Ubuntu 또는 RHEL
* AI/DL 프레임워크:  PyTorch, TensorFlow, JAX 등 **최적화된 NVIDIA 컨테이너**
* 도구: **NVIDIA NGC**, **NCCL**, **cuDNN**, **CUDA Toolkit**
* 멀티 노드 훈련용: **Slurm**, **Kubernetes + KubeFlow**, **MLFlow**, **NVIDIA Base Command**



## 5. 실제 아키텍처 예시: DGX POD 8노드 구성

| 구성 요소         | 세부 스펙                           |
| ----------------- | ----------------------------------- |
| DGX 시스템 수     | 8대 (DGX A100 기준)                 |
| GPU 수            | 8 GPU × 8 노드 = 64 A100            |
| 네트워크          | HDR 200Gbps InfiniBand / Leaf-Spine |
| 스토리지          | DDN AI400X (NVMe 기반 GDS 지원)     |
| 소프트웨어        | PyTorch DDP + NCCL + Slurm          |
| 네트워크 레이아웃 | 8 Leaf ↔ 2 Spine, full mesh CLOS    |



## 6. 왜 DGX POD인가?

| 장점                | 설명                                                 |
| ------------------- | ---------------------------------------------------- |
| **확장성**          | 노드를 8 → 32 → 128까지 쉽게 확장                    |
| **높은 GPU 활용률** | NVLink, NCCL, IB 최적화로 멀티노드/멀티GPU 효율 증대 |
| **최적화된 스택**   | NVIDIA NGC 컨테이너 + GDS + NCCL 통합                |
| **효율적인 학습**   | LLM, Vision, GenAI 학습 시간 단축 (ex: GPT, BERT)    |
