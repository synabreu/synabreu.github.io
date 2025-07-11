---
title: "Azure AI Foundry 기반 챗봇 기초 아키텍처 분석(4)-비용 최적화"
date: 2025-01-04
tags: [마이크로소프트, Microsoft, Build 2025, Azure AI Foundry, Azure, Azure AI Foundry SDK, Azure OpenAI Studio, Azure OpenAI Service, Azure Machine Learning, Azure App Service, Azure Key Vault, Azure Monitor]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

비용 최적화는 **불필요한 지출을 줄이고 운영 효율성을 개선**하는 방법에 중점을 둔다. Azure AI Foundry 기반 챗봇 기초 아키텍처 분석 중 비용 최적화에 대한 노트를 정리해 보자. 



## 1. 기초(Basic) 챗봇 아키텍처의 한계

* 권장 사항에는 콘텐츠 필터링 및 악용 모니터링, ID 및 액세스 관리, 역할 기반 액세스 제어가 포함함. 
* 챗봇 아키텍처는 프로덕션 배포용으로 설계되지 않았기 때문에 네트워크 보안과 같은 미구현된 보안 기능도 포함하여 정리함.



## 2. Azure OpenAI 모델 호출 비용

* 기초 아키텍처에서는 Azure OpenAI 모델이 **제한된 호출량만 받는다는 전제**하에 설계
* **사전 프로비저닝된 처리량 방식** 대신**종량제(Pay-as-you-go)** 가격이 적용되는 **Global Standard 배포 유형** 사용을 권장
* 프로덕션 환경으로 전환 시에는, **기초 아키텍처의 비용 최적화 지침**을 따라야 함



## 3. Foundry Agent Service 비용

* 채팅 중에 업로드된 **파일은 비용이 발생**함 → 사용자 경험에 필요하지 않다면, **파일 업로드 기능을 비활성화**하라.
* Bing 기반 지식 연결(Grounding with Bing) 등 **추가 연결 기능**도 **별도 요금 체계**가 적용
* Foundry Agent Service는 **노코드(no-code)** 솔루션이기 때문에,
   각 요청이 어떤 도구나 지식 소스를 호출할지 **결정적으로 제어할 수 없음**→ 따라서 **모든 연결의 최대 사용량을 기준**으로 비용 모델링을 해야 함



## 4. App Service 요금제

* 기초 아키텍처는 **App Service의 Basic 요금제(단일 인스턴스)** 를 사용
* **가용 영역 장애(Availability Zone Outage)** 에 대한 보호 기능이 없음
* 기초 App Service 아키텍처에서는 **3개 이상의 워커 인스턴스가 포함된 Premium 요금제** 사용을 권장함 → 이는 고가용성을 위한 조치이며, **비용 증가에 영향을 미침**



## 5. AI Search 요금제

* **AI Search Basic 요금제**를 사용하며 **복제본(replica)이 없음**
* 따라서 이 구성은 **Azure 가용 영역 장애를 견딜 수 없음**
* 기초 종단 간 챗봇 아키텍처에서는 **Standard 이상 요금제**와 **3개 이상의 복제본 배포**를 권장함 → 역시 **프로덕션 전환 시 비용 증가 요인**이 됨



## 6. 비용 거버넌스 미구현

* 기초 아키텍처는 **비용 거버넌스 또는 비용 억제(Control) 기능**을 포함하지 않음

* **Azure AI Foundry의 종량제 모델**처럼 사용량 기반 요금 구조에서는,**관리되지 않은 프로세스나 과도한 사용**이 **예상치 못한 고비용**을 유발함

  

## 7. 기본 아키텍처와 프로덕션 환경에서의 비용 최적화의 차이점

| 항목                   | 기초 아키텍처 특징                                 | 프로덕션 권장 사항                                        |
| ---------------------- | -------------------------------------------------- | --------------------------------------------------------- |
| Azure OpenAI 모델 사용 | 종량제(Global Standard), 호출량 제한 가정          | 비용 최적화 기준 모델 적용 필요                           |
| Foundry Agent Service  | 파일 업로드로 비용 발생, 호출되는 리소스 제어 불가 | 불필요 기능 제한, 연결별 최대 사용량 기준으로 비용 모델링 |
| App Service 요금제     | Basic 요금제, 단일 인스턴스, 고가용성 미보장       | Premium 요금제, 3개 이상 인스턴스로 고가용성 확보         |
| AI Search 요금제       | Basic 요금제, 복제본 없음, 가용성 영역 장애에 취약 | Standard 이상 요금제, 3개 이상 복제본 배포                |
| 비용 거버넌스          | 비용 제한/통제 정책 없음                           | Azure Policy 및 거버넌스 구성 필요                        |



## 8. Premium 요금제 계층 예시 (2025 기준)

| 요금제 | vCPU | RAM   | 자동 확장 | 고가용성 구성 | 워커 인스턴스 |
| ------ | ---- | ----- | --------- | ------------- | ------------- |
| P1v3   | 1    | 3.5GB | O         | 사용자가 구성 | 1~30개 가능   |
| P2v3   | 2    | 7GB   | O         | 사용자가 구성 | 1~30개 가능   |
| P3v3   | 4    | 14GB  | O         | 사용자가 구성 | 1~30개 가능   |

* **Premium 요금제는 워커 인스턴스 수와 직접 연결된 것이 아니라**, 고성능 기능을 제공하는 **상위 요금제 플랜**
* **3개 이상의 워커 인스턴스를 사용하는 것은 고가용성을 위한 권장 사항**이지, 요금제 자체의 요구사항은 아님
* 프로덕션 환경에서는 Premium 요금제를 사용하고, **App Service Plan 설정에서 워커 인스턴스를 3개 이상으로 구성하는 것**이 모범 사례임.



## 7. 참고 사항

* [Design review checklist for Cost Optimization](https://learn.microsoft.com/en-us/azure/well-architected/cost-optimization/checklist)
* 