---
title: "GPU에서 Inter-node와 Intra-node 차이점"
date: 2023-11-22
tags: [NVIDIA, GPU, RDMA, 지연, Latency, MPI, NCCL, RoCE, InfiniBand,A100,DeepSpeed, DeepSpeed Zero]
typora-root-url: ../
toc: true
categories: [NVIDIA, Accelerated Computing, HPC, GPU, RDMA]
---

NCCL(NVIDIA Collective Communications Library)에서 **intra-node**와 **inter-node**는 비슷한 발음으로 헤깔리기 쉬운 데, 다음과 같이 정리해본다. 



## 1. 노드 내 통신 - Intra-node Communication 

* 정의: **같은 물리적 머신(노드) 안에 있는 GPU들 간의 통신**
* 특징:
  * **NVLink** 또는 **PCIe** 버스를 통해 GPU 간 통신
  * **속도 매우 빠름**: 대역폭 높고 지연시간 낮음
  * 예시: A100 4개가 한 서버 안에 있을 때
* **NCCL 활용**: topology-aware하게 GPU 간 직접 연결(NVLink 등)을 자동 탐지하고 최적 경로로 통신



## 2. 노드 간 통신 - Inter-node Communication

* 정의: 서로 다른 머신(노드)에 있는 GPU들 간의 통신
* 특징
  * **Ethernet**, **InfiniBand (RDMA)** 등을 통해 통신
  * **속도 느림**: 네트워크 대역폭 제약, 전송 지연 존재
  * 예시: A100이 2개씩 있는 서버 2대를 서로 연결했을 때
* **NCCL 활용**:
  * NCCL은 **NCCL-Net plugin**을 통해 **RDMA, UCX, IP-over-IB, TCP** 등을 자동 사용
  * 성능 향상을 위해 환경변수 조정 필요 (`NCCL_IB_HCA`, `NCCL_SOCKET_IFNAME` 등)



## 3. 인트라노드 vs. 인터노드 비교

| 항목             | Intra-node         | Inter-node                |
| ---------------- | ------------------ | ------------------------- |
| 위치             | 같은 노드 내 GPU들 | 다른 노드 간 GPU들        |
| 통신 방식        | NVLink, PCIe       | Ethernet, InfiniBand      |
| 대역폭           | 매우 높음          | 상대적으로 낮음           |
| 지연 시간        | 낮음               | 높음                      |
| 성능 튜닝 필요성 | 낮음 (자동 최적화) | 높음 (네트워크 설정 중요) |



## 4. **inter-node 와  cross-node 통신**

* 일반적으로 **inter-node communication**과 **cross-node communication**은 **동일한 의미**로 사용

* 서로 다른 물리적 노드(서버, 머신)에 있는 GPU 또는 프로세스 간의 통신

* 예시

  * **Intra-node**:

    * GPU0 ↔ GPU1 (같은 머신) → intra-node

  * **Inter-node (또는 Cross-node)**:

    * Node 0: GPU0, GPU1
    * Node 1: GPU2, GPU3
       → GPU1이 GPU2와 통신하는 경우 → **inter-node / cross-node**

    
