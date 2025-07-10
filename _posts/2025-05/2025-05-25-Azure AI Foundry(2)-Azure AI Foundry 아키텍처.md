---
title: "Azure AI Foundry(2)-Azure AI Foundry 아키텍처"
date: 2025-05-25
tags: [마이크로소프트, Microsoft, Build 2025, Azure AI Foundry, Azure, Azure AI Foundry SDK, Azure OpenAI Studio, Azure AI Studio]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

 Azure AI Foundry 서비스는 2024년 12월, Microsoft Ignite 행사에서 샤티야 회장이 처음 소개했다. 이 노트에서는 Azure AI Foundry 서비스에 대해 좀더 깊게 파고들기 위해 아키텍처에 대해 노트를 한번 정리해 보도록 하겠다. 

![그림1 - Azure AI Foundry 아키텍처](/../images/2025-05/AzureAI-04.png)



## 1. Azure AI Foundry 아키텍처  

* 위의 그림은 왼쪽에는 개발 도구나 개발통합환경(IDE)에 관한 내용이 있고, 가운데에는 모델 카탈로그, 오른쪽에는 관측 가능성(Observability)에 대한 내용으로 나눠진다. 
* Azure AI Foundry: 다양한 AI 모델에 접근할 수 있는 플랫폼
  *  **"다양한 AI 모델"**이란 OpenAI 모델만이 아니라 다른 벤더의 모델들까지 포함
  * AI 모델, 서비스, 그리고 통합 기능을 통해 개발자가 AI 기반 애플리케이션을 구축하고, 커스터마이징하며, 배포함



## 2.  Azure AI Foundry 개발 도구  

* **Copilot Studio**: AI 기반 코파일럿을 구축하기 위한 도구

* **Visual Studio**: Microsoft의 인기 있는 통합 개발 환경(IDE)

* **GitHub**: 이미 많은 개발자들이 사용 중인 협업 및 버전 관리 플랫폼으로, AI 프로젝트에도 활용

* **Azure AI Foundry SDK:** AI 모델 및 서비스와 상호작용할 수 있게 해주는 도구. 동일한 SDK로 다양한 모델에 접근할 수 있어 AI 개발자에게 엄청난 가능성과 기회를 열어 줌

  

## 3.  모델 카탈로그(Model Catalog)  

* **파운데이션 모델(Foundation Models)**: 범용으로 활용 가능한 AI 모델
* **오픈소스 모델(Open Source Models)**: 오픈소스 커뮤니티에서 제공하는 사전 학습된 모델. ex) Meta의 Llama나 DeepSeek 
* **태스크 모델(Task Models)**: 특정 작업을 위해 설계된 특화된 AI 모델
* **산업별 모델(Industry Models)**: 특정 산업에 맞게 최적화된 AI 솔루션



## 4.  코어 AI 서비스  

* **Azure OpenAI 서비스**: 이전에는 독립형 서비스였지만, 이제 Azure AI Foundry에 통합되었음. 이를 통해 OpenAI의 GPT-3, GPT-4, GPT-4o 등 최신 대형 언어 모델에 접근할 수 있음
* **Azure AI Search**: 지능형 검색 및 색인 기능을 제공함. 텍스트 기반 검색 또는 벡터 검색, AI 기반 인덱싱과 같은 AI 기반의 검색 엔진을 애플리케이션에 쉽게 통합할 수 있도록 도와주는 Azure의 검색 서비스. 이전 서비스 명은 ***Azure Cognitive Search.*** 
* **Azure AI Agent Service**: 작업 자동화 및 워크플로우를 위한 AI 에이전트를 생성할 수 있는 기능을 제공. AI 에이전트를 만들고 실행할 수 있도록 지원하는 Azure의 통합 에이전트 플랫폼.
* **AI Content Safety:** 유해하거나 부적절한 콘텐츠를 자동으로 감지하고 필터링하는 Azure의 AI 기반 책임있는 안전성 서비스



## 5. 관측 가능성(Observability)   

* **관측 가능성:** AI 애플리케이션의 투명성, 모니터링, 지속적인 개선을 보장
  * **Evaluations(평가)**: 다양한 AI 모델의 성능을 평가함
  * **Customization(커스터마이징)**: 특정 요구에 맞게 모델을 미세 조정하는 기능
  * **Governance(거버넌스)**: 보안, 컴플라이언스, 책임 있는 AI 사용을 위한 관리 기능
  * **Monitoring(모니터링)**: AI 성능을 추적하고 문제를 식별하는 기능