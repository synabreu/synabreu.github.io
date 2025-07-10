---
title: "Azure AI Foundry(2)-Azure AI Foundry 아키텍처"
date: 2025-05-25
tags: [마이크로소프트, Microsoft, Build 2025, Azure AI Foundry, Azure, Azure AI Foundry SDK, Azure OpenAI Studio, Azure AI Studio]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

 Azure AI Foundry 서비스는 2024년 12월, Microsoft Ignite 행사에서 샤티야 회장이 처음 소개했다. 이 노트에서는 Azure AI Foundry 서비스에 대해 좀더 깊게 파고들기 위해 아키텍처에 대해 노트를 한번 정리해 보도록 하겠다. 

![그림1 - Azure AI Foundry 아키텍처](/../images/2025-05/AzureAI-04.png)

## 1. Azure AI Foundry 아키텍처  

* 위의 그림은 왼쪽에는 개발 도구나 개발통합환경(IDE)에 관한 내용이 있고, 가운데에는 모델 카탈로그, 오른쪽에는 관측 가능성(Observability)에 대한 내용으로 나눠진다. 
* Azure AI Foundry: 다양한 AI 모델에 접근할 수 있는 플랫폼
  *  **"다양한 AI 모델"**이란 OpenAI 모델만이 아니라 다른 벤더의 모델들까지 포함
  * AI 모델, 서비스, 그리고 통합 기능을 통해 개발자가 AI 기반 애플리케이션을 구축하고, 커스터마이징하며, 배포함



## 2.  Azure AI Foundry 개발 도구  

* **Copilot Studio**: AI 기반 코파일럿을 구축하기 위한 도구

* **Visual Studio**: Microsoft의 인기 있는 통합 개발 환경(IDE)

* **GitHub**: 이미 많은 개발자들이 사용 중인 협업 및 버전 관리 플랫폼으로, AI 프로젝트에도 활용

* **Azure AI Foundry SDK:** AI 모델 및 서비스와 상호작용할 수 있게 해주는 도구. 동일한 SDK로 다양한 모델에 접근할 수 있어 AI 개발자에게 엄청난 가능성과 기회를 열어 줌

  

