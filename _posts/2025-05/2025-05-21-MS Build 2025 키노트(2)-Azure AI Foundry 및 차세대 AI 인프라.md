---
title: "MS Build 2025 키노트(2)-Azure AI Foundry 및 차세대 AI 인프라"
date: 2025-05-21
tags: [마이크로소프트, Microsoft, Azure AI Foundary, Azure AI Agent, GitHub Copilot, Github, Visual Studio, Copilot, 코파일럿, OpenAI, Codex, Microsoft 365 Copilot, Copilot Studio, Teams, Azure, NL Web, SQL Server 2025, Microsoft Fabric ]
typora-root-url: ../
toc: true
categories: [Microsoft, Build]
---

이번 마이크로소프트 Build 2025가 너무 길어서 파트2로 나누어서 요약하겠다. 특히, AI 중심 앱 개발을 위한 풀스택 플랫폼인 Azure AI Foundry 와 다양한 AI 모델과 RAG, 에이전트 오케스트레이션, 보안, 컴퓨팅 유연성, 운영 가시성을 통합하여,기업이 Copilot 수준의 지능형 애플리케이션을 직접 구축하고 운영할 수 있도록 지원하는 얘기부터 전하고 최상의 AI 성능을 최저 비용과 최대 규모로 제공하는 인프라 전략 등을 주요로 설명하겠다. 



![그림 1 - Azure AI Foundry](/../images/2025-05/build25-02-01.jpg)

## 1. Azure AI Foundry: AI 시대를 위한 차세대 애플리케이션 서버 

* Azure AI Foundry: Copilot 및 Copilot Studio를 구성하는 핵심 기술을 개발자들이 자유롭게 사용할 수 있도록 플랫폼 제공
* Foundry의 핵심 역량
  * 상태 저장형 + 멀티모달 + 다중 에이전트 지원
  * **AI 모델 주변 시스템**(예: 평가, 오케스트레이션, RAG)의 중요성을 기본으로 내장
  * 전 세계 70,000개 이상 조직이 사용 중
  * 지난 3개월 간 **100조 개 이상의 토큰 처리** (전년 대비 5배 증가)
  * 대표 사례: **청각 처리 장애(APD)** 환자를 위한 Aishin/Katosan 앱

![그림 2 - Foundry Local](/../images/2025-05/build25-02-02.jpg)

* **Foundry Local**
  * Microsoft의 **AI Foundry 플랫폼의 온프레미스(on-premise) 또는 고객 전용 환경(local/private environment)** 배포 버전
  * RAG 기반 애플리케이션 개발, 에이전트 기반 워크플로우 구성, 다양한 오픈 및 커스텀 모델 통합, 보안·프라이버시가 중요한 환경에서 엔터프라이즈 LLM 활용을 지원하는 통합 GenAI 플랫폼
  * **Azure가 아닌**, 고객의 **자체 데이터센터(local)** 또는 **프라이빗 클라우드 환경**에 설치/운영할 수 있게 한 버전
  * 데이터 주권, 보안, 규제 이슈 대응을 위한 **로컬화 배포 옵션**.  air-gapped 환경 또는 government cloud 
  * WIndows 와 mac 운영체제 둘 다 지원

![그림 3 - Windows AI Foundry](/../images/2025-05/build25-02-03.jpg)

* Windows AI Foundry 

  * 이번 Build 행사에서 발표한 새로운 **로컬 GenAI 개발/배포 플랫폼**으로, **Windows PC에서 실행되는 AI Foundry의 로컬 경량 버전**

  | 항목          | 설명                                                         |
  | ------------- | ------------------------------------------------------------ |
  | **목적**      | Windows PC 환경에서 RAG 기반 앱 개발, LLM 실행, 커스터마이징 지원 |
  | **위치**      | 클라우드(X) → 로컬(Windows PC 내부)                          |
  | **기반 기술** | Microsoft AI Foundry 기술 (GraphRAG, Prompt Flow 등)         |
  | **모델**      | Phi-3, LLaMA, Mistral 등 소형 또는 미드사이즈 LLM 지원       |
  | **출시 시점** | 2025년 하반기 (개발자 프리뷰 예정)                           |

  * **로컬 RAG 엔진:** PC 내부에 저장된 문서를 인덱싱하고 검색, OpenAI Embedding API 또는 로컬 embedding 모델 사용 가능

  * **경량 LLM 실행 환경:** ONNX Runtime, DirectML 등을 활용해 GPU 없이도 inference 가능, Phi-3, Mistral 등 경량화된 모델을 로컬 실행

  * **Prompt Flow 연동:** Azure AI Studio에서 쓰던 Prompt Flow 기능을 로컬에서도 실행. 워크플로우 기반 LLM 파이프라인 작성

  * **GraphRAG:** 문서 간 관계 기반 검색 및 응답 향상. 일반적인 vector RAG보다 더 깊이 있는 문맥 연결 지원

  * **Copilot Extensions 개발 환경:** Windows Copilot과 연동 가능한 플러그인 또는 기능 확장 앱 개발 가능. ex) 개인화된 로컬 도우미 제작 가능

* MCP Registry for Windows - developer preview 발표

![그림 4 - MCP Registry for Windows](/../images/2025-05/build25-02-04.jpg)

* AI 모델 선택과 유연한 활용

  * **모델 카테고리**: 1,900개 이상 지원 모델 지원
  * **OpenAI와 공동 출시**: 15개 OpenAI 모델을 Azure에서 동시 출시, **Sora** 모델도 곧 출시 예정
  * **새로운 모델 라우터**: 작업에 가장 적합한 모델을 자동 선택
  * **통합 처리량 프로비저닝**: 한 번 처리량을 설정하면 Grok, Mistral, Llama, Black Forest Labs 등 다양한 모델에 사용 가능
    - Mistral: **EU 주권 클라우드 배포** 가능
    - Llama: 전체 “라마 무리”를 Azure로 이식 중
  * **Hugging Face 파트너십**: 11,000개 이상의 오픈소스/프론티어 모델에 Foundry에서 접근 가능

  ![그림 7 - Grok3 on Azure](/../images/2025-05/build25-02-07.jpg)

  * **xAI의 Grok 3**: 추론, 검색, 응답 통합 기능을 갖춘 모델로 Azure 출시 예정
     → 일론 머스크는 Grok 3.5가 “제1원칙 추론 기반”의 고신뢰 AI임을 강조

* 현대적 지식 검색 및 RAG

  * 단순 벡터 검색을 넘어선 **에이전트 최적화 지식 검색 스택**
  * 사용자 맞춤형 검색 기반 RAG 구현 용이

* Foundry 에이전트 서비스 (GA)

  * **선언형 에이전트 생성**, **다중 에이전트 오케스트레이션 지원**
  * **Semantic Kernel** 및 **AutoGen**과 통합
  * 이미 **1만 개 이상의 조직이 사용 중**

* 다양한 컴퓨팅 옵션

  * **서버리스**: Azure Functions, Azure Container Apps
  * **하이브리드**: Azure Kubernetes Service(AKS) 및 Arc 기반 클러스터 → 워크로드 특성에 따라 **유연한 성능·비용 최적화**

* Copilot Studio와의 연계

  * Foundry에서 학습/미세 조정한 모델을 **Copilot Studio로 가져와**→ 워크플로우 자동화 및 맞춤형 에이전트 구축 가능

* 실제 활용 예시

  * **Stanford Medicine**: 헬스케어 에이전트 오케스트레이터를 Foundry에서 운영
  * **Vibe Travel 앱 (Kadesha 데모)**:
    - Foundry 에이전트를 통한 여행 일정 추천
    - Bing 기반 검색 기능으로 근거 제시
    - GitHub Copilot의 에이전트 모드로 이슈 해결
    - GitHub MCP 서버 연결, 시각 입력 기반 스케치 이해

* 보안 및 거버넌스 기능 강화

  * **Entra ID 통합**:
    * 에이전트에 자체 ID, 권한, 정책 설정 가능
    * ServiceNow 및 Workday와 연계한 **자동 프로비저닝 및 권한 관리**
  * **Purview 통합**: 종단 간 **데이터 보호 및 규정 준수** 보장
  * **Microsoft Defender 통합**: 에이전트를 **지갑 남용, 자격 증명 도용** 등 위협으로부터 보호

* 운영 관찰 가능성 (Observability): 성능, 품질, 안전성, 비용을 **한눈에 모니터링** 가능한 **AI 관찰 기능** 제공 → 프로덕션 환경에서 **투명성과 안정성** 확보

![그림 5 - 차세대 Azure AI Infra](/../images/2025-05/build25-02-05.jpg)

## 2. 최대 성능 및 최소 비용을 진화하는 차세대 Azure AI 인프라 

![그림 6 - 차세대 AI 인프라 핵심 목표](/../images/2025-05/build25-02-06.jpg)

* 차세대 Azure AI 인프라의 핵심 목표는 와트당/달러당 가장 많은 토큰을 생성하는 것 -> 무어의 법칙 뿐만 아니라 **시스템 소프트웨어 최적화**, **모델 최적화**라는 복수의 S-커브를 동시에 활용함

![그림 8 - NVIDIA GB200](/../images/2025-05/build25-02-08.jpg)

* 최첨단 하드웨어: NVIDIA GB200 도입
  * Azure는 NVIDIA GB200을 대규모로 도입한 최초의 클라우드 → GB200 NVLink 72 랙 기반 인프라는 **초당 865,000 토큰** 생성 가능, 이는 **모든 클라우드 플랫폼 중 가장 높은 처리량**
  * NVIDIA CEO 젠슨 황 인터뷰
    * CUDA + 모델 최적화 + Azure 인프라의 결합으로 **Hopper 대비 40배 속도 향상** 달성
    * CUDA는 **범용성과 알고리즘 가속 능력**이 뛰어나며, 트랜스코딩, 이미지 처리, 데이터 처리 등 다양한 워크로드에 사용 가능
  * **세계 최대 규모의 GB200 슈퍼컴퓨터**, **Azure에 구축 예정**
* 글로벌 인프라 확장
  * **Azure는 현재 70개 이상의 리전(데이터센터 권역)**을 운영 중 -> 최근 **3개월간 10개 신규 데이터센터 개설**
  * AI를 위해 설계된 최신 Azure 데이터센터는 **작년까지 전체 Azure 인프라보다 많은 광섬유를 사용**
  * 모든 데이터센터는 **400Tbps 규모의 AI WAN 백본**으로 연결
* 차세대 냉각 기술: Maya 사이드킥
  * Maya 액체 냉각 시스템: GB200 최적화를 위한 설계, 물 소비 없이 작동하는 폐쇄 루프 방식
* 새로운 컴퓨팅 옵션: Arm 기반 Cobalt VM
  * Microsoft Teams, Defender 등 **내부 워크로드에 Cobalt VM 적용**
  * **Adobe, Databricks, Snowflake**도 Cobalt 기반 워크로드 확장 중

* 산업 사례 및 전용 인프라
  * **영국 기상청(Met Office)**→ 날씨·기후 예측을 위한 **세계 최고 수준의 슈퍼컴퓨터를 Azure에 구축**
  * **Azure Local**→ **초저지연 또는 연결 불안정 환경**을 위한 엣지 및 로컬 컴퓨팅 솔루션 제공
* 디지털 주권 및 보안
  * 디지털 복원력 및 주권 통제 기능 제공: 데이터 상주(Location Bound), 기밀 컴퓨팅, AI 가속기 및 GPU, PaaS 워크로드에 대한 **규정 준수·보안 강화**

![그림 9 - Microsoft Discovery](/../images/2025-05/build25-02-09.jpg)

## 3. 과학 연구 플랫폼으로 진화하는 마이크로소프트 디스커버리 

* 마이크로소프트 디스커버리(Microsoft Discovery)는 **AI 플랫폼 전체 스택을 과학 연구와 R&D 워크플로우에 직접 적용**하려는 새로운 전략의 일환 -> 단순한 분석 도구를 넘어, **지능형 연구 수행이 가능한 에이전트 중심 플랫폼**을 지향
* 핵심 기술: 그래프 기반 지식 추론(GraphRAG)

  * **GraphRAG(그래프 기반 RAG)** 기술을 기반으로 작동 → 단순한 사실 검색을 넘어서, **과학적 개념 간의 관계**, **함의**, **미묘한 도메인 지식**까지 이해 가능 → 공개 데이터와 사내 전용 데이터를 함께 활용

![그림 10 - 다양한 사례](/../images/2025-05/build25-02-10.jpg)

* Foundry 기반 R&D 에이전트

  * **Azure AI Foundry** 위에서 동작하며, 추론뿐만 아니라 **직접 연구 수행에 최적화된 에이전트**를 제공
  * 기존의 정적인 파이프라인 대신, 연구 에이전트들이 **후보 물질 생성 → 시뮬레이션 → 결과 학습**의 **지속적·반복적 순환(cyclic workflow)**을 통해 협업

![그림11 - Discovery Demo](/../images/2025-05/build25-02-11.jpg)

* 실제 데모: PFAS-Free 침지 냉각제 발견

  * 데모 시나리오: 유해한 PFAS(과불화화합물)를 사용하지 않는 침지 냉각제 개발

    * 3단계 워크플로우

      * **지식 추론 (Knowledge Inference)**: GraphRAG 기반 지식 그래프를 통해 심층적인 과학 인사이트 도출
      * **가설 생성 (Hypothesis Generation)**: 에이전트가 관련 모델과 데이터를 활용하여 후보 물질 제안
      * **실험 실행 (Experiment Execution)**: 생성 화학(Generative Chemistry), AI 기반 스크리닝, HPC 기반 시뮬레이션 검증

    * **도구 및 모델 연동**:

      * Microsoft 플랫폼, 오픈소스 툴, 제3자 솔루션, 자체 조직의 R&D 툴까지 통합 사용
      * 워크플로우는 에이전트가 자동 구축

    * 결과: 실제로 **유망한 비-PFAS 냉각 후보 물질을 합성**하여 PC에서 **Forza Motorsport를 실행하며 냉각 성능을 실시간으로 시연**

      

      

## 4. 결론 

* Azure AI Foundry는 AI 중심 앱 구축을 위한 **인텔리전스를 위한 생산 라인**으로, 상태 저장형, 멀티모달, 다중 에이전트 기반의 **프로덕션 수준 AI 앱**을 손쉽게 개발하고 운영할 수 있도록 설계했다. 
* **Microsoft의 차세대 AI 인프라 전략은 ‘최고의 처리량을 최적의 비용으로 제공’하는 데 초점**을 두고 있으며, NVIDIA GB200 도입, AI 전용 데이터센터, 글로벌 네트워크, Arm 기반 컴퓨팅, 주권형 보안 인프라까지 **AI 시대를 위한 완전한 스택을 갖춘 플랫폼으로 Azure를 강화**하고 있음
* **Microsoft Discovery는 과학 연구를 위한 AI-first R&D 플랫폼**으로,
   GraphRAG 기반의 심화 지식 추론과 Foundry 기반의 반복적 워크플로우를 통해
   **에이전트가 직접 연구를 주도하고 실행하는 새로운 과학 탐색 방식**을 제시하며, AI는 이제 단순 도우미를 넘어, **실제 과학자와 협력하는 연구 파트너**가 되고 있다. 