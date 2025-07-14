---
title: "Azure AI Foundry 기반 챗봇 베이직 아키텍처 분석(5)-운영 우수성"
date: 2025-01-06
tags: [마이크로소프트, Microsoft, Build 2025, Azure AI Foundry, Azure, Azure AI Foundry SDK, Azure OpenAI Studio, Azure OpenAI Service, Azure Machine Learning, Azure App Service, Azure Key Vault, Azure Monitor]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

비용 최적화는 **불필요한 지출을 줄이고 운영 효율성을 개선**하는 방법에 중점을 둔다. Azure AI Foundry 기반 챗봇 기초 아키텍처 분석 중 비용 최적화에 대한 노트를 정리해 보자. 



## 1. 챗봇 베이직 아키텍처 모니터링

* 챗봇 베이직 아키텍처는 모든 서비스에 대해 진단 구성이 되어 있음
* App Service와 Azure AI Foundry를 제외한 모든 서비스는 로그를 수집함
* **App Service**는 다음 로그를 수집함
  * `AppServiceHTTPLogs`
  * `AppServiceConsoleLogs`
  * `AppServiceAppLogs`
  * `AppServicePlatformLogs`
* **Azure AI Foundry**는 `RequestResponse` 로그를 수집함
* 개념 검증(PoC) 단계에서는 수집 가능한 로그와 메트릭이 무엇인지 파악해야 함
* 프로덕션 단계로 전환할 때는 유용하지 않고 비용만 발생시키는 로그 소스는 제거하는 것
* Azure AI Foundry의 모니터링 기능을 사용하려면 **Application Insights 리소스를 Azure AI Foundry 프로젝트에 연결**해야 함
  * 프롬프트, 응답, 전체 토큰 등 **토큰 사용량의 실시간 모니터링**
  * 지연 시간(latency), 예외, 응답 품질 등 **요청-응답에 대한 상세 텔레메트리**
  * **OpenTelemetry**를 활용한 에이전트 추적 기능도 제공함



## 2. 모델 개발 및 평가, 운영 

* **Azure AI Foundry의 베이직 챗봇 아키텍처 개발**

  * **Azure AI Foundry 포털의 브라우저 기반 UI**를 통해 에이전트를 생성할 수 있음
  * 프로덕션 단계로 나아갈 때는 **버전 관리와 개발 프로세스에 대한 베이스라인 아키텍처 가이드**를 따르는 것이 좋음
  * 더 이상 필요 없는 에이전트는 반드시 삭제해야 하며, 해당 에이전트가 사용하는 연결(Connection)이 마지막 하나라면 그 연결도 함께 제거해야 함

* **Azure AI Foundry의 베이직 챗봇 아키텍처 평가**

  * Azure AI Foundry에서는 생성형 애플리케이션에 대한 평가 기능을 제공함
  * 생성형 AI 애플리케이션을 평가하는 방법을 배우는 것이 중요 -> 과정을 통해 선택한 모델이 고객 및 워크로드의 설계 요구사항을 충족하는지 확인함

* **Azure AI Foundry의 모델 운영 가이드 준수:** 챗봇 기초 아키텍처는 학습 목적으로 최적화되어 있으며, 프로덕션 사용을 위한 것이 아니므로 GenAIOps와 같은 운영 가이드는 포함되어 있지 않음 

  

## 3. 성능 효율성 (Performance Efficiency)

* 성능 효율성은 워크로드가 사용자 수요에 맞춰 **효율적으로 확장(Scale)** 될 수 있는 능력

* 챗봇 베이직 아키텍처는 **프로덕션 배포를 위한 설계가 아니기 때문에**, 다음과 같은 **핵심 성능 효율성 기능이 생략**되어 있음.

  * 개념 검증(POC)) 결과를 통해 워크로드에 가장 적합한 **App Service 제품**을 선택해야 함
  * 워크로드는 수요를 효율적으로 처리할 수 있도록 **수평 확장(horizontal scaling)** 을 고려하여 설계해야 함
    * 수평 확장은 App Service 플랜에서 컴퓨팅 인스턴스의 수를 조절함으로써 이뤄짐
    * 수요에 따라 컴퓨팅 제품을 변경해야만 대응 가능한 구조는 피하라.

* 챗봇 베이직 아키텍처는 대부분의 구성 요소에서 **소비 기반(Consumption)** 또는 **사용량 기반 요금제(Pay-as-you-go)** 모델을 사용함

* 챗봇 베이직 모델은 **최선의 노력을 기반으로 한 모델(Best-effort model)** 이며, **노이즈 이웃 문제(noisy neighbor problem)** 또는 기타 플랫폼 부하 문제가 발생할 수 있음

* 프로덕션 환경으로 전환할 때는 애플리케이션이 [**프로비저닝된 처리량(Provisioned Throughput, PTU)**을](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/concepts/provisioned-throughput?tabs=global-ptum) 필요로 하는지 판단해야 함

  * **프로비저닝된 처리량**은 Azure OpenAI 모델 배포를 위해 **처리 용량을 미리 예약**할 수 있도록 해줌
  * **예약된 용량(Reserved Capacity)** 을 사용하면 모델에 대해 **예측 가능한 성능과 처리량**을 확보할 수 있음

  

## 4. 기타 설계 권장 사항

* AI 솔루션 아키텍트는 챗봇과 같은 AI 및 머신러닝 워크로드를 설계할 때, [**Azure의 Well-Architected Framework의 AI 워크로드 설계 가이드**를](https://learn.microsoft.com/en-us/azure/well-architected/ai/get-started) 참고해야 함

* 시나리오 배포: 권장 사항과 고려사항을 반영한 **참조 구현(reference implementation)** 을 배포 권장

  

## 5. 참고 자료

* [성능 효율성 설계 검토 체크리스트](https://learn.microsoft.com/en-us/azure/well-architected/performance-efficiency/checklist)

* [Azure AI Foundry Agent service chat basic reference implementation 오픈소스](https://github.com/Azure-Samples/openai-end-to-end-basic/)

* [AI workloads on Azure](https://learn.microsoft.com/en-us/azure/well-architected/ai/get-started)

* [Deployment overview for Azure AI Foundry Models](https://learn.microsoft.com/en-us/azure/ai-foundry/concepts/deployments-overview)

* [Explore Azure AI Foundry Models](https://learn.microsoft.com/en-us/azure/ai-foundry/concepts/foundry-models-overview)

* [What is Azure AI Foundry Agent Service?](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/overview)

  
