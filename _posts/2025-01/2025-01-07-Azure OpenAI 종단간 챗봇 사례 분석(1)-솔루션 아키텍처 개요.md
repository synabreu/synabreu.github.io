---
title: "Azure OpenAI 종단간 챗봇 사례 분석(1)-솔루션 아키텍처 개요"
date: 2025-01-07
tags: [마이크로소프트, Microsoft, Build 2025, Azure AI Foundry, Azure, Azure AI Foundry SDK, Azure OpenAI Studio, Azure OpenAI Service, Azure Machine Learning, Azure App Service, Azure Key Vault, Azure Monitor]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

Azure OpenAI 종단간 챗봇 사례 분석은 챗 애플리케이션과 AI 오케스트레이션 계층을 단일 리전에서 실행하는 접근 방식을 보여준다. 구현에서는 Azure OpenAI 기반 모델과 Azure AI Foundry Agent 서비스를 오케스트레이터로 사용하고, 깃허브에 있는 리포지토리는 Microsoft Learn의 **베이직 종단 간 챗봇 레퍼런스 아키텍처**를 직접 지원한다. 이 노트는 그러한 챗봇 애플리케이션의 기본적인 예제를 정리한 것이다. 

참고로 보다 기업 환경에 적합한 요구 사항을 반영한 구현을 원하신다면, **Azure AI Foundry Agent 서비스 End-to-End 베이스라인 참조 구현**을 참고하기 바란다.  해당 구현은 이 코드에서 지적된 다양한 **운영 환경 대비(production readiness)** 변경 사항들을 반영하고 있다.



## 1. 아키텍처 시나리오

* 시나리오들을 통해 기본적인 에이전트 기반 챗 애플리케이션을 Azure 환경에 배포하고 실행하는 방법을 보여줌
  * 에이전트를 호스팅하기 위한 **Azure AI Foundry 설정**
  * Azure AI Foundry Agent 서비스에 에이전트 배포
  * **Azure Web App**에 호스팅된 .NET 코드에서 에이전트를 호출



## 2. 에이전트를 호스팅하기 위한 Azure AI Foundry 설정

* Azure AI Foundry는 **Azure AI Foundry Agent 서비스**를 하나의 기능(capability)으로 호스팅함

* Foundry Agent 서비스의 REST API는 **인터넷에 노출된 엔드포인트**로 제공

  ![그림1 - Basic Chatbot architecture](/../images/2025-01/basic-chat-reference.png)

* 위의 다이어그램을 번호 순서 별
  * 애플리케이션 사용자는 채팅 기능이 포함된 웹 애플리케이션과 상호작용한다. 사용자는`azurewebsites.net`의 App Service 기본 도메인으로 HTTPS 요청을 보낸다. 이 도메인은 자동으로 App Service의 내장 퍼블릭 IP 주소를 가리키며, 전송 계층 보안(TLS)) 연결은 클라이언트에서 App Service로 직접 설정된다. 인증서는 Azure에서 완전히 관리한다.
  * **Easy Auth**라는 App Service 기능은 웹사이트에 접근하는 사용자가 **Microsoft Entra ID**를 통해 인증되도록 보장함
  * App Service에 배포된 애플리케이션 코드는 사용자의 요청을 처리하고 사용자에게 채팅 UI를 렌더링함
  * 이 채팅 UI는 동일한 App Service 인스턴스에 호스팅된 API와 연결되며, 이 API 코드는 **Azure AI Persistent Agents SDK**를 통해**Azure AI Foundry** 내의 AI 에이전트와 통신함
  * **Azure AI Foundry Agent 서비스**는 쿼리에 대한 **그라운딩 데이터(grounding data)**를 가져오기 위해 **Azure AI Search**와 연결됩니다. 이 그라운딩 데이터는 다음 단계에서 **Azure OpenAI 모델**에 전달될 프롬프트에 포함됨
  * Foundry Agent 서비스는 **Azure AI Foundry**에 배포된 **Azure OpenAI 모델**과 연결되어, 그라운딩 데이터와 대화 문맥이 포함된 프롬프트를 전송함
  * **Application Insights**는 App Service로 들어온 원래 요청과 에이전트 호출(interaction)에 대한 정보를 로그로 기록함



## 3. Azure AI Foundry Agent 서비스에 에이전트 배포

* 에이전트는 Azure AI Foundry 포털, [Azure AI Persistent Agents 클라이언트 라이브러리](https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/ai/Azure.AI.Agents.Persistent), [REST API](https://learn.microsoft.com/en-us/rest/api/aifoundry/aiagents/) 등과 같은 방법들을 통해 생성할 수 있음
* 에이전트의 생성 및 호출은 **데이터 플레인 작업(data plane operation)**임
* 에이전트는 **소스 제어가 가능하고 버전이 관리되는 자산**이어야 함 -> 이렇게 하면 애플리케이션의 다른 코드와 함께 일관된 방식으로 에이전트를 배포할 수 있음
* 배포 가이드에서는 REST API를 사용하여 에이전트를 생성하며, 이는 실제 배포 파이프라인이 에이전트를 생성하는 시나리오를 **시뮬레이션**하는 것



## 4. Azure Web App에 호스팅된 .NET 코드에서 에이전트 호출

* 채팅 UI 애플리케이션은 **Azure App Service**에 배포됨
* 채팅 애플리케이션의 .NET 코드는 **Azure AI Persistent Agents 클라이언트 라이브러리**를 사용하여
  워크로드용 에이전트에 연결함
* 에이전트의 엔드포인트는 **Azure AI Foundry를 통해 인터넷에 노출**되어 있어 웹 애플리케이션에서 외부 호출이 가능함



## 5. 참고 레퍼런스

* [openai-end-to-end-basic](https://github.com/Azure-Samples/openai-end-to-end-basic)

