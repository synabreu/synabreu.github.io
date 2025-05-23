---
title: "NVIDIA vGPU(1)-vGPU 개념과 아키텍처"
date: 2024-02-01
tags: [NVIDIA, GPU, RDMA, vGPU, HyperVisor]
typora-root-url: ../
toc: true
categories: [NVIDIA, Accelerated Computing, HPC, GPU, vGPU]

---

NVIDIA의 **vGPU (Virtual GPU)**는 물리적인 GPU 리소스를 여러 가상 머신(VM)이나 컨테이너에서 동시에 사용할 수 있도록 해주는 **가상화 기술**을 말한다. NVIDIA의 **vGPU 소프트웨어 스택**을 통해 구현되며, 주로 가상 데스크톱 인프라(VDI), AI, HPC, CAD/CAM, 3D 그래픽, Deep Learning 워크로드 등에서 활용하는 데 사용한다.



## 1. vGPU 개념

![그림 - vGPU 전체 다이어그램](/../images/2023-11/vGPU-02.png)

* vGPU
  * **GPU의 효율적인 공유**: 하나의 물리 GPU를 여러 VM 또는 컨테이너에서 나눠 사용
  * **GPU 리소스 격리**: 사용자 또는 워크로드 간 간섭 없이 사용
  * **유연한 워크로드 할당**: 필요한 GPU 리소스만큼 할당 및 스케일
* vGPU 아키텍처
  * NVIDIA의 vGPU(Virtual GPU) 아키텍처는 물리적인 GPU를 가상화하여 여러 가상 머신(VM)에서 동시에 사용할 수 있도록 설계된 구조
  * NVIDIA Virtual GPU Manager를 통해 하이퍼바이저에서 물리 GPU를 제어하며, 각 VM은 전용 드라이버를 통해 GPU 리소스에 접근함.
* NVIDIA vGPU 아키텍처 구성 요소
  * **물리 GPU (Physical GPU)**: 서버에 설치된 NVIDIA GPU로, 가상화된 환경에서 여러 vGPU 인스턴스를 지원
  * **NVIDIA Virtual GPU Manager**: 하이퍼바이저에 설치되어 물리 GPU를 가상화하고, 각 VM에 vGPU를 할당하는 역할
  * **하이퍼바이저 (Hypervisor)**: VMware vSphere, Citrix Hypervisor, Red Hat KVM 등 다양한 플랫폼에서 NVIDIA vGPU를 지원
  * **가상 머신 (VM)**: 각 VM은 NVIDIA 드라이버를 설치하여 vGPU를 통해 GPU 리소스를 활용
  * **vGPU 프로파일**: 각 vGPU는 메모리 크기, 성능 수준, CUDA 지원 여부 등을 정의하는 프로파일을 가짐



## 2. vGPU 아키텍처 구성도

![그림 2 - vGPU 아키텍처](/../images/2023-11/vGPU-01.png)

* 하드웨어 계층
  * **NVIDIA GPU**: A100, L40, H100 등
  * **vGPU를 지원하는 서버**: Dell, HPE, Lenovo 등 주요 OEM 서버
  * **GPU 가상화 기능 활성화**: BIOS에서 SR-IOV 또는 GPU 파티셔닝 기능 필요
* Hypervisor 계층
  * **지원 하이퍼바이저**: VMware vSphere (ESXi), Citrix Hypervisor, Red Hat KVM, NVIDIA의 NVIDIA AI Enterprise for vGPU 
* vGPU Manager (Host Driver)
  * Hypervisor에 설치되어 **GPU와 VM 간의 리소스를 관리**
  * 각 VM에 할당된 vGPU 프로파일을 통해 리소스 양과 기능 결정
* VM 계층 (Guest OS)
  * 각 VM은 **vGPU 프로파일에 따라 GPU 리소스를 분할해 사용**
  * NVIDIA 게스트 드라이버 설치 필요
  * CUDA, OpenGL, DirectX, TensorRT, PyTorch, TensorFlow 등 사용 가능



## 3. vGPU 프로파일

* 각 GPU는 여러 개의 **vGPU 프로파일**로 나뉘며, 이 프로파일은 다음을 정의

  * **메모리 용량** (예: 1GB, 4GB, 16GB 등)
  * **GPU 성능** (전용 스케줄링 여부)
  * CUDA 이용 가능 여부
  * **사용 시나리오** (Graphics, Compute, VDI 등)

* 예시

  | vGPU 프로파일 이름 | 전용 메모리 | 용도                         |
  | ------------------ | ----------- | ---------------------------- |
  | `A100-4C`          | 4GB         | AI 추론용                    |
  | `A100-20C`         | 20GB        | AI 학습용                    |
  | `L40-1Q`           | 1GB         | Virtual Workstation (Quadro) |
  | `L40-16Q`          | 16GB        | 고급 그래픽 처리용           |



## 4. vGPU 사용 시나리오

| 용도                   | 설명                                          |
| ---------------------- | --------------------------------------------- |
| VDI                    | 디자이너, 개발자 등 고성능 그래픽 환경 지원   |
| AI Training            | 여러 사용자 또는 프로젝트에 GPU 리소스를 공유 |
| AI Inference           | 추론 워크로드를 병렬로 처리                   |
| HPC Simulation         | 수치해석 또는 병렬 시뮬레이션 처리            |
| 클라우드 게임 스트리밍 | GPU 리소스를 플레이어 단위로 분리             |



## 5. vGPU 장점 및 고려사항

* 장점
  * **TCO 절감**: GPU 리소스를 효율적으로 공유하여 비용 절감
  * **워크로드 유연성**: 다양한 용도에 따라 프로파일 선택
  * **엔터프라이즈 지원**: 보안, 모니터링, 할당제어 등 강화된 기능
* 고려사항
  * **라이선스 필요**: vGPU 기능은 **NVIDIA vGPU 라이선스** 필요
  * **성능 분할**에 따른 **성능 저하 가능성**
  * 일부 고성능 워크로드는 **전용 GPU가 더 적합**



## 6. 관련 제품군

| 제품명                                   | 설명                                    |
| ---------------------------------------- | --------------------------------------- |
| **NVIDIA vGPU Software**                 | vGPU 가상화를 위한 핵심 소프트웨어 스택 |
| **NVIDIA AI Enterprise**                 | AI용 가상화 환경 제공 (vGPU 포함)       |
| **NVIDIA RTX Virtual Workstation (vWS)** | 그래픽 및 CAD 가상화                    |
| **NVIDIA Virtual Compute Server (vCS)**  | AI/HPC 용도로 CUDA 가상화 지원          |



## 7. 참고 자료

* [NVIDIA Virtual GPU Software User Guide](https://docs.nvidia.com/vgpu/13.0/grid-vgpu-user-guide/index.html)