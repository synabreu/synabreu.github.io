---
title: "Azure AI Foundry Agent Service(3)-[실습] Foundry portal 에서 새 에이전트 생성"
date: 2025-07-23
tags: [마이크로소프트, Microsoft, Build 2025, Azure AI Foundry, Azure, Azure AI Foundry SDK, Azure OpenAI Studio, Azure OpenAI Service, Azure Machine Learning, Azure App Service, Azure Key Vault, Azure Monitor, Agentic DevOps, Github Copilot, DevOps, MLOps, Software Factory]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

Azure AI Foundry Agent Service는 사용자 요구에 맞게 맞춤형 지침을 설정하고, 코드 인터프리터나 사용자 정의 함수와 같은 고급 도구를 활용하여 AI 에이전트를 구성할 수 있도록 지원한다. 이번 실습은 Azure AI Foundry Agent Service의 Foundry portal 에서 빠르게 새 에이전트 생성하는 법을 정리했다.



## 1. 사전 요구 사항

* **Azure 구독** – 무료로 생성 가능.
* 계정 및 프로젝트를 생성하는 사용자는 **구독 범위에서 Azure AI 계정 소유자(Azure AI Account Owner)** 역할을 가져야 하며, 이는 프로젝트 생성을 위한 필수 권한을 부여한다.
* 또는, 구독 수준에서 **기여자(Contributor)** 또는 **Cognitive Services 기여자(Cognitive Services Contributor)** 역할을 보유하고 있어도 프로젝트 생성이 가능하다.
* 프로젝트가 생성된 이후에는, 해당 프로젝트 내에서 에이전트를 생성하는 사용자가 **프로젝트 수준에서 Azure AI 사용자(Azure AI User)** 역할을 가지고 있어야 한다.



## 2. Basic Agent Setup

* [https://learn.microsoft.com/en-us/azure/ai-foundry/agents/environment-setup](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/environment-setup) 웹사이트에서 Basic Agent Setup 으로 자동 배포를 한다.

* [Deploy to Azure] 버튼을 누르면, 아래와 같은 사용자 지정 배포 할 수 있는 설정 화면으로 바로 넘어간다. 참고로 위의 사전 요구 사항을 템플릿으로 자동 생성해 주고, 프로젝트 정보의 값은 다음과 같이 정의 한다.

  ![그림1 - 사용자 지정 배포](/../images/2025-07/BasicAgent-01.png)

  * 구독: `Azure-subscription-synabreu`
    * 리소스 그룹: `rg-synabreu-2461`. 예) 리소스 그룹이 없다면, 새로 만들고, 만약 있다면, 여러분의 리소스 그룹명으로 지정한다.

  * 인스턴스 정보
    * 지역: `(US) East US 2`
    * AI Services Name: `my-foundy`
    * Project Name: `my-project`
    * Project Description: `my project description`

* 사용자 지정 배포 화면에서 [검토+만들기] 탭에서 이전 화면에서 설정한 내역들을 모두 확인 후 [만들기] 버튼을 누르면 된다.

  ![그림2 - 사용자 지정 배포의 검토 화면](/../images/2025-07/BasicAgent-02.png)

* 사용자 지정 배포가 [검토+만들기] 후, 배포가 완료된다. 그런 후, [리소스 그룹으로 이동] 버튼을 클릭한다.

  ![그림3 - 배포 완료](/../images/2025-07/BasicAgent-03.png)

* 그러면 자동적으로 `https://ai.azure.com` 웹사이트의 Azure AI Foundry 으로 넘어간다. 그런 후, `Overview` 메뉴에서`Agents`메뉴를 선택한다. 

  ![그림4 - Azure AI Foundry](/../images/2025-07/BasicAgent-04.png)

* 그런 후, 자동적으로 `Deploy gpt-4o` 다이얼로그 화면이 떠오르고, Deployment name 과 Deployment type등을 확인한 다음 [Deploy] 버튼을 클릭한다. 

  ![그림5 - Deploy gpt-4o](/../images/2025-07/BasicAgent-05.png)

* `gpt-4o` 모델을 배포가 성공적으로 끝났다면, 아래와 같이 Agent Name과 ID, Model, Created,  Description, Tools 를 확인할 수 있다.

  ![](/../images/2025-07/BasicAgent-06.png)

* Create and debug your agents 화면의 오른쪽 부분에 Agent 에 대한 속성에서 값을 추가할 수 있다.

  ![](/../images/2025-07/BasicAgent-07.png)

  * **AgentID:** 에이전트를 고유하게 식별하는 **UUID 또는 문자열 식별자**로 API 호출, 로그 추적, 에이전트 간 연결 등에서 특정 에이전트를 참조할 때 사용함. 예) agent_123456789abcdef

  * **AgentName:** 사용자 또는 개발자가 지정하는 **에이전트의 이름**으로, UI 상에서 에이전트를 식별하거나, 다중 에이전트 환경에서 명확한 역할 구분을 위해 사용함. 예) "Agent167" 또는 "DocumentSummarizer"

  * **Deployment:** 에이전트가 사용할 **LLM 모델의 배포 이름**으로, Azure OpenAI 또는 Azure AI Studio에 등록된 모델 배포 이름을 지정하여 해당 모델을 호출함. 예) "gpt-4o-2024-05-13-preview"

  * **Instruction:** 에이전트의 행동 방식을 정의하는 **시스템 지침(System Prompt)**으로, 에이전트가 어떤 어조로 말해야 하는지, 어떤 정보를 중시해야 하는지, 특정 업무를 어떻게 처리해야 하는지를 규정함. 예) You are a helpful agent that can answer questions about geography.

    ![그림8 - Instruction](/../images/2025-07/BasicAgent-08.png)

  * **Agent Description:** 에이전트의 **목적과 역할에 대한 설명**으로, 관리 콘솔에서 에이전트 목록을 볼 때, 에이전트의 기능을 설명하는 메타정보로 사용함. 예) "Extracts key insights from customer support chat logs."

  * **Knowledge:** 에이전트가 참조하는 **지식 소스(Knowledge Base)**으로, 문서 기반 QA를 위해 Azure AI Search, Blob Storage, Cosmos DB 등 외부 지식 기반과 연결됨. Retrieval-Augmented Generation (RAG)을 구성할 수 있음. 예): `azureservices-legal-kb` (Azure Cognitive Search 인덱스 이름)

  * **Actions:** 에이전트가 수행할 수 있는 **외부 작업 또는 도구 (Tool Use)** 으로, REST API 호출, 코드 실행, 검색 등 LLM이 직접 수행할 수 없는 작업을 도와주는 함수. [Function Calling 또는 Tool 사용 기능]. 예) `"getEmployeeInfo"` → 내부 HR API 호출

  * **Connected agents:** 협업하는 **다른 AI 에이전트들과의 연결**로, 하나의 복합 시나리오에서 여러 에이전트가 각자 역할을 나누어 협업하도록 구성합니다. 예: 다중 에이전트 워크플로우. 예) DataIngestorAgent` ↔ `FinancialSummaryAgent` ↔ `EmailComposerAgent

  * **Model Settings:**  LLM 호출 시 사용할 **세부 하이퍼파라미터 설정**으로, Temperature, Top_p, Max Tokens, Stop Sequences 컴포넌트 등으로  생성 품질, 안정성, 응답 길이 등을 미세하게 조정함.

    * **Temperature**: 출력 다양성 조절 (0 = 결정적, 1 = 창의적)
    * **Top_p**: 확률 질량 기반 토큰 샘플링
    * **Max Tokens**: 응답 길이 제한
    * **Stop Sequences**: 특정 문자열에서 응답 중단

* Agents Playground 에서 간단하게 `What is the capital of South Korea?`질문하면, 답변을 에이전트가 아래와 같이 해준다.

  ![그림9 - Agents Playground](/../images/2025-07/BasicAgent-09.png) 

  

