---
title: "HPC Compute에 대하여"
date: 2023-12-05
tags: [HPE, Compute, NVIDIA, GPU, HPC, AI, CUDA, NCCL, HPE ProLiant, HPE Apollo, Cray XD]
typora-root-url: ../
toc: true
categories: [HPE, HPE Compute, Accelerated Computing, HPC, HPE Cray, HPE Apollo, HPE Greenlake, NVIDIA, GPU]
---



저희 회사인 Hewlett Packard Enterprise의 서버 제품군인 HPE Compute에 대해 그동안 스터디한 내용을 한 번 정리보겠다. 덧붙여, HPE Compute는 일반 컴퓨팅, 가상화, 데이터 분석, AI, HPC 등과 같은 기업의 다양한 워크로드를 지원하기 위해 설계된 포괄적인 서버 및 컴퓨팅 인프라를 말한다. 



## 1. HPE Compute의 주요 특징

| 항목                  | 설명                                                         |
| --------------------- | ------------------------------------------------------------ |
| **유연한 포트폴리오** | 랙형, 타워형, 블레이드형, 컴포저블(Composable) 인프라까지 다양한 폼팩터 제공 |
| **워크로드 최적화**   | AI/ML, 데이터베이스, 가상화, 엣지 컴퓨팅 등 특정 워크로드에 맞춘 서버 구성 가능 |
| **HPE iLO**           | HPE만의 **Integrated Lights-Out** 관리 칩셋으로 원격 관리 및 보안 기능 제공 |
| **GreenLake 통합**    | HPE GreenLake와 연동되어 **as-a-service** 방식의 온프레미스 클라우드 구현 가능 |
| **실시간 보안 기능**  | 실리콘 루트 오브 트러스트(Silicon Root of Trust), 보안 부트 및 펌웨어 무결성 보장 |
| **에너지 효율**       | 고효율 파워 서플라이, 고밀도 설계로 데이터센터 에너지 최적화 |



## 2. 대표적인 HPE 서버 제품군 및 사양

#### 1. **HPE ProLiant DL 시리즈 (랙형)**

- **DL380 Gen11**: 가장 범용적으로 사용되는 2U, 2소켓 서버
  - 최대 2x Intel Xeon Scalable (4세대)
  - 최대 8TB DDR5 RAM
  - 최대 12x NVMe 또는 24x SAS/SATA 스토리지
  - GPU 옵션 지원 (최대 4x NVIDIA A100 가능)
  - iLO 6 탑재

#### 2. **HPE ProLiant ML 시리즈 (타워형)**

- **ML350 Gen11**: 중소기업용 타워형 서버
  - 최대 2x Xeon Scalable
  - 높은 확장성의 PCIe 슬롯
  - 사무실 환경을 고려한 저소음 설계

#### 3. **HPE Apollo 시리즈 (HPC & AI용)**

- **Apollo 6500 Gen10 Plus**: GPU 기반 HPC/AI에 특화
  - 최대 8x NVIDIA A100 또는 H100
  - NVLink 브릿지 및 NVMe 고속 저장장치 구성 가능
  - PCIe Gen4 지원, 고속 Interconnect 옵션(NIC, Infiniband)

#### 4. **HPE Cray XD 시리즈 (Exascale 기반 HPC)**

- Cray 슈퍼컴퓨터 기술 기반 고성능 서버
  - NVIDIA H100/H200 및 AMD Instinct 지원
  - Slingshot 네트워킹, 액체 냉각 지원 등 고성능 연구기관 대상

#### 5. **HPE Synergy (Composable Infrastructure)**

- 다양한 워크로드를 위한 유연한 컴포저블 아키텍처
  - 컴퓨팅, 스토리지, 네트워크 자원을 소프트웨어 정의 방식으로 구성
  - DevOps 및 자동화 환경에 적합
