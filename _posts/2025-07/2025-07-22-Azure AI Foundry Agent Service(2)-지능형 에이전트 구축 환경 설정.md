---
title: "Azure AI Foundry Agent Service(2)-지능형 에이전트 구축 환경 설정"
date: 2025-07-22
tags: [마이크로소프트, Microsoft, Build 2025, Azure AI Foundry, Azure, Azure AI Foundry SDK, Azure OpenAI Studio, Azure OpenAI Service, Azure Machine Learning, Azure App Service, Azure Key Vault, Azure Monitor, Agentic DevOps, Github Copilot, DevOps, MLOps, Software Factory]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

Azure AI Foundry Agent Service로 첫 번째 에이전트를 생성하는 과정은 에이전트 환경 설정과 여러분이 선택하는 언어별 SDK 또는 Azure Foundry 포털을 사용하여 에이전트를 생성하고 구성 등 두 단계로 이루어진다. 이번 노트에서는 Azure AI Foundry Agent Service에서 지능형 에이전트 구축 환경 설정하는 방법에 대해 정리해보겠다. 



## 1. 권한 필수 사항

| 액션                                                         | 권한 요구                               |
| ------------------------------------------------------------ | --------------------------------------- |
| 계정 및 프로젝트 생성                                        | Azure AI Account Owner                  |
| [Standard setup](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/environment-setup#choose-your-setup) 전용: 필요한 리소스(Cosmos DB, Search, Storage 등)에 대해 RBAC(역할 기반 액세스 제어) 권한 할당 | Role Based Access Control Administrator |
| 에이전트 생성 및 편집                                        | Azure AI User                           |



## 2. 에이전트 환경 설정하기

* 에이전트 환경 설정을 시작하려면 Azure AI Foundry 리소스와 Foundry 프로젝트가 필요하다.
* 에이전트는 특정 프로젝트 내에서 생성되며, 각 프로젝트는 독립된 작업 공간 역할을 한다.
  * 동일한 프로젝트 내의 모든 에이전트는 동일한 파일 저장소, 스레드 저장소(대화 기록), 검색 인덱스에 접근할 수 있다.
  * 프로젝트 간에는 데이터가 격리되어 있으며, 한 프로젝트의 에이전트는 다른 프로젝트의 리소스에 접근할 수 없다.
  * 현재 Foundry에서는 프로젝트 단위로 공유 및 격리를 관리한다. 



## 3. 사전 요구 사항

* **Azure 구독**이 필수 - 구독하지 않았다면, 무료로 가능
* **계정 및 프로젝트를 생성하는 사용자**는 구독 범위에서 **Azure AI Account Owner** 역할을 가지고 있어야 함
* **표준 설정(Standard setup)**을 구성하는 경우, 해당 사용자는 Cosmos DB, Azure AI Search, Azure Blob Storage 리소스 등에 역할을 가진다.
* Azure AI Foundry Agent Service에 특화된 RBAC 역할에 대한 자세한 내용은 Azure AI Foundry Agent Service RBAC 역할 문서를 참고
  * 필요한 기본 역할은 **Role Based Access Administrator**임
  * 또는, 구독 수준에서 **Owner** 역할을 가지고 있는 경우에도 위 조건을 충족함
  * `Microsoft.Authorization/roleAssignments/write` 권한 필수



## 4. 설정 방식 선택하기

* 기본 설정 (Basic Setup)
  *  **OpenAI Assistants**와 호환되며, 플랫폼에 내장된 스토리지를 사용해 에이전트 상태를 관리함
  * OpenAI Assistants API와 동일한 도구 및 기능을 제공하며, 여기에 **비-OpenAI 모델** 및 **Azure AI Search**, **Bing** 등의 도구도 추가로 지원함
* 표준 설정 (Standard Setup)
  * 기본 설정의 모든 기능을 포함하며, 사용자가 자신의 **Azure 리소스**를 사용할 수 있어 데이터에 대한 세밀한 제어가 가능함
  * 파일, 대화 스레드, 벡터 스토어 등 모든 고객 데이터는 사용자의 Azure 리소스에 저장되어, **완전한 소유권과 제어권**을 제공함
* 가상 네트워크를 사용하는 표준 설정
  * Standard Setup with Bring Your Own Virtual Network
  * 표준 설정의 모든 기능을 포함하며, **사용자 소유의 가상 네트워크(BYO Virtual Network)** 내에서 완전히 작동할 수 있는 기능이 추가됨
  * 가상 네트워크를 사용하는 표준 설정은 데이터 이동에 대해 **엄격한 제어**가 가능하며, 네트워크 외부로의 데이터 유출을 방지하기 위해 모든 트래픽을 사용자의 네트워크 환경 내에 유지할 수 있음

| 사용 사례                                                    | 기본 설정 | 공용 네트워킹을 사용하는 표준 설정 | 사설 네트워킹을 사용하는 표준 설정 |
| ------------------------------------------------------------ | --------- | ---------------------------------- | ---------------------------------- |
| 리소스를 직접 관리하지 않고 빠르게 시작하기                  | ✅         |                                    |                                    |
| 모든 대화 기록, 파일, 벡터 스토어가 사용자의 리소스에 저장됨 |           | ✅                                  | ✅                                  |
| 고객 관리 키(CMK) 지원                                       |           | ✅                                  | ✅                                  |
| 사설 네트워크 격리 (사용자 소유 가상 네트워크 사용)          |           |                                    | ✅                                  |



## 5. 배포 옵션

* 이 템플릿을 사용자 지정하려면 [**자신의 리소스 사용** 문서를](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/how-to/use-your-own-resources) 참고하기 바람.
* **사설 네트워크 격리**를 지원하려면, [**네트워크 보안 설정(network-secured setup)** 문서를](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/how-to/virtual-networks) 참고하여 사용자 소유의 가상 네트워크를 설정하는 방법에 대해 자세히 알아보기 바람.

| 자동 배포 및 설명                                            | 다이어그램                                                   |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| <u>* 관리형 ID(Managed Identity를 사용해 인증하는 기본 에이전트 설정을 배포함</u><br />계정과 프로젝트가 생성됨<br />GPT-4o 모델이 배포됨<br />기본적으로 Microsoft에서 관리하는 Key Vault가 사용됩<br /> | ![그림1 - Basic agent setup](/../images/2025-07/AIAgent-04.png) |
| * 관리형 ID(Managed Identity))를 사용하여 인증하는 표준 에이전트 설정을 배포함<br />계정과 프로젝트가 생성<br />GPT-4o 모델이 배포됨<br />고객 데이터를 저장하기 위한 Azure 리소스(Azure Storage, Azure Cosmos DB, Azure AI Search)는 기존 리소스를 제공하지 않는 경우 자동으로 생성됨<br />이러한 리소스는 파일, 대화 스레드, 벡터 데이터를 저장하기 위해 프로젝트에 연결됩<br />기본적으로 Microsoft에서 관리하는 Key Vault가 사용됨<br /> | ![그림2 - Standard agent setup](/../images/2025-07/AIAgent-05.png) |

* [선택 사항] 자동 배포 템플릿에서 모델 선택
  * modelFormat 매개변수는 변경하지 마라.
  * 자동 배포(autodeploy) 템플릿은 **Azure OpenAI 모델**의 배포만 지원합니다. 
  * 지원되는 Azure OpenAI 모델 목록은 [**모델 지원 문서**를](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/concepts/model-region-support?tabs=global-standard) 참고하라.
  * 자동 배포 템플릿에서 모델 매개변수를 편집하여 에이전트가 사용할 모델을 사용자 지정할 수 있음
  * 만일 다른 모델을 배포하려면 최소한 `modelName`과 `modelVersion` 매개변수를 업데이트해야 함
  * 배포 템플릿 구성

​		

| Model Parameter | 기본값                    |
| --------------- | ------------------------- |
| modelName       | gpt-4o                    |
| modelFormat     | OpenAI (for Azure OpenAI) |
| modelVersion    | 2024-11-20                |
| modelSkuName    | GlobalStandard            |
| modelLocation   | eastus                    |



## 6. 참고

* [[Azure AI Foundry(4)-Azure AI Foundry 소개](https://synabreu.github.io/microsoft/azure/Azure-AI-Foundry(4)-Azure-AI-Foundry-소개/)](https://synabreu.github.io/microsoft/azure/Azure-AI-Foundry(4)-Azure-AI-Foundry-%EC%86%8C%EA%B0%9C/)

* [Choose your setup](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/environment-setup#choose-your-setup)

* [Use your own resources](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/how-to/use-your-own-resources)

* [Create a new network-secured environment with user-managed identity](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/how-to/virtual-networks)

* [Models supported by Azure AI Foundry Agent Service](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/concepts/model-region-support?tabs=global-standard)

  