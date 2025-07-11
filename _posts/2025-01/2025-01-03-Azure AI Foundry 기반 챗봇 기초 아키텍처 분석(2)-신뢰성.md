---
title: "Azure AI Foundry 기반 챗봇 기초 아키텍처 분석(2)-신뢰성"
date: 2025-01-03
tags: [마이크로소프트, Microsoft, Build 2025, Azure AI Foundry, Azure, Azure AI Foundry SDK, Azure OpenAI Studio, Azure OpenAI Service, Azure Machine Learning, Azure App Service, Azure Key Vault, Azure Monitor]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

Basic Azure AI Foundry 서비스에서 챗봇 아키텍처 디자인에서 고려사항들 중에 신뢰성에 대해 좀 더 상세히 알아보자! 



## 1. 신뢰성

* 고려사항은 **Azure Well-Architected Framework**의 핵심 원칙을 구현한 것

*  **Azure Well-Architected Framework**은 워크로드의 품질을 향상시키기 위해 따를 수 있는 가이드라인을 제공함 

* 기본 아키텍처는 **운영 환경용이 아닌 학습용**으로 설계되었음

  * 기능보다는 단순성과 비용 효율성을 중시하여 **Azure AI Foundry와 Azure OpenAI**를 활용한 종단 간 채팅 애플리케이션을 학습하는 데 적합

* **신뢰성 (Reliability)**

  * 신뢰성은 애플리케이션이 고객에게 약속한 서비스 수준을 안정적으로 제공할 수 있도록 보장함 -> [신뢰성을 위한 설계 검토 체크리스트](https://learn.microsoft.com/en-us/azure/well-architected/reliability/checklist)

  * **App Service Basic 요금제 사용**

    * 챗봇 아키텍처는 [**Azure 가용 영역(Availability Zones)**](https://learn.microsoft.com/en-us/azure/reliability/availability-zones-overview?tabs=azure-cli)을 지원하지 않는 App Service Basic 요금제를 사용
    * 인스턴스, 랙 또는 해당 인스턴스를 호스팅하는 데이터 센터에 문제가 생기면 서비스가 중단될 수 있음
    * 운영 환경으로 전환할 경우, App Service 인스턴스를 위한 [신뢰성 가이드](https://learn.microsoft.com/en-us/azure/architecture/web-apps/app-service/architectures/baseline-zone-redundant#app-services) 를 따라야 함

  * **클라이언트 UI의 자동 확장 미사용**

    * 컴퓨팅 리소스가 비효율적으로 할당되면 신뢰성 문제로 이어질 수 있음
    * 최대 동시 사용자 수를 처리할 수 있도록 리소스를 과잉 할당(overprovisioning)하는 것이 필요함

  * **Foundry Agent Service의 종속 리소스에 대한 제어권 없음**

    * Foundry Agent Service는 **Microsoft가 완전하게 호스팅하는 솔루션**으로, Azure Cosmos DB, Azure Storage 계정, AI Search 인덱스를 포함한 리소스들을 사용함
    * 이러한 종속 리소스들은 사용자 구독에 나타나지 않으며, 직접적인 제어가 불가능함
    * 비즈니스 연속성과 재해 복구 전략을 구현하려면 이들 종속 리소스를 사용자 소유로 전환하는 것이 좋음
    * 참고: 컴포넌트의 AI Service 와 위의 아키텍처 다이어그램에서 그린 **AI Search 인스턴스**는 Foundry Agent Service의 종속 리소스로 사용되는 인스턴스와 다릅 -> 컴포넌트 섹션의 인스턴스는 사용자 고유의 **그라운딩 데이터(grounding data)**를 저장하며, 종속 인스턴스는 채팅 세션 중 업로드된 파일을 **실시간으로 청킹(chunking)** 합

  * **Global Standard 배포 유형 사용**

    * 학습 목적의 기본 아키텍처에서는 Global Standard 배포 유형을 사용할 수 있음
    * 운영 환경으로 전환할 때는 **처리량(throughput)** 및 **데이터 거주(data residency)** 요건을 명확히 파악한 후 적절한 배포 유형을 선택해야 함
      * 처리량 요건이 명확해지면 **Data Zone Provisioned(로컬 리전 기반)** 또는 **Global Provisioned** 유형을 고려
      * 데이터 거주 요건이 있다면 **Data Zone Provisioned** 유형을 선택

  * **AI Search Basic 요금제 사용**

    * 이 요금제는 **Azure 가용 영역**을 지원하지 않음

    * **Zone Redundancy**(가용 영역 중복성)를 확보하려면:

      * **AI Search Standard 요금제 이상**을 사용하라

      * 가용 영역을 지원하는 리전에 배포하고 **3개 이상의 복제본(replicas)**을 구성하라

        

## 2.  Data Zone Provisioned 과 Global Provisioned 배포 차이점

* 애플리케이션이 운영 환경에 진입하면, **동시 사용자 수**, **요청 빈도**, **문서 업로드량**, **검색 쿼리 수** 등의 측면에서 **예상 처리량(throughput)**이 점점 더 중요해짐

* Azure AI Foundry 또는 AI Search와 같은 서비스는 이러한 처리량을 기준으로 적절한 성능을 제공할 수 있도록 **프로비저닝(사전 리소스 예약)**된 배포 방식을 제공

* **실제 사용량**이나 **트래픽 수준**을 분석하여,  Data Zone Provisioned 나 Global Provisioned 로 배포시켜야 함

* 데이터가 **특정 리전에 머물러야 하는 경우** 또는 해당 리전 내에서 **예측 가능한 처리량과 성능을 확보**하려면 **Data Zone Provisioned**를 선택

* 데이터가 전 세계 어디에 저장되어도 괜찮고, **글로벌 사용자에게 빠른 응답성과 높은 가용성**을 제공하고 싶다면 **Global Provisioned**를 선택

  

## 3.  Data Zone Provisioned 과 Global Provisioned 배포 옵션 비교

| 항목            | **Data Zone Provisioned**                                    | **Global Provisioned**                                       |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **설명**        | 지정된 **데이터 존(region)**에 리소스를 고정적으로 프로비저닝 | Azure의 여러 글로벌 리전에서 사용자를 위한 최적화된 분산 배포 |
| **사용 사례**   | 데이터 거주 요건이 있는 기업 (예: "데이터는 반드시 한국 내에 저장되어야 한다") | 글로벌 사용자 대상 서비스 (예: 여러 국가에서 접근)           |
| **데이터 거주** | 지원 (리전을 지정할 수 있음)                                 | 일부 제한됨 (Azure가 리전을 최적화해 배포)                   |
| **지연 시간**   | 리전 내 사용자에게 빠른 응답                                 | 글로벌 최적화되어 있지만, 데이터가 특정 지역에 저장되지 않을 수 있음 |
| **처리량 제어** | 고정된 지역에서의 처리량을 정밀하게 조절 가능                | 글로벌 전체 처리량 최적화에 초점                             |