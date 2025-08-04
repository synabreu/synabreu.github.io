---
title: "Azure AI Foundry(4)-Azure AI Foundry 소개"
date: 2025-05-27
tags: [마이크로소프트, Microsoft, Build 2025, Azure AI Foundry, Azure, Azure AI Foundry SDK, Azure OpenAI Studio, Azure OpenAI Service, Azure Machine Learning, Azure App Service, Azure Key Vault, Azure Monitor]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

Azure AI Foundry를 한마디로 말하자면, 엔터프라이즈 AI 운영, 모델 개발자, 애플리케이션 개발을 위한 통합 Azure 플랫폼 서비스(PaaS)이다. 따라서, 프로덕션 수준의 인프라와 사용자 친화적인 인터페이스를 결합해서 개발자가 인프라 관리보다 애플리케이션 구축에 집중할 수 있도록 해 준다. 이번 노트에서는 Azure AI Foundry 에 대한 개념에 대해 간단히 정리하겠다. 



## 1. 애저 AI 파운드리의 소개  

* 정의
  * 에이전트, 모델, 도구를 단일 관리 그룹으로 통합하며, 추적, 모니터링, 평가, 맞춤형 엔터프라이즈 설정 구성과 같은 기업용 기능을 기본으로 제공한다. 
  * Azure 리소스 공급자 네임스페이스 내에서 통합된 역할 기반 액세스 제어(RBAC), 네트워킹, 정책을 통해 간소화된 관리를 제공한다.
* 개발자 작업 설계
  * 엔터프라이즈급 플랫폼 위에서 생성형 AI 애플리케이션 및 AI 에이전트를 구축
  * 최신 AI 도구와 머신러닝 모델을 활용해 탐색, 개발, 테스트, 배포를 수행하며, 책임 있는 AI 원칙을 기반으로 작업
  * 애플리케이션 개발의 전체 수명 주기에 걸쳐 팀과 협업
  * 일관된 API 계약을 바탕으로 다양한 모델 제공업체와 연동하여 작업
* Azure AI Foundry를 사용하면 다양한 모델, 서비스, 기능을 탐색하고, 자신의 목표에 가장 적합한 AI 애플리케이션을 구축할 수 있다.
* Azure AI Foundry 플랫폼은 개념 증명(POC)을 손쉽게 프로덕션 수준의 애플리케이션으로 확장할 수 있도록 지원하며, 지속적인 모니터링과 개선 기능을 통해 장기적인 성공을 도모할 수 있다.



## 2. Azure AI Foundry 프로젝트에서 작업하기 

* Azure AI Foundry 프로젝트는 대부분의 개발 작업이 이루어지는 공간이다.
* 개발자는 Azure AI Foundry 포털에서 프로젝트를 다루거나, 선호하는 개발 환경에서 SDK를 사용하여 작업할 수 있다.
* Azure AI Foundry 프로젝트는 개발자에게 셀프 서비스 방식의 기능을 제공하여, 아이디어를 탐색하고 프로토타입을 구축할 수 있는 새로운 환경을 독립적으로 생성할 수 있도록 지원한다. 
* 동시에 데이터는 격리된 상태로 관리된다. 프로젝트는 보안성과 협업을 위한 단위로 작동하며, 같은 프로젝트 내의 에이전트들은 파일 저장소, 스레드 저장소(대화 기록), 검색 인덱스를 공유한다. 또한 민감한 데이터에 대한 컴플라이언스 및 제어를 위해 자체 Azure 리소스를 가져와 사용할 수도 있다.



## 3. Azure AI Foundry API and SDKs 

* [Azure AI Foundry API는](https://learn.microsoft.com/en-us/rest/api/aifoundry/) 에이전트 기반 애플리케이션을 구축하기 위해 특별히 설계되었음
* 다양한 모델 제공업체를 넘나들며 일관된 방식으로 작업할 수 있는 계약(Contract)을 제공함
* Azure AI Foundry API는 애플리케이션에 AI 기능을 쉽게 통합할 수 있도록 SDK와 함께 제공함
* [SDK 클라이언트 라이브러리 지원 언어](https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/develop/sdk-overview?pivots=programming-language-csharp)
  * Python
  * C#
  * JavaScript/TypeScript (프리뷰)
  * Java (프리뷰)
* [Visual Studio Code용 Azure AI Foundry 확장 프로그램(Azure AI Foundry for VS Code Extension](https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/develop/get-started-projects-vs-code)을 사용하면 개발 환경 내에서 직접 모델을 탐색하고 에이전트를 개발할 수 있다.



## 4. 프로젝트 유형 

* **허브 기반 프로젝트**와 **Foundry 프로젝트** 두 가지 유형을 지원함. 대부분의 경우에는 **Foundry 프로젝트**를 사용하는 것이 적합하다.
  * **Foundry 프로젝트**: Azure AI Foundry 리소스를 기반으로 구축된다. 이 프로젝트 유형은 간편한 설정이 가능하며, 에이전트와 OpenAI, Mistral, Meta 등 업계 최고의 모델에 접근할 수 있다.
  * **허브 기반 프로젝트**: Azure AI Foundry 허브에서 호스팅된다. 회사의 관리 팀이 허브를 생성해둔 경우, 해당 허브에서 프로젝트를 만들 수 있다. 개인적으로 작업하는 경우에는 프로젝트 생성 시 기본 허브가 자동으로 생성된다.
* 새로운 에이전트 및 모델 중심 기능은 **Foundry 프로젝트**에서만 제공함
* Azure AI Foundry API 및 Azure AI Foundry Agent Service의 정식 버전에 대한 접근
* 허브 기반 프로젝트를 Foundry 프로젝트로 마이그레이션하려면, [허브 기반 프로젝트에서 Foundry 프로젝트로 마이그레이션하기 문서를](https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/migrate-project?tabs=azure-ai-foundry) 참고하라.

| 기능                                                         | Foundry 프로젝트  | hub 기반 프로젝트                      |
| :----------------------------------------------------------- | :---------------- | :------------------------------------- |
| 에이전트 (Agents)                                            | ✅ (GA)            | ✅ (Preview 만 지원)                    |
| 에이전트 및 다양한 모델과 연동하기 위한 AI Foundry API       | ✅ (네이티브 지원) | 연결을 통해 사용 가능                  |
| Azure에서 직접 제공하는 모델 – Azure OpenAI, DeepSeek, xAI 등 | ✅                 | 연결을 통해 사용 가능                  |
| 마켓플레이스를 통해 제공되는 파트너 및 커뮤니티 모델 – Stability, Bria, Cohere 등 | ✅                 | 연결을 통해 사용 가능                  |
| HuggingFace 에 올라져 있는 오픈 소스 모델                    |                   | ✅                                      |
| 평가(Evaluations)                                            | ✅                 | ✅                                      |
| 플레이그라운드 (Playground)                                  | ✅                 | ✅                                      |
| 프롬프트 플로우(Prompt flow)                                 |                   | ✅                                      |
| 컨텐트 이해(Content understanding)                           | ✅                 | ✅                                      |
| 프로젝트 파일 (파일을 직접 업로드하고 실험을 시작하세요)     | ✅                 |                                        |
| 프로젝트 수준에서 파일 및 출력 결과의 격리                   | ✅                 | ✅                                      |
| 필수 Azure 종속 항목                                         | -                 | Azure Storage account, Azure Key Vault |



## 5. 참고 자료 

* [What is Azure AI Foundry?](https://learn.microsoft.com/en-us/azure/ai-foundry/what-is-azure-ai-foundry)

* [Work with the Azure AI Foundry for Visual Studio Code extension (Preview)](https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/develop/get-started-projects-vs-code)

* [Work with Azure AI Foundry Agent Service in Visual Studio Code (Preview)](https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/develop/vs-code-agents)

* [Use VS Code containers in hub based projects (Preview)](https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/develop/vscode)

* [Azure AI Foundry REST API reference](https://learn.microsoft.com/en-us/rest/api/aifoundry/)

* [Azure AI Foundry SDK client libraries](https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/develop/sdk-overview?pivots=programming-language-csharp)

  

  