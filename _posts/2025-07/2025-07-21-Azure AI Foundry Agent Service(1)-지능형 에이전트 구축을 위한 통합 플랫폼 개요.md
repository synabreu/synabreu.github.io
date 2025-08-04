---
title: "Azure AI Foundry Agent Service(1)-지능형 에이전트 구축을 위한 통합 플랫폼 개요"
date: 2025-07-21
tags: [마이크로소프트, Microsoft, Build 2025, Azure AI Foundry, Azure, Azure AI Foundry SDK, Azure OpenAI Studio, Azure OpenAI Service, Azure Machine Learning, Azure App Service, Azure Key Vault, Azure Monitor, Agentic DevOps, Github Copilot, DevOps, MLOps, Software Factory]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

대부분의 기업은 단순한 챗봇이 아닌 더 빠르고 오류가 적은 자동화를 원한다. 문서 요약, 인보이스 처리, 고객 지원 티켓 관리, 블로그 게시물 발행 등 다양한 업무에서 이러한 자동화가 활용된다. 궁극적인 목표는 반복적이고 예측 가능한 작업을 자동화함으로써 사람과 자원을 더 가치 있는 일에 집중할 수 있도록 하는 것이다.

초대형 언어 모델(LLM)은 비정형 데이터를 이해하고, 결정을 내리며, 콘텐츠를 생성할 수 있는 새로운 자동화 방식의 가능성을 열었다. 하지만 기업이 데모 수준을 넘어서 실제 운영 환경에 적용하기까지는 여러 어려움이 있다. LLM은 시간이 지나며 성능이 달라질 수 있고, 오류를 범하거나 책임성이 부족할 수 있다. 가시성, 정책 준수, 오케스트레이션이 뒷받침되지 않으면 이러한 모델을 실제 비즈니스에 신뢰하고 적용하기 어렵다. 이번 노트는 애저 AI 파운드리 에이전트 서비스에 대해 간단히 정리해 보도록 하겠다. 



## 1. Azure AI Foundry 생태계

![그림1 - Azure AI Foundry Azure Service](/../images/2025-07/agent-service-the-glue.png)

* Azure AI Foundry 플랫폼은 모델, 도구, 프레임워크, 거버넌스를 통합해 지능형 에이전트를 구축할 수 있도록 지원하는 하나의 시스템
* Azure AI Foundry 시스템의 중심에는 Azure AI Foundry 에이전트 서비스(Agent Service)가 있으며, 개발과 배포, 운영 전반에서 에이전트를 실행할 수 있게 한다.
* AI Foundry 에이전트 서비스는 Azure AI Foundry의 핵심 구성 요소인 모델, 도구, 프레임워크를 단일 런타임 환경으로 연결해 준다.
* 에이전트 서비스는 쓰레드를 관리하고, 도구 호출을 오케스트레이션하며, 콘텐츠 안전성을 강화하고, ID 관리 및 네트워크, 관측 시스템과의 통합을 제공하여, 에이전트가 안전하고 확장 가능하며 즉시 운영 환경에 적용될 수 있도록 지원한다.
* 인프라 복잡성을 추상화하고 신뢰성과 안전성을 설계 단계부터 적용함으로써, AI Foundry 에이전트 서비스는 프로토타입에서 운영 환경으로 자신 있게 전환함



## 2. AI Agent 정의

* 에이전트 정의
  * 에이전트는 의사결정을 내리고, 도구를 호출하며, 워크플로우에 참여하고, 때로는 독립적으로, 때로는 다른 에이전트나 사람과 협력하기도 한다.
  * 에이전트가 일반적인 어시스턴트와 다른 점은 자율성이다.
  * 어시스턴트가 사람을 지원하는 반면, 에이전트는 목표를 직접 달성한다. 따라서 에이전트는 실질적인 프로세스 자동화의 기반이 된다.
  * AI Foundry를 통해 만들어진 에이전트는 하나의 거대한 구조체가 아니라, 조합 가능한(composable) 유닛이다. 
  * 각 유닛은 특정 역할을 가지며, 적절한 모델로 구동되고, 필요한 도구를 갖추며, 안전하고 관찰 가능하며 관리 가능한(runtime) 환경에서 운영된다.
* 에이전트 핵심 구성 요소
  * **모델(LLM)**: 추론과 언어 이해를 담당한다.
  * **지침(Instructions)**: 에이전트의 목표, 행동 방식, 제약조건을 정의한다.
  * **도구(Tools)**: 에이전트가 지식을 검색하거나 실제 행동을 취할 수 있도록 지원한다.

![그림2 - AI Agent](/../images/2025-07/AIAgent-02.png)

* 에이전트 동작 원리
  * 에이전트는 사용자 프롬프트, 알림, 다른 에이전트로부터 온 메시지와 같은 비정형 입력을 받아들인다. 
  * 그리고 도구 실행 결과나 메시지 형태의 출력을 생성한다. 
  * 이 과정에서 정보를 검색하거나 특정 행동을 수행하기 위해 도구를 호출할 수도 있다.



## 3. Azure AI Foundry의 에이전트는 어떻게 동작할까?

* Azure AI Foundry를 지능형 에이전트를 위한 하나의 조립 라인

  * 최신 공장처럼 다양한 전문 작업장들이 있고, 각각은 최종 제품의 일부를 만드는 역할을 한다.
  * 기계나 컨베이어 벨트 대신, 이 '에이전트 공장(Agent Factory)'은 모델, 도구, 정책, 오케스트레이션을 활용해 안전하고, 테스트 가능하며, 실제 운영 환경에 즉시 적용할 수 있는 에이전트를 만든다.

* Azure AI Foundry AI Agent 동작 6 단계

  ![그림3 - AI Agent 6 Step](/../images/2025-07/AIAgent-03.png)

  * **모델(Models)**
    * 조립 라인은 에이전트에게 지능을 부여할 모델을 선택하는 단계부터 시작한다. 
    * GPT-4o, GPT-4, GPT-3.5(Azure OpenAI)를 비롯해 Llama와 같은 다양한 대형 언어 모델(LLM) 중에서 선택할 수 있으며, 이러한 모델은 계속해서 추가되고 있다. 
    * 선택된 모델은 에이전트의 핵심적인 추론 능력을 제공하여, 의사 결정의 중심이 된다.
  * **커스터마이징(Customization)**
    * 선택한 모델을 원하는 용도에 맞게 조정한다.
    * 사용자 용도 조정을 위해 파인튜닝(fine-tuning), 증류(distillation), 도메인별 프롬프트(domain-specific prompts) 등의 방식을 사용해 에이전트를 커스터마이징할 수 있다.
    * 이 단계에서는 실제 스레드(thread) 콘텐츠와 도구 결과에서 확보한 데이터를 기반으로 에이전트의 행동 방식, 특정 역할에 필요한 지식, 과거 수행 결과의 패턴을 모델에 반영하게 된다.
  * **AI 도구(AI Tools)**
    * 에이전트에 도구를 장착한다. 이를 통해 에이전트는 기업 내 지식(Bing, SharePoint, Azure AI Search 등)에 접근하거나, 실제 작업을 수행(Logic Apps, Azure Functions, OpenAPI 등 활용)할 수 있다.
    *  AI 도구 과정에서 에이전트의 능력과 범위가 확장된다. 
  * **오케스트레이션(Orchestration)**
    * 에이전트에는 조정(Orchestration) 과정이 필요하다. 
    * 연결된 에이전트는 도구 호출 처리, 스레드 상태 업데이트, 재시도(retry) 관리, 출력 결과 로깅과 같은 전체 작업 주기를 조정하여 원활하게 동작하도록 지원한다.
  * **가시성(Observability)**
    * 에이전트를 테스트하고 모니터링한다.
    * AI Foundry는 각 단계에서 로그(log), 추적(trace), 평가(evaluation) 데이터를 수집한다. 
    * 전체 스레드(thread) 단위로 완벽한 가시성을 제공하며, Application Insights와의 통합을 통해 팀은 에이전트의 모든 의사결정을 면밀히 검토하고, 지속적으로 성능을 개선할 수 있다.
  * **신뢰성**
    * 에이전트가 주어진 작업에 적합하고 신뢰할 수 있도록 보장하는 것이 중요하다.
    * AI Foundry는 Microsoft Entra를 통한 신원 관리, 역할 기반 접근 제어(RBAC), 콘텐츠 필터, 암호화, 네트워크 격리 등 엔터프라이즈급 신뢰 기능을 제공한다. 
    * 사용자는 플랫폼에서 관리하는 인프라 또는 자체 인프라를 선택하여 에이전트를 실행할 수 있다.

* 결과적으로, 신뢰할 수 있고 확장 가능하며 안전하게 워크플로 전반에 배포할 수 있는 운영 환경에 적합한 에이전트가 완성된다.



## 4. Azure AI Foundry Agent Service 사용 이유

* Azure AI Foundry 에이전트 서비스는 엔터프라이즈 환경에서 지능형 에이전트를 운영 환경에 배포할 수 있도록 지원하는 프로덕션 준비 완료 기반을 제공한다. 

  | 기능                    | **Azure AI Foundry Agent Service**                           |
  | :---------------------- | ------------------------------------------------------------ |
  | 1. 대화에 대한 가시성   | 사용자↔에이전트, 에이전트↔에이전트 간 메시지를 포함한 구조화된 [스레드(thread)](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/concepts/threads-runs-messages#threads)에 완전 접근 가능. UI 구성, 디버깅, 학습에 이상적임. |
  | 2. 멀티 에이전트 조정   | 에이전트 간 메시지 전달을 위한 내장 지원 기능.               |
  | 3. 도구 조정            | [도구 호출의](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/how-to/tools/overview) 서버 측 실행 및 재시도 기능을 구조화된 로그와 함께 제공. 수동 오케스트레이션이 필요하지 않음. |
  | 4. 신뢰성과 안전성      | 통합된 [콘텐츠 필터는](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/how-to/content-filters) 오용을 방지하고 프롬프트 인젝션 위험(XPIA)을 완화하는 데 도움을 준다. 모든 출력은 정책에 따라 관리됨 |
  | 5. 엔터프라이즈 통합    | 컴플라이언스 요구 사항을 충족하기 위해 자체 [저장소(storage)](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/how-to/use-your-own-resources#use-an-existing-azure-cosmos-db-for-nosql-account-for-thread-storage), [Azure AI Search Index](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/how-to/use-your-own-resources#use-an-existing-azure-ai-search-resource), [가상 네트워크를](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/how-to/virtual-networks) 사용할 수 있음. |
  | 6. 관측 가능성과 디버깅 | 스레드, 도구 호출, 메시지 추적이 [모두 완전히 추적 가능](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/concepts/tracing)하며, 텔레메트리를 위한 [Application Insights와 통합됨](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/how-to/metrics). |
  | 7. 신원 및 정책 제어    | 역할 기반 액세스 제어(RBAC), 감사 로그, 엔터프라이즈 조건부 액세스를 완벽히 지원하며 Microsoft Entra 기반으로 구축됨. |



## 5. Foundry Agent Service 시작 절차

* Foundry Agent Service를 시작하려면, 먼저 Azure 구독 내에 Azure AI Foundry 프로젝트를 생성해야 한다.
* 처음 서비스를 사용하는 경우, [환경 설정](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/environment-setup) 및 [빠른 시작 가이드를](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/quickstart?pivots=ai-foundry-portal) 참고하는 것이 좋다.
* 필요한 리소스를 포함한 프로젝트를 생성할 수 있으며, 프로젝트 생성 후에는 GPT-4o와 같은 호환 가능한 모델을 배포할 수 있다.
* 모델을 배포한 후에는 SDK를 사용하여 서비스에 API 호출도 시작할 수 있다.



## 6. 참고 사항

* [What is Azure AI Foundry Agent Service?](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/overview)

* [Build collaborative, multi-agent systems with Connected Agents](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/how-to/connected-agents?pivots=portal)

* [Threads, Runs, and Messages in Azure AI Foundry Agent Service](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/concepts/threads-runs-messages#threads)

* [What are tools in Azure AI Foundry Agent Service?](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/how-to/tools/overview)

* [Configure content filters](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/how-to/content-filters)

* [Use your own resources](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/how-to/use-your-own-resources#use-an-existing-azure-cosmos-db-for-nosql-account-for-thread-storage)

* [Use an existing Azure Index](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/how-to/use-your-own-resources#use-an-existing-azure-ai-search-resource)

* [Create a new network-secured environment with user-managed identity](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/how-to/virtual-networks)

* [Tracing  agents](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/concepts/tracing)

* [Monitor Azure AI Foundry Agent Service](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/how-to/metrics)

* [Set up your environment](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/environment-setup)

* [Quickstart: Create a new agent](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/quickstart?pivots=ai-foundry-portal)

  
