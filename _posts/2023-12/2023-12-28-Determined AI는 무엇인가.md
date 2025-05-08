---
title: "Determined AI는 무엇인가"
date: 2023-12-28
tags: [NVIDIA, GPU, CUDA, NVLink, NVSwitch, GraceHopper, NVLink Switch, NVIDIA Blackwell]
typora-root-url: ../
---

지난번 HPE MLDE에 대해 간략하게 알아보았다. 사실 HPE MLDE의 핵심 엔진은 바로 Determined AI이다. HPE는 2021년 AI 스타트업 Determined AI를 인수하면서, 이 플랫폼을 MLDE의 기반 기술로 삼았다. 따라서 MLDE의 내부 아키텍처는 사실상 Determined AI의 아키텍처와 거의 동일하다.

Determined AI는 분산 딥러닝 학습에 최적화된 오픈소스 플랫폼이다. 주요 목표는 효율적인 GPU 자원 활용, 실험 추적, 분산 학습 자동화이다. 그렇다면, Determined AI 아키텍처 주요 구성에 대해 알아보자.



![그림 - Determind AI 아키텍처](/../images/2023-12/DAI-01.png)



##  **1. Master (Control Plane)**

* **역할:** 전체 클러스터와 실험을 관리하는 중앙 관리자
* **기능:**
  - 실험 스케줄링
  - 클러스터 상태 모니터링
  - 사용자 인증 및 역할 관리
  - Web UI/API 제공
* **배포:** 단일 Pod 또는 HA(고가용성) 모드 가능



##  **2. Agent (Worker Node)**

* **역할:** 실제 GPU 작업을 실행하는 노드
* **기능:**
  - GPU 상태 수집 및 마스터에 보고
  - 할당된 작업 컨테이너 실행
* **특징:**
  - 각 Agent는 여러 GPU를 보유 가능
  - GPU 단위로 작업이 할당됨



##  **3. Task Container (Trial Worker)**

* **역할:** 하나의 실험(Trial)을 실행하는 컨테이너

* **구성:**

  - 사용자 모델 코드 + Determined runtime

  - Python SDK 및 Callback 포함

* **특징:**
  - 여러 Trial이 병렬 실행 가능 (Hyperparameter tuning 등에 유용)



## **4. Distributed Training Support**

* **기능:** Horovod, PyTorch DDP, DeepSpeed 등을 통한 분산 학습 자동화
* **특징:**
  - 유저가 복잡한 통신 코드를 작성할 필요 없음
  - Determined가 자동으로 GPU/NODE를 조율하고 Launch



## **5. Experiment Tracking & Checkpointing**

* **기능:**
  - 하이퍼파라미터, 성능 로그, 학습 곡선 자동 기록
  - 중간 Checkpoint 저장 및 복원
* **스토리지 연동:**
  - S3, NFS, HDFS, GCS 등과 통합 가능



## **6. Web UI / Python SDK / REST API**

* **Web UI:** 실험 관리, 리소스 모니터링, 체크포인트 비교 등
* **Python SDK:** 실험 정의, 모델 제출, 결과 분석
* **REST API:** 자동화된 MLOps 워크플로우 통합



## **7. 결론**

* HPE MLDE는 단순히 Python 코드만 실행하는 환경이 아니라 Determined AI 기반의 아키텍처로  **자동 분산 학습, GPU 자원 최적 스케줄링, 학습 실험 자동 추적**을 제공한다.
* 고객이 수십 개의 실험을 동시에 관리하고, 고성능 GPU 자원을 낭비 없이 활용할 수 있는 **AI 엔지니어링 플랫폼**이다.





