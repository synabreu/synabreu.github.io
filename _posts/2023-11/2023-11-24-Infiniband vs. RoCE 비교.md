---
title: "Infiniband vs. RoCE 비교"
date: 2023-11-24
tags: [NVIDIA, GPU, RDMA, Infiniband, RoCE, 인피니밴드]
typora-root-url: ../
toc: true
categories: [NVIDIA, Accelerated Computing, HPC, RDMA, GPU, RoCE, Infiniband]
---

HPC&AI 고성능 네트워킹에서 NVIDIA의 인피니티밴드(Infiniband)과 RoCE(RDMA over Converged Ethernet) 기술은 양대 산맥으로 흔히 사용한다.

두 기술 모두 고속 네트워크 인터커넥트 기술로, 특히 HPC, AI, 머신러닝, 데이터센터에서 사용되는 기술인데, 이에 대해 좀 더 알아보자!



## 1. **InfiniBand vs. RoCE 개요**

| **항목**            | **InfiniBand (IB)**                   | **RoCE (RDMA over Converged Ethernet)**  |
| :------------------ | :------------------------------------ | :--------------------------------------- |
| **기반 네트워크**   | InfiniBand 전용 네트워크              | 기존 이더넷 (Converged Ethernet)         |
| **RDMA 지원**       | 네이티브 RDMA 지원                    | 이더넷 상의 RDMA (RDMA over Ethernet)    |
| **네트워크 속도**   | 최대 1,200Gbps (HDR, NDR, XDR)        | 최대 800Gbps (Ethernet 800G)             |
| **지연 시간**       | 1µs 미만 (0.6~1.0µs)                  | 약 1~2µs                                 |
| **품질 보장 (QoS)** | 크레딧 기반 흐름 제어 (Lossless 전송) | PFC (Priority-based Flow Control) 필요   |
| **확장성**          | 대규모 HPC 클러스터에 적합            | 일반 데이터센터 환경에서 우수한 확장성   |
| **주요 사용처**     | HPC, AI, 슈퍼컴퓨터, 클라우드         | 데이터센터, AI 학습/추론, 금융, 스토리지 |



## 2. 주요 비교 분석

* 네트워크 기반
  * InfiniBand: Mellanox(NVIDIA)에서 주도하는 전용 네트워크 프로토콜로, 고성능과 낮은 지연 시간을 보장.
  * RoCE: 기존 이더넷 위에서 RDMA 기능을 구현하며, 스위치 및 기존 인프라 활용 가능.
* 성능 (Throughput & Latency)
  * InfiniBand: 일반적으로 RoCE보다 더 낮은 지연 시간과 높은 성능을 제공.
  * 최신 NVIDIA Quantum-2 IB 스위치는 최대 400Gbps 속도를 지원하며, 지연 시간은 1μs 미만으로 낮음.
  * RoCE v2는 1~2μs 지연 시간이지만, 고속 이더넷(400GbE, 800GbE)이 등장하면서 성능 격차가 줄어듦.
*  QoS 및 신뢰성
  * InfiniBand: 크레딧 기반 흐름 제어(Flow Control)를 사용해 Lossless 전송을 보장.
  * RoCE: 이더넷 기반이므로 PFC(Priority Flow Control)과 ECN(Explicit Congestion Notification)을 활용하여 Lossless 환경을 구현해야 함.
    * 네트워크 설정이 제대로 안 되면 패킷 손실이 발생할 수 있음.
* 확장성 및 비용
  * InfiniBand: 별도의 네트워크 장비(스위치, NIC 등)가 필요하며, 구축 비용이 높음.
  * RoCE: 기존 이더넷 인프라에서 활용 가능하므로 비용이 낮고, 클라우드 및 데이터센터에 최적화됨.
* 소프트웨어 및 호환성
  * InfiniBand: NVIDIA HPC-X, OpenMPI, Slurm, MLNX_OFED 등을 통해 HPC 환경에 최적화.
  * RoCE: NVMe-oF(Storage), AI 클러스터, 클라우드(NIC 직접 연결) 등에서 더 많이 사용됨.



## 3. 환경 분석

| **사용 사례**                                        | **추천 솔루션**            |
| :--------------------------------------------------- | :------------------------- |
| HPC, 슈퍼컴퓨터, AI 훈련 (LLM, GPT-4 등)             | InfiniBand                 |
| 데이터센터, 클라우드, 일반적인 AI/ML                 | RoCE                       |
| 스토리지 (NVMe-oF, GPUDirect Storage)                | RoCE v2                    |
| 고성능 금융 애플리케이션 (Algorithm Trading, HFT 등) | InfiniBand (초저지연 필수) |
| 기존 이더넷 인프라 활용                              | RoCE                       |
| 데이터센터 내 멀티 테넌시 및 확장성 고려             | RoCE                       |



## 4. 결론

* InfiniBand: 최고의 성능과 최소 지연 시간을 요구하는 AI, HPC, 금융, 슈퍼컴퓨터 등에 적합.
* RoCE: 기존 이더넷 기반 데이터센터, 클라우드, AI/ML, 스토리지 환경에서 비용 효율적이고 확장성이 뛰어남.



결론적으로 말해서, **HPC 및 초저지연이 중요한 환경이라면 InfiniBand, 기존 이더넷 인프라와의 통합을 원한다면 RoCE가 적합함.**
