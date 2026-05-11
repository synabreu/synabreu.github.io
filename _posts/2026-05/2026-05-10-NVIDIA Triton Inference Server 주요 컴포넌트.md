---
title: "NVIDIA Triton Inference Server 주요 컴포넌트"
date: 2026-05-10
tags: [nvidia, 엔비디아, GPU, AI, Triton Inference Server, ]
typora-root-url: ../
toc: true
categories: [NVIDIA]
---
NVIDIA Triton Inference Server 에서 가장 핵심적인 **Model Store, Model Analyzer, Model Performance Analyzer** 등의 주요 컴포넌트 및 도구에 대해 정리해 보도록 하겠습니다.

### 1. 전체 흐름도

[Model Store]  ← 모델 파일 보관
        ↓
[Triton Inference Server] ← 모델 로딩 & 서빙
        ↓
[Performance Analyzer] ← 단일 설정 성능 측정
[Model Analyzer]      ← 여러 설정 탐색 + 최적화
        ↓
Production Deployment


### 2. Model Store

* **역할** : 모델 아티팩트(가중치, 토크나이저, 환경변수(config), 양자화 모델 등)를 보관하고 관리하는 공간
* **Triton과의 관계** : Triton 서버는 Model Store/Repository에 있는 모델을 로딩하여 inference를 수행
* **위치** : 로컬 NVMe, NFS, S3, MinIO 등 GPU 서버가 접근 가능한 스토리지
* **주요 기능** : 모델 버전 관리, 배포 후보 제공, 롤백 및 재현성 보장

### 3. Model Analyzer

* **역할** : Triton에 모델을 올리기 전에 **여러 배포 설정(batch size, instance count, dynamic batching 등)을 자동으로 테스트**하고 최적 설정을 추천함.
* **측정 지표** : latency, throughput, GPU memory usage, GPU utilization
* **사용 목적** : Triton 배포 시 성능 최적화, GPU 자원 효율 극대화

### 4. Performance Analyzer

* **역할** : Triton에서 특정 설정 하에서 모델의 성능을 **벤치마크**하는 도구
* **차이점** : Model Analyzer가 여러 설정을 탐색하는 “자동 튜닝 도구”라면, Performance Analyzer는 **단일 설정에서 성능 측정**
* **측정 지표** : latency, throughput, GPU utilization, memory usage
* **사용 목적** : 배포 후보 성능 검증,  QoS 기준 충족 확인
