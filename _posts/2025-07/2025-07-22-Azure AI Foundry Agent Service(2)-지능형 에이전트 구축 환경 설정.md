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





## 5. 참고

* [Choose your setup](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/environment-setup#choose-your-setup)

  