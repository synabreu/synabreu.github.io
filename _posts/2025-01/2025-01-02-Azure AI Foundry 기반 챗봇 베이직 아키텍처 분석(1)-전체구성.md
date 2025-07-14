---
title: "Azure AI Foundry 기반 챗봇 베이직 아키텍처 분석(1)-전체구성"
date: 2025-01-02
tags: [마이크로소프트, Microsoft, Build 2025, Azure AI Foundry, Azure, Azure AI Foundry SDK, Azure OpenAI Studio, Azure OpenAI Service, Azure Machine Learning, Azure App Service, Azure Key Vault, Azure Monitor]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

Azure AI Foundry와 Azure OpenAI 언어 모델을 활용하여 챗 애플리케이션을 실행하는 방법을 학습할 수 있도록 돕는 기본 아키텍처에 대해 노트를 한번 정리해 보았다. 



## 1. 베이직 Azure AI Foundry Chat 아키텍처 조건

* Azure AI Foundry와 Azure OpenAI 언어 모델을 사용하여 챗 애플리케이션을 실행하는 데 도움이 되는 기본 아키텍처
* 챗봇 아키텍처는 **Azure App Service**에서 실행되는 클라이언트 사용자 인터페이스(UI)를 포함
  * 사용자로부터 들어오는 프롬프트를 처리하고 관련 데이터를 가져오기 위해, UI는 **Azure AI 에이전트**를 통해 워크플로우를 조정하여 다양한 데이터 저장소와 연동됨
* 챗봇 아키텍처는 **단일 리전 내에서 작동하도록 설계**되었음 
  * Azure 생태계 내에서 챗봇 서비스를 안정적으로 운영할 수 있도록 필요한 구성 요소들을 통합하여 보여주며, 보안, 모니터링, 추론, 데이터 접속 등을 포함한 엔드투엔드 챗봇 실행 환경을 제공함



## 2. 워크플로우

* 애플리케이션 사용자는 채팅 기능이 포함된 웹 애플리케이션과 상호작용한다. 사용자는 `azurewebsites.net` 도메인의 App Service 기본 도메인으로 HTTPS 요청을 보낸다. 이 도메인은 자동으로 App Service의 기본 제공 공용 IP 주소를 가리킨다. TLS(Transport Layer Security) 연결은 클라이언트에서 App Service로 직접 설정되며, Azure가 인증서를 완전히 관리한다.
* App Service의 **Easy Auth** 기능은 사용자가 웹사이트에 접근할 때 **Microsoft Entra ID**를 통해 인증되도록 보장한다. 
* App Service에 배포된 애플리케이션 코드는 요청을 처리하고 사용자에게 채팅 UI를 렌더링한다. 이 채팅 UI 코드는 동일한 App Service 인스턴스에 호스팅된 API들과 연결한다. 이 API 코드는 **Azure AI Persistent Agents SDK**를 사용하여 Azure AI Foundry에 있는 Azure AI 에이전트와 연결한다.
* **Azure AI Foundry Agent Service**는 **Azure AI Search**에 연결하여 쿼리에 필요한 **그라운딩 데이터(grounding data)**를 가져온다. 이 그라운딩 데이터는 다음 단계에서 Azure OpenAI 모델로 전송되는 프롬프트를 포함한다. 
* Foundry Agent Service는 **Azure AI Foundry**에 배포된 **Azure OpenAI 모델**과 연결되어, 그라운딩 데이터와 대화 컨텍스트가 포함된 프롬프트를 모델에 전송한다. 
* **Application Insights**는 App Service에 대한 원래 요청과 에이전트 간 상호작용에 대한 정보를 로그로 기록한다. 



## 3. 베이직 챗봇 컴포넌트

* 기본 사항
  * Basic App Service 웹 애플리케이션 아키텍처와 동일함. (채팅 UI 기반)
* **Azure AI Foundry**
  * **Azure AI Foundry**는 AI 솔루션과 모델을 서비스 형태(MaaS, Model as a Service)로 구축, 테스트, 배포하는 데 사용하는 플랫폼
  * Azure AI Foundry를 사용하여 **Azure OpenAI 모델을 배포**
  * Azure AI Foundry 프로젝트는 데이터 소스에 대한 연결을 설정하고, 에이전트를 정의하며, Azure OpenAI 모델을 포함한 배포된 모델을 호출함
  * **하나의 Azure AI Foundry 프로젝트만** 포함되어 있으며, 이는 단일 Azure AI Foundry 계정 내에 존재함
* **Foundry Agent Service**
  * **Foundry Agent Service**는 Azure AI Foundry 내에서 호스팅되는 기능
  * 개발자는 Foundry Agent Service를 사용하여 **채팅 요청을 처리하는 에이전트를 정의하고 호스팅**할 수 있음. 
  * **Foundry Agent Service 수행**
    * **채팅 스레드 관리:** 챗봇 또는 AI 에이전트가 사용자와의 대화 이력을 구조화된 방식으로 대화 흐름 추적 (Conversation History Tracking), 메시지 단위 분리 및 저장 (Turn-based Message Storage), 대화 ID 및 스레드 단위 그룹화 (Thread Identification) 및 장기 메모리 지원 또는 컨텍스트 리셋 등을 유지하는 기능.
    * **툴 호출 오케스트레이션:** 언어 모델 기반 에이전트(예: Azure AI Foundry의 Foundry Agent Service)가 사용자 질문을 처리하는 과정에서 **외부 기능(툴)을 적절한 순서와 방식으로 호출하고 응답을 통합**하는 것
    * 콘텐츠 안전성 적용
    * ID, 네트워킹, 관측 시스템과의 통합
  * 워크플로우 오케스트레이션
    * AI Search 인스턴스로부터 그라운딩 데이터를 가져오고, 해당 데이터를 프롬프트와 함께 배포된 Azure OpenAI 모델에 전달
    * Foundry Agent Service에서 정의된 에이전트는 **코드 작성 없이 정의 가능하며, 비결정적(non-deterministic)**
    * **시스템 프롬프트**, **temperature**, **top_p** 파라미터의 조합을 통해 에이전트의 응답 동작이 결정됨
* **Azure AI Foundry 모델 배포**
  * Azure AI Foundry 모델은 Microsoft가 호스팅하는 환경에서 Azure AI 카탈로그에 포함된 **대표 모델(예: OpenAI 모델)**을 배포할 수 있도록 함
  * Azure AI Foundry 모델 배포는 **MaaS(Model as a Service) 배포 방식**으로 간주됨
  * 챗봇 아키텍처는 **고정 할당량을 갖는 Global Standard 구성**을 사용하여 모델을 배포
* **AI Search**
  * **AI Search**는 전체 텍스트 검색, 시맨틱 검색, 벡터 검색, 하이브리드 검색을 지원하는 **클라우드 검색 서비스**
  * 챗봇 아키텍처에는 **AI Search**가 포함되어 있으며, 이는 채팅 애플리케이션에서 흔히 사용되는 오케스트레이션 컴포넌트 구조를 가지고 있음.
  * **AI Search는 애플리케이션 사용자의 쿼리에 적합한 인덱싱된 데이터를 검색**하는 데 사용할 수 있음
  * AI Search는 **RAG(Retrieval-Augmented Generation)** 패턴에서 **지식 저장소(Knowledge Store)** 역할을 함
  * RAG 패턴은 프롬프트로부터 적절한 쿼리를 추출하고, AI Search를 질의하여 그 결과를 **Azure OpenAI 모델의 그라운딩 데이터**로 활용함



## 5. 참고 레퍼런스

* [기본 웹 애플리케이션 아키텍처 디자인](https://learn.microsoft.com/en-us/azure/architecture/web-apps/app-service/architectures/basic-web-app)

* [Azure Well-Architected Framework](https://learn.microsoft.com/en-us/azure/well-architected/)

* [Baseline Azure AI Foundry chat reference architecture](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/architecture/baseline-azure-ai-foundry-chat)

  