---
title: "NVLink 와 NVLInk 스위치"
date: 2023-12-04
tags: [NVIDIA, GPU, CUDA, NVLink, NVSwitch, GraceHopper, NVLink Switch, NVIDIA Blackwell]
typora-root-url: ../
toc: true
categories: [NVIDIA, CPU, GPU, NVLink, NVSwitch, Infiniband, RDMA, Accelerated Computing, HPC]
---

NVLink 와 NVLink Switch 는 대규모 데이터셋을 모델에 더 빠르게 공급하고, GPU 간 데이터를 신속하게 교환하기 위한 고속 멀티-GPU 통신의 기본 구성 요소이다. 



## 1. 더 빠른 확장형 인터커넥트의 필요성

* 엑사스케일 컴퓨팅(exascale computing)과 **수조 개 파라미터를 가진 AI 모델**의 잠재력을 완전히 실현하기 위해서는, 서버 클러스터 내 **모든 GPU 간에 빠르고 매끄러운 통신**이 필수적
* **5세대 NVIDIA NVLink**는 이러한 요구에 맞춘 **확장형(scale-up) 인터커넥트**로,
   AI 추론(AI reasoning)을 가속화하고, **수조 개 파라미터를 가진 모델의 추론 성능을 극대화**



## 2. NVIDIA NVLink로 시스템 처리량 극대화

* **5세대 NVLink**는 **멀티-GPU 시스템의 확장성**을 대폭 향상
* **훈련(training)**, **추론(inference)**, **추론 기반 추리(reasoning)** 워크플로우에서
   GPU들이 **메모리와 연산을 공유**할 수 있도록 지원
* 하나의 **NVIDIA Blackwell GPU**는 최대 **18개의 NVLink 연결(각 100GB/s)** 을 지원하며, 총 **1.8TB/s**의 대역폭을 제공함 -> 이전 세대 대비 **2배**, 그리고 **PCIe Gen5 대비 14배 이상의 대역폭**
* **NVIDIA GB300 NVL72**와 같은 서버 플랫폼은 이 기술을 활용하여, 오늘날 가장 복잡한 대규모 AI 모델에 대해 **더 뛰어난 확장성과 처리 성능**을 제공



## 3. NVLink 통신으로 추론 처리량 향상

* NVLink 통신으로 추론 처리량 향상 -> NVLink 스위치 칩 (NVLink Switch Chip)
* NVIDIA NVLink 및 NVLink Switch로 GPU 완전 연결
  * **NVLink**는 **양방향 1.8TB/s** 속도를 제공하는 **직접 GPU-간 연결(Direct GPU-to-GPU Interconnect)** 기술로, 서버 내에서 **멀티-GPU 입력 및 출력(IO)** 확장을 가능
  * **NVIDIA NVLink Switch 칩**은 여러 개의 NVLink를 연결하여,
     **단일 랙 내부뿐 아니라 랙 간에서도 모든 GPU 간(all-to-all) 통신을 NVLink 속도로 지원**
* 고속 집단 연산을 위한 SHARP 지원
  * 각 **NVLink Switch**에는 **NVIDIA SHARP(Scalable Hierarchical Aggregation and Reduction Protocol)** 엔진이 내장
  * 네트워크 상에서의 **집계 연산(in-network reductions)** 및 **멀티캐스트 가속(multicast acceleration)**을 가능 -> **고속 집합 연산(collective operations)**을 보다 효율적으로 수행
* 더 읽어 볼 것:  [NVIDIA NVLink and NVIDIA NVSwitch Supercharge Large Language Model Inference](https://developer.nvidia.com/blog/nvidia-nvlink-and-nvidia-nvswitch-supercharge-large-language-model-inference/?ncid=so-face-325963)



## 4. NVLink 스위치 시스템으로 수조 파라미터 모델의 추론 속도 가속(update!)

* **NVLink Switch**를 사용하면 NVLink 연결을 **노드 간으로 확장**하여,**매끄럽고 고대역폭의 멀티 노드 GPU 클러스터**를 구성
* 실질적으로 **데이터센터 규모의 단일 GPU 시스템**을 형성하는 것
* **NVIDIA NVLink Switch**는 **GB300 NVL72 시스템 하나에서 130TB/s에 달하는 GPU 대역폭**을 제공하며,**대규모 모델 병렬 처리(large model parallelism)**에 최적화되어 있음
* NVLink 기반의 멀티 서버 클러스터는, **연산량 증가에 맞춰 GPU 간 통신도 함께 확장되도록 설계되어**  NVL72는 **기존 8-GPU 시스템 대비 최대 9배 많은 GPU 수**를 지원
* 더 읽어 볼 것: [NVIDIA GB300 NVL72](https://www.nvidia.com/en-us/data-center/gb300-nvl72/?ncid=so-face-325963)



## 5. NVIDIA NVLink 스위치

* **총 144개의 NVLink 포트**를 갖추고 있으며,**14.4TB/s의 논블로킹(non-blocking) 스위칭 용량**을 제공함
* 랙 스위치는 **5세대 NVLink 외부 연결**을 지원하는 **NVIDIA GB300 NVL72 시스템**에
   **고대역폭 및 저지연 통신**을 제공하도록 설계되었음



