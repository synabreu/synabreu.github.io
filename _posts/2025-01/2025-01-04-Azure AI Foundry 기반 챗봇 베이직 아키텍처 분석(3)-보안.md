---
title: "Azure AI Foundry 기반 챗봇 베이직 아키텍처 분석(3)-보안"
date: 2025-01-04
tags: [마이크로소프트, Microsoft, Build 2025, Azure AI Foundry, Azure, Azure AI Foundry SDK, Azure OpenAI Studio, Azure OpenAI Service, Azure Machine Learning, Azure App Service, Azure Key Vault, Azure Monitor]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

보안은 고의적인 공격과 귀중한 데이터 및 시스템의 오용으로부터 보호해 주는 보장을 제공한다.  Azure AI Foundry 서비스에서 챗봇 베이직 아키텍처 디자인에서 고려사항들 중에 아키텍처가 구현하는 주요 보안 권장 사항에 대해 노트에 정리한다. 



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

  | 리소스                            | 역할                     | 범위                     |
  | --------------------------------- | ------------------------ | ------------------------ |
  | App Service                       | Azure AI User            | Azure AI Foundry account |
  | Azure AI Foundry project          | Search Index Data Reader | AI Search                |
  | Portal user (for each individual) | Azure AI Developer       | Azure AI Foundry account |

* 실제 역할은 Azure Portal 또는 CLI에서 프로젝트 환경에 맞게 설정



## 5. 네트워크 보안 (Network Security)

* 엔드 투 엔드 챗봇 솔루션을 쉽게 학습할 수 있도록 하기 위해, Basic 챗봇 아키텍처는 **네트워크 보안(Network Security)** 을 구현하지 않음
  * **ID를 경계(perimeter)** 로 사용하고 **퍼블릭 클라우드 구성요소**를 기반으로 구성함
  * AI Search, Azure AI Foundry, App Service 같은 서비스는 **인터넷에서 직접 접근 가능**하며, 이로 인해 아키텍처의 공격 표면(attack surface)이 넓어짐
*  Basic 챗봇 아키텍처는 **아웃바운드 트래픽(egress traffic)** 을 제한하지 않음
  * 에이전트는 OpenAPI 스펙을 기반으로 어떤 **퍼블릭 엔드포인트**에도 연결되도록 설정할 수 있음
  * **프라이빗 데이터의 외부 유출(data exfiltration)** 은 네트워크 제어만으로는 방지할 수 없음



## 6. 마이크로소프트 디펜더(Microsoft Defender)

* 챗봇 베이직 아키텍처에서는 **Microsoft Defender 클라우드 워크로드 보호 계획**을 어떤 서비스에도 활성화할 필요는 없음

* 프로덕션 환경으로 전환할 경우, **기본 보안 아키텍처의 보안 지침**을 따라야 하며, 여기에는 워크로드를 보호하기 위한 **여러 Defender 계획**이 포함해야 함

  

## 7. Azure Policy를 통한 거버넌스 (Governance through Policy)

* 챗봇 베이직 아키텍처는 **Azure Policy를 통한 거버넌스**를 구현하지 않음
* 프로덕션 환경으로의 이전을 고려할 때, 베이직 아키텍처에서 제안하는 **거버넌스 권장 사항**을 따라야 함
* 권장 사항은 **워크로드 구성 요소 전체에 Azure Policy를 적용**하는 방식



## 8. 참고 사항

* [Basic Azure AI Foundry 챗봇 레퍼런스 아키텍처](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/architecture/basic-azure-ai-foundry-chat)

* [Baseline Azure AI Foundry 챗봇 레퍼런스 아키텍처 - 네트워킹](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/architecture/baseline-azure-ai-foundry-chat#networking)

  