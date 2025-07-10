---
title: "Azure AI Foundry(1)-OpenAI Studio와 Azure AI Studio 차이점"
date: 2025-05-24
tags: [마이크로소프트, Microsoft, Build 2025, Azure AI Foundry, Azure, Azure AI Foundry SDK, Azure OpenAI Studio, Azure AI Studio]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

오늘은 Azure AI Foundry의 흥미로운 탄생의 비밀(?)을 밝히고자 한다. Azure AI Foundry에 대해 이야기하려면, 그 배경부터 살펴보는 것이 이 서비스를 이해하기 쉽기 때문이다. 



## 1. Azure AI Foundry 탄생의 배경  

* **Azure OpenAI Studio:** Microsoft가 OpenAI와 맺은 협약 이후, Azure 내에서 OpenAI 모델을 사용할 수 있도록 만든 서비스
* **Azure AI Studio:** 이후 Microsoft는 AI Studio도 자체적으로 발전시켜야 한다고 판단했고 OpenAI 의 모델 외, 마이크로소프트 자체 모델인 Phi 모델이나 메타 라마, 딥시크와 같은 모델을 서비스하기 위해 만듦
* **Azure AI Foundry = Azure OpenAI Studio + Azure AI Studio**



## 2. Azure AI Foundry 란?   

* Azure AI Foundry

  * **신뢰할 수 있는 플랫폼**으로, 개발자들이 **안전하고 책임감 있는 방식으로 혁신적인 AI를 구축**
  * Microsoft는 기업들이 AI를 도입할 때 **보안과 신뢰성**을 가장 중요하게 여긴다는 점을 잘 알고 , Foundry는 **Azure의 일부이긴 하지만**, **AI에 특화된 전용 환경** 만듦

* 기존 Azure 서비스(**Azure AI Studio**) - [https://https://portal.azure.com/#home](https://portal.azure.com/#home)

  ![그림1 - Azure AI Studio](/../images/2025-05/AzureAI-01.png)

* **Azure AI Foundry** - [https://ai.azure.com/](https://ai.azure.com/)

​     ![그림2 - Azure AI Foundry](/../images/2025-05/AzureAI-02.png)

* **모델 카탈로그 :** Foundry에서는 두 가지 종류의 모델을 이용할 수 있음

     ![그림3 - 모델 카탈로그](/../images/2025-05/AzureAI-03.png)

  * OpenAI 모델
  * **기타 오픈소스 및 타사 모델들** (예: Meta LLaMA, DeepSeek 등)
  * 특징:  **오픈소스, 태스크 특화형, 산업 특화형 모델**들을 한 곳에서 사용할 수 있음



## 3. 개발 도구 통합   

* **Copilot Studio:** 비즈니스용 생성형 AI 챗봇을 누구나 쉽게 만들 수 있는 Microsoft의 로우코드 플랫폼

* **Visual Studio**: 전문 개발자를 위한 통합 개발 환경(IDE)

* **GitHub:** 전 세계 개발자들이 협업하며 코드를 공유하고 관리하는 플랫폼

* **Azure AI Foundry SDK:** OpenAI 모델을 쓰고 있다가도 손쉽게 LLaMA나 DeepSeek 모델로 전환할 수 있음

  

## 4. 주요 포함 서비스   

* **Azure OpenAI Service** (기존에는 독립적이었지만, 이제는 Foundry의 일부)
* **Azure AI Search:** 텍스트 기반 검색 또는 벡터 검색, AI 기반 인덱싱과 같은 AI 기반의 검색 엔진을 애플리케이션에 쉽게 통합할 수 있도록 도와주는 Azure의 검색 서비스. 이전 서비스 명은 **Azure Cognitive Search.** 
* **Azure AI Agent Service:** AI 에이전트를 만들고 실행할 수 있도록 지원하는 Azure의 통합 에이전트 플랫폼
* **AI Content Safety:** 유해하거나 부적절한 콘텐츠를 자동으로 감지하고 필터링하는 Azure의 AI 기반 안전성 서비스



## 5. 엔터프라이즈 기능   

* Foundry는 기업 고객을 위해 **모델 커스터마이징 및 거버넌스 기능**을 제공함
  * 모델 파인튜닝
  * 성능 평가 및 벤치마킹
  * 보안 및 책임 있는 AI 사용에 대한 컴플라이언스 확인
* Foundry는 단순한 AI 플랫폼이 아니라, **AI 전 주기 관리**가 가능한 **엔터프라이즈용 솔루션**



