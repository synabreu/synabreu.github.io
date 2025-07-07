---
title: "Azure AI Studio"
date: 2025-04-01
tags: [마이크로소프트, Microsoft, Build 2025, Azure AI Studio, Azure OpenAI Service, Azure Cognitive Search, Azure Machine Learning, Azure Functions]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

**Azure AI Studio**는 Microsoft Azure에서 제공하는 **생성형 AI 기반 애플리케이션을 개발, 테스트, 배포할 수 있는 통합 개발 환경(IDE)**이다. 특히 OpenAI, Hugging Face, Meta 등의 최신 LLM 모델을 활용한 애플리케이션을 **코드 작성 없이도 빠르게 프로토타이핑**할 수 있도록 도와준다. 간단하게 Azure AI Studio에 대해 노트를 작성해 보았다. 



## 1. Azure AI Studio 주요 특징  

| 기능                                | 설명                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| **드래그 앤 드롭 방식 UI**          | 워크플로우 기반으로 AI 앱을 시각적으로 구성 가능             |
| **다양한 LLM 지원**                 | GPT-4, LLaMA, Mistral 등 다양한 모델을 손쉽게 선택하여 사용  |
| **프롬프트 작성 및 튜닝 도구 내장** | 프롬프트 엔지니어링 실험 및 평가 가능                        |
| **RAG 지원**                        | 자체 데이터에 기반한 검색 기반 생성(Retrieval-Augmented Generation) 기능 제공 |
| **Fine-tuning 지원**                | Custom 데이터로 모델 파인튜닝 가능 (Azure Machine Learning 연계) |
| **모델 평가 및 비교**               | 다양한 모델 간 성능 및 응답 비교를 위한 실험 기능 제공       |
| **배포 및 API화**                   | 만든 에이전트를 Azure API로 즉시 배포 가능                   |



## 2. Azure 작업 시나리오  

* **AI 챗봇 제작**: 자체 문서나 DB에 연결해 RAG 기반의 응답 가능
* **프롬프트 실험**: 다양한 LLM을 대상으로 빠르게 프롬프트 테스트
* **Chain-of-Thought 구성**: 여러 단계의 reasoning을 시각적으로 구성
* **모델 비교 평가**: A/B 테스트나 human preference 평가 등 지원
* **Web앱 및 API 배포**: 만든 AI 앱을 API 또는 웹 앱 형태로 배포 가능



## 3. Azure 관련 서비스와의 연계  

* **Azure OpenAI Service**: GPT-4, GPT-3.5 등 모델 호출
* **Azure Cognitive Search**: RAG에서 검색 기능 수행
* **Azure Machine Learning**: 학습 및 파인튜닝 파이프라인 연동
* **Azure Functions / Logic Apps**: 워크플로우 자동화와 통합



## 4. Azure 용도  

| 시나리오                 | 활용 방식                                      |
| ------------------------ | ---------------------------------------------- |
| **고객지원 봇 구축**     | 내부 FAQ PDF 연동 + GPT-4 기반 응답 생성       |
| **문서 요약 시스템**     | 대용량 문서 입력 → 핵심 요약 제공              |
| **코드 리뷰 어시스턴트** | GitHub 연동 → 코드 리뷰 및 코멘트 생성         |
| **마케팅 문구 생성기**   | 제품 설명 → 이메일/SNS 마케팅 콘텐츠 자동 생성 |



## 5. 결론  

* Azure AI Studio는 비즈니스 사용자와 개발자 모두가 생성형 AI 앱을 쉽고 빠르게 구축하고 배포할 수 있는 저코드/노코드 개발 플랫폼
* Microsoft의 생태계(Azure OpenAI, Search, ML 등)와의 통합이 강력한 장점