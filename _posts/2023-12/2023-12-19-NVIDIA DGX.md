---
title: "NVIDIA DGX"
date: 2023-12-19
tags: [NVIDIA, GPU, RDMA, DGX, InfiniBand, Spine-Leaf, NDR, NCCL]
typora-root-url: ../
toc: true
---



**NVIDIA DGX는 단일 서버**으로써 고성능 AI 모델 훈련과 추론을 위한 **All-in-One GPU 컴퓨팅 시스템**이다. 기업이나 연구기관이 **멀티-GPU 학습 환경**을 손쉽게 구축할 수 있도록 NVIDIA가 직접 설계한 서버 제품군이며, 대표적으로는 **DGX A100**과 **DGX H100** 모델이 있다.

그러므로, **DGX 서버는 1대만으로도 완전한 멀티-GPU 딥러닝 훈련/추론 환경**을 제공하는 데,  내부에 고속 GPU 간 연결 (NVLink), 대용량 메모리, 스토리지, 고속 네트워크 NIC가 통합되어 있어, **"AI 데스크탑 슈퍼컴퓨터"**라고도 부른다. 

그렇다면, DGX에 대해 주요 모델 비교와 구성 요소, 기본 소프트웨어, 주의 사항 등에 대해 한 번 알아 보자!



## 1. 주요 모델 비교

| 항목          | DGX A100                              | DGX H100                                  |
| ------------- | ------------------------------------- | ----------------------------------------- |
| GPU           | 8× A100 80GB                          | 8× H100 80GB                              |
| GPU 간 연결   | **NVLink + NVSwitch (600 GB/s 이상)** | **NVLink 4 + NVSwitch 3 (900 GB/s 이상)** |
| CPU           | 2× AMD EPYC 7742 (64코어)             | 2× Intel Xeon Platinum 8480+ (56코어)     |
| 시스템 메모리 | 1TB DDR4                              | 2TB DDR5                                  |
| 저장소        | 15TB NVMe SSD                         | 30TB NVMe SSD                             |
| 네트워크      | 8× 200Gbps HDR InfiniBand             | 8× 400Gbps NDR InfiniBand                 |
| 소비전력      | 6.5 kW                                | 10.2 kW                                   |
| 크기          | 6U 랙 마운트                          | 6U 랙 마운트                              |



## 2. 구성 요소

| 구성           | 설명                                                         |
| -------------- | ------------------------------------------------------------ |
| **GPU**        | 8개 GPU가 **NVLink**로 완전 연결 (All-to-All), AllReduce 최적화 |
| **CPU**        | 멀티 코어 서버급 CPU로 I/O와 전처리 연산 지원                |
| **NIC**        | 200~400Gbps의 InfiniBand NIC를 8개 탑재 – 멀티 노드 확장 대응 |
| **스토리지**   | 로컬 고속 NVMe SSD 내장 (OS, 데이터 저장)                    |
| **운영체제**   | Ubuntu 기반 + NVIDIA 커널 모듈 최적화                        |
| **소프트웨어** | NVIDIA NGC 컨테이너, Docker, NCCL, CUDA, PyTorch, TensorFlow 등 사전 설치됨 |



## 3. 기본 탑재 소프트웨어

| 소프트웨어                     | 설명                                       |
| ------------------------------ | ------------------------------------------ |
| **Base OS**                    | Ubuntu 또는 RHEL 기반                      |
| **NGC Container Registry**     | 사전 빌드된 PyTorch, TensorFlow, Triton 등 |
| **NCCL**                       | 고속 AllReduce, 멀티GPU 통신 최적화        |
| **CUDA Toolkit**               | GPU 프로그래밍 툴킷                        |
| **DCGM**                       | GPU 상태 모니터링 도구                     |
| **Base Command Manager (BCM)** | Slurm 기반 워크로드 관리, 기본 포함        |



## 4. 사용 예시

| 예시                 | 내용                                       |
| -------------------- | ------------------------------------------ |
| **LLM 파인튜닝**     | LoRA 또는 PEFT 방식으로 8개 GPU 활용       |
| **GenAI 추론**       | Triton Inference Server로 대규모 응답 처리 |
| **멀티GPU DDP 훈련** | `torchrun --nproc_per_node=8` 형태로 훈련  |
| **데이터 병렬 처리** | CUDA Stream, NCCL 통신 활용                |
| **Fine-tuning BERT** | 1~2시간 내로 고속 처리 가능 (A100 기준)    |



## 5. 장점

* 즉시 사용 가능한 AI 슈퍼컴 환경
* GPU 간 **고속 직접 연결 (NVLink)** 덕분에 **inter-GPU 통신 병목 최소화**
* **최적화된 소프트웨어 환경**으로 빠른 배포 가능
* 멀티 노드 클러스터로 **확장 가능성 내재**
* NVIDIA 기술지원 및 NGC 활용 가능



## 6. 주의사항

* 6U 공간, 고전력 소비(6~10kW 이상)를 요구하므로 **데이터센터급 인프라 필요** 
* 수냉식(air-cooled 또는 liquid-cooled) 버전 구분 필요 
* Base Command Platform(클라우드 기반)은 별도 구매임



## 7. 요약

* **NVIDIA DGX 단일 서버는 완전한 멀티-GPU 딥러닝 인프라를 단일 박스로 제공**하며, 기업/연구기관이 자체 AI 모델 훈련을 위한 최적의 장비로 사용됨.