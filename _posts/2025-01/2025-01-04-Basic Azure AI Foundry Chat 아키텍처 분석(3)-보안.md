---
title: "Basic Azure AI Foundry Chat 아키텍처 분석(2)-신뢰성"
date: 2025-01-02
tags: [마이크로소프트, Microsoft, Build 2025, Azure AI Foundry, Azure, Azure AI Foundry SDK, Azure OpenAI Studio, Azure OpenAI Service, Azure Machine Learning, Azure App Service, Azure Key Vault, Azure Monitor]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

보안은 고의적인 공격과 귀중한 데이터 및 시스템의 오용으로부터 보호해 주는 보장을 제공한다.  Basic Azure AI Foundry 서비스에서 챗봇 아키텍처 디자인에서 고려사항들 중에 아키텍처가 구현하는 주요 보안 권장 사항에 대해 노트에 정리한다. 



## 1. 고려사항 - 보안

* 권장 사항에는 콘텐츠 필터링 및 악용 모니터링, ID 및 액세스 관리, 역할 기반 액세스 제어가 포함함. 
* 챗봇 아키텍처는 프로덕션 배포용으로 설계되지 않았기 때문에 네트워크 보안과 같은 미구현된 보안 기능도 포함하여 정리함.



## 2. 콘텐츠 필터링 및 악용 모니터링

* Azure AI Foundry는 분류 모델을 조합하여 사용하는 콘텐츠 필터링 시스템을 포함
* 컨텐츠 필터링 시스템은 입력 프롬프트와 출력 결과에서 **잠재적으로 유해한 콘텐츠**를 탐지하고 차단함
  * 증오 표현, 성적 콘텐츠, 자해 관련 콘텐츠, 폭력, 욕설, 탈옥(jailbreak) 시도 (언어 모델 제한 우회 목적의 콘텐츠)
* 각 범주에 대해 필터링 강도를 **낮음(low)**, **중간(medium)**, **높음(high)**으로 설정함
* 챗봇 레퍼런스 아키텍처에서는 모델을 배포할 때 `DefaultV2` 콘텐츠 필터를 사용. (필요에 따라 설정을 조정할 수 있음)



## 3. ID 및 액세스 관리

* ID 및 액세스 관리는 App Service 기본 아키텍처의 지침을 기반으로 확장함

* **채팅 UI**는 **관리형 ID(managed identity)** 를 사용하여 **Foundry Agent Service**와 통신하는 **Chat UI API 코드**에 대해 인증함.

* **Azure AI Foundry 프로젝트** 역시 자체 **관리형 ID**를 가지고 있으며, 이 ID는 AI Search와 같은 서비스에 연결을 정의하는 데 사용함. (이 연결은 Foundry Agent Service에서 사용할 수 있음) -> 하나의 Azure AI Foundry 계정에는 여러 프로젝트를 포함할 수 있음

* 각 프로젝트는 **자체 시스템 할당 관리형 ID(system-assigned managed identity)** 를 사용해야 함

* 서로 다른 워크로드 구성 요소가 연결된 데이터 소스에 대해 **격리된 액세스**를 필요로 할 경우, 동일한 계정 내에서 별도의 프로젝트를 생성하고 연결을 공유하지 않는 것을 권장

* 격리가 필요하지 않은 경우, 하나의 프로젝트를 사용하는 것이 가능

  

## 4. 역할 기반 액세스 제어(RBAC)

* 시스템 할당 관리형 ID에 대한 **필요한 역할 할당(Role Assignment)** 은 사용자가 직접 설정해야 함

* App Service, Azure AI Foundry 프로젝트 및 포털 사용자에 대해 설정해야 할 역할 할당 표

  | 대상                       | 역할 예시                                     |
  | -------------------------- | --------------------------------------------- |
  | App Service 관리형 ID      | Foundry Agent Service에 액세스할 수 있는 역할 |
  | Foundry 프로젝트 관리형 ID | AI Search 등 리소스에 대한 데이터 조회 권한   |
  | 사용자                     | Foundry Portal을 통한 프로젝트 관리 권한      |

* 실제 역할은 Azure Portal 또는 CLI에서 프로젝트 환경에 맞게 설정



## 5. 참고 사항

* [Design review checklist for Security](https://learn.microsoft.com/en-us/azure/well-architected/security/checklist)
* [Basic Azure AI Foundry chat reference architecture](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/architecture/basic-azure-ai-foundry-chat)