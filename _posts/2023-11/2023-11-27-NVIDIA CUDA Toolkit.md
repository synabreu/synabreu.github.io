---
title: "NVIDIA CUDA Toolkit"
date: 2023-11-27
tags: [NVIDIA, GPU, CUDA, Accelerated Computing, HPC, Visual Studio, NVIDIA Nsight Visual Studio]
typora-root-url: ../
toc: true
categories: [NVIDIA, Accelerated Computing, HPC, GPU, CUDA, Visual Studio, NVIDIA Nsight Visual Studio]
---

CUDA(Compute Unified Device Architecture)는 여러분들도 잘 아시다시피, NVIDIA의 GPU를 활용해 병렬 계산을 수행할 수 있도록 해주는 프로그래밍 플랫폼을 말한다. 이에 우리는 최신 NVIDIA CUDA Toolkit 12.8에 대해 좀 더 알아보자.



## 1. **NVIDIA CUDA Toolkit 12.8**

* 고성능 GPU 가속 애플리케이션을 개발하기 위한 개발 환경을 제공함.
* CUDA Toolkit을 사용하면 GPU 가속 임베디드 시스템, 데스크탑 워크스테이션, 엔터프라이즈 데이터 센터, 클라우드 기반 플랫폼, HPC 슈퍼컴퓨터에서 실행되는 응용 프로그램을 개발, 최적화 및 배포할 수 있음.
* GPU 가속 라이브러리, 디버깅 및 최적화 도구, C/C++ 컴파일러, 응용 프로그램 배포를 위한 런타임 라이브러리가 포함되어 있음.
* 내장된 멀티 GPU 구성을 활용한 분산화 시킨 계산(distributed computations) 기능을 사용하여, 과학자와 연구원들은 단일 GPU 워크스테이션부터 수천 개의 GPU가 있는 클라우드 설치 환경까지 확장 가능한 응용 프로그램을 개발할 수 있음.



## 2. **CUDA 주요 라이브러리**

* C/C++ Compiler, Visual Profiler
* **GPU-accelerated BLAS libary :** GPU의 병렬 처리 능력을 활용하여 선형대수의 기본 연산(벡터 및 행렬 연산 등)을 고속으로 수행할 수 있도록 최적화된 BLAS (Basic Linear Algebra Subprograms) 함수들을 제공하는 라이브러리. ex) cuBLAS
* **GPU-accelerated FFT library :** GPU의 병렬 처리 능력을 활용하여 고속 푸리에 변환(FFT, Fast Fourier Transform) 연산을 수행할 수 있도록 최적화된 라이브러리. ex) 신호 처리, 이미지 처리, 과학 및 공학 분야에서 주파수 분석, 필터링, 스펙트럼 분석
* **GPU-accelerated Sparse Matrix library(cuSPARSE) :** GPU의 병렬 처리 능력을 활용하여 희소 행렬(sparse matrix) 연산을 효율적으로 수행할 수 있도록 최적화된 SW 라이브러리. ex) 고성능 컴퓨팅, 과학 계산, 머신러닝, 그래프 분석
  - 희소 행렬은 대부분의 원소가 0인 행렬을 의미하며, 이러한 행렬에 대한 연산은 메모리 사용과 계산 효율성을 고려할 때 특별한 처리가 필요.
  - CUDA 아키텍처 기반 GPU에서 희소 행렬-벡터 곱셈, 희소 행렬-행렬 곱셈 등과 같은 다양한 희소 행렬 연산을 빠르고 효율적으로 수행
    - **GPU-accelerated RNG library(cuRand) :** GPU의 병렬 처리 능력을 활용하여 난수(Random Number)를 빠르고 효율적으로 생성할 수 있도록 최적화된 소프트웨어. CUDA 아키텍처를 기반으로 하며, 다양한 난수 생성 알고리즘을 지원하여 GPU에서 직접 난수를 생성. ex) 대규모 데이터나 고성능 계산 환경, 고성능 컴퓨팅, 통계 시뮬레이션, 게임 개발, 보안 응용 프로그램



## 3. **CUDA 주요 특징**

* **더 쉬운 애플리케이션 포팅**

  * 멀티 쓰레드 간의 GPU 공유
  * 단일 호스트 스레드에서 시스템의 모든 GPU를 동시에 사용
  * 시스템 메모리를 복사 없이 핀 고정(No-copy pinning), cudaMallocHost()보다 빠른 대안 제공
  * C++의 new/delete 및 가상 함수 지원
  * 인라인 PTX 어셈블리 지원
  * sort, reduce 등 템플릿화된 성능 기본 기능을 제공하는 Thrust 라이브러리
  * 이미지/비디오 처리를 위한 NPP(Nvidia Performance Primitives) 라이브러리
  * 동일 크기/포맷 텍스처를 더 큰 크기와 높은 성능으로 다루기 위한 계층화 텍스처

* 더 빠른 멀티 GPU 프로그래밍

  * Unified Virtual Addressing: 멀티 GPU 환경에서 프로그래밍을 **더 간편하고 빠르게** 만들어주는 메모리 주소 공간 통합 기능이며, 이를 통해 CPU와 GPU, 그리고 여러 GPU 간의 **가상 주소 공간을 하나로 통일**시킴. **CPU, 모든 GPU, 그리고 pageable memory**가 **하나의 통합된 가상 주소 공간**을 공유하도록 하는 CUDA 4.0 부터 지원함.
  * Peer-to-Peer 커뮤니케이션용 GPUDirect v2.0 지원

* 새로운 또는 향상된 개발 도구 지원

  * NVIDIA Hopper (H100) 및 Ada Lovelace (L40/L40S) 아키텍처 지원

  * ARM 서버 프로세서 지원

  * 지연 모듈 및 커널 로딩(Lazy Module and Kernel Loading)

  * 개편된 동적 병렬성 API(Dynamic Parallelism APIs)

  * 개선된 CUDA 그래프 API, 성능 최적화된 라이브러리 및 새로운 개발자 도구 기능

  * NVIDIA Hopper 아키텍처 지원 - 차세대 텐서 코어 및 트랜스포머 엔진, 고속 NVLink 스위치 시스템, 혼합 정밀도 모드, 2세대 다중 인스턴스 GPU(MIG), 고급 메모리 관리, 표준 C++/Fortran/Python 병렬 언어 구성 요소

    

## 4. **CUDA 12 새로운 기능**

* Visual Profiler에서 자동화된 성능 분석
* Linux와 MacOS용 CUDA-GDB에서의 C++ 디버깅
* Fermi 아키텍처용 GPU 바이너리 디스어셈블러 (cuobjdump)
* 새로운 디버깅 및 프로파일링 기능을 갖춘 [Parallel Nsight 2.0이](https://developer.nvidia.com/nsight-visual-studio-edition) Windows 개발자용으로 이제 사용 가능함



## 5. **예제 소스**

* Visual Studio 2022 Community Edition + NVIDIA Nsight Visual Studio Edition 2025.1 버전에서 직접 프로그래밍을 해보고 테스트한 예제 소스 공개
  * [HelloCuda](https://github.com/synabreu/nvidia-note/tree/main/CudaWorkshop/HelloCuda)
  * [CheckDeviceMemory](https://github.com/synabreu/nvidia-note/tree/main/CudaWorkshop/CheckDeviceMemory)



## 6. **CUDA 테크 블로그**

* [Dynamic Loading in the CUDA Runtime](https://developer.nvidia.com/blog/dynamic-loading-in-the-cuda-runtime/?ncid=so-face-314879&linkId=100000336560009)
* [CUDA Context-Independent Module Loading](https://developer.nvidia.com/blog/cuda-context-independent-module-loading/)
* [Constructing CUDA Graphs with Dynamic Parameters](https://developer.nvidia.com/blog/constructing-cuda-graphs-with-dynamic-parameters/)
* [Improving GPU Application Performance with NVIDIA CUDA 11.2 Device Link Time Optimization](https://developer.nvidia.com/blog/improving-gpu-app-performance-with-cuda-11-2-device-lto/)
* [CUDA Pro Tip: Control GPU Visibility with CUDA_VISIBLE_DEVICES](https://developer.nvidia.com/blog/cuda-pro-tip-control-gpu-visibility-cuda_visible_devices/)
* [CUDA Pro Tip: Understand Fat Binaries and JIT Caching](https://developer.nvidia.com/blog/cuda-pro-tip-understand-fat-binaries-jit-caching/)
