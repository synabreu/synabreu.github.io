---
title: "NVIDIA Dynamo - LLM 추론을 위한 차세대 분산 서빙 플랫폼"
date: 2025-03-21
tags: [NVIDIA, Dynamo, GenAI, 쿠버네티스, Kubernetes, 추론, Inference, 분리형 서빙, Disaggregated Serving, 분산 추론, Distributed Inference, GTC2025, NVIDIA Triton Inference]
typora-root-url: ../
toc: true
---



개인적으로 이번 GTC 2025에 눈길을 끄는 것은 엔비디아  다이나모(Dynamo) 였다. 그동안 NVIDIA Triton Inference 라고 부르는 서비스가 이제 NVIDIA Dynamo 에 하나의 기능으로 변경 확장되었다.  

**Dynamo는** LLM 추론을 위한 차세대 분산 서빙 플랫폼으로, 데이터센터급의 자원 스케줄링, 메모리 계층 활용, 네트워크 전송 최적화를 통합적으로 해결하는 솔루션이다. [오픈소스로 공개되어](https://github.com/ai-dynamo) 확장성 높은 환경에서도 빠르게 실험 및 배포가 가능하다. 그러면 오늘은 Dynamo 에 대해 요약해 보겠다.



## **1. Dynamo 개발 배경 및 목표**

* **생성형 AI 분야에서는 Training 시대에서 Reasoning 시대**로 넘어가며, 추론 컴퓨팅에 더 많은 연산량과 메모리가 요구됨.
* 기존 라운드로빈 방식 라우팅이나 일체형(aggregated) 서빙으로는 LLM의 성능을 최대한 활용하기 어려움.
* 특히, LLM은 prefill (KV 캐시 생성)과 decode (토큰 생성) 두 단계가 서로 다른 자원 특성을 가지므로 분리(disaggregated serving)가 필요.
* **Dynamo의 목표**는 데이터센터 수준에서 **스케줄링, 메모리 관리, 데이터 전송 최적화**를 통해 LLM 추론을 효율적으로 가속하는 것.



## **2. 분리형 서빙(Disaggregated) 지원**

* **Prefill은 compute-bound**, **Decode는 메모리 대역폭 중심 (memory-bound)** 이므로 서로 다른 GPU 자원 사용 가능.
* KV 캐시를 prefill에서 decode로 **저지연 전송**하는 기술 필요 (NIXL 사용).
* 실제 테스트에서 2노드 이상 환경에서 **선형 이상의 성능 확장성**(better-than-linear scaling) 입증.



## **3. 지능형 라우팅인 KV 캐시 인지 라우터 지원**

* 단순 부하 기반 라우팅 대신, **KV 캐시 히트율을 고려한 라우팅**으로 **최대 3배 추론 속도 향상**.
* 각 워커의 KV 캐시와 로드를 고려한 **점수 기반 경로 선택** 구현.



## **4. KV 캐시 블록 관리로 계층적 메모리 활용**

* **HBM, 시스템 메모리, SSD, 객체 저장소**까지 포함한 메모리 계층 활용.
* 캐시 재계산 없이 빠른 prefill 가능 → **다중 턴 대화에 특히 유리**.
* 초기 구현에서 최대 **16배 추론 속도 향상** 보고됨.



## **5. 데이터 전송 라이브러리(NIXL)**

* KV 캐시 전송을 위한 **RDMA, TCP, InfiniBand** 등 다양한 전송 방식 지원.
* 중앙 컨트롤러 없이 **피어 간 연결 자동 탐색 및 확장성 확보**.
* 사용자 정의 백엔드도 플러그인 형태로 연결 가능.



## **6. 개발자 도구 및 배포**

* **Rust 기반 핵심 로직 + Python 바인딩** 제공 → 고성능 + 개발자 친화성.
* Dynamo CLI 도구 (예: `dynamo run`, `dynamo serve`, `dynamo build`, `dynamo deploy`) 제공.
* **Kubernetes 연동**, 모델 종속 설정(Tokenizer, Template 등)을 통합 관리할 수 있는 **Model Deployment Card** 도입.
* 모든 컴포넌트는 **공개 저장소(GitHub)** 및 **pip 패키지** 형태로 제공됨.
* Discord 커뮤니티로 실시간 피드백 및 지원 가능.



## **7. 실습 예제**

* [Hello World](https://github.com/ai-dynamo/dynamo/tree/main/examples/hello_world) : 간단한 다중 서비스 파이프라인을 생성하여 **Dynamo의 기본 개념**을 설명하는 예제
* [LLM Deployment](https://github.com/ai-dynamo/dynamo/tree/main/examples/llm): 다양한 구성으로 대형 언어 모델(LLM)을 배포하기 위한 예제와 참조 구현이 포함
* [LLM Deployment Examples using TensorRT-LLM](https://github.com/ai-dynamo/dynamo/tree/main/examples/tensorrt_llm): TensorRT-LLM을 사용한 LLM(대형 언어 모델) 배포 예제
* [Multimodal Deployment](https://github.com/ai-dynamo/dynamo/tree/main/examples/multimodal):  `llava-1.5-7b-hf` 모델 기반으로 Dynamo를 사용하여 멀티모달 모델을 배포하기 위한 예제 
* [LLM Deployment Examples using SGLang](https://github.com/ai-dynamo/dynamo/tree/main/examples/sglang): 내부적으로 Ingress 프로세스와 엔진 프로세스 간의 통신을 위해 ZMQ를 사용하는 SGLang을 사용하여 다양한 구성으로 대형 언어 모델(LLM)을 배포 예제



## **8. 간단한 실습**

* Ubuntu 24.04에서 Python 가상환경을 생성하고 필요한 패키지를 설치

  ```bash
  apt-get update
  DEBIAN_FRONTEND=noninteractive apt-get install -yq python3-dev python3-pip python3-venv libucx0
  python3 -m venv venv
  source venv/bin/activate
  pip install "ai-dynamo[all]"
  ```

* 예제 디렉토리로 이동하여 Dynamo 서버를 실행

  ```bash
  cd examples/llm
  dynamo serve graphs.disagg:Frontend -f ./configs/disagg.yaml
  ```

* 다른 터미널에서 클라이언트 요청을 보내 결과를 확인함

  ```bash
  curl localhost:8000/v1/chat/completions -H "Content-Type: application/json" -d '{
    "model": "deepseek-ai/DeepSeek-R1-Distill-Llama-8B",
    "messages": [{"role": "user", "content": "Hello, world!"}],
    "stream": false,
    "max_tokens": 30
  }'
  ```



## **9. 추가 참고 자료**

* [공식문서](https://docs.nvidia.com/dynamo/latest/architecture/disagg_serving.html): Prefill과 Decode 분리 아키텍처에 대해 자세히 설명
* [Distributed Inference 101: Disaggregated Serving with NVIDIA Dynamo](https://www.youtube.com/watch?v=_UDJy_5_Czw) 유투브
* [깃허브 오픈소스](https://github.com/ai-dynamo/dynamo)

