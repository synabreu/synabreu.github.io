---
title: "ㅁWS 서버리스 생성형 AI 플랫폼, Amazon Bedrock"
date: 2024-12-01
tags: [아마존, Amazon, AWS, Bedrock, Agent, 생성형AI, Generative AI]
typora-root-url: ../
toc: true
categories: [AWS]
---

AWS에서 제공하는 서버리스(serverless) 생성형 AI 플랫폼으로, 다양한 최신 대규모 언어 모델(LLM)을 API 형태로 손쉽게 사용할 수 있도록 해준다. 사용자는 인프라를 직접 관리하지 않고도 챗봇, 요약, 분류, 검색, RAG 등 생성형 AI 애플리케이션을 신속하게 개발할 수 있는 서비스가 바로 **아마존 베드락(Bedrock)** 이다. 

이 아마존 베드락은 2023년에 처음 출시되었으며, 2024년 10월 2일부터 국내에서 서울 리전으로 사용 가능해졌다. 주로 사내 문서 기반 질의 응답 시스템 (RAG), 고객 상담 자동화 봇, 계약서 요약 및 분류 시스템 

특히, 사용자가 고성능의 다양한 대규모 언어 모델(LLM) 및 파운데이션 모델에 쉽게 접근할 수 있다. 그동안 아마존 베드락 서비스에 대해 익히 소식을 듣고 있었지만 어떠한 개념이고, 무엇을 할 수 있는 지, 대략적으로 알았지만 정리할 기회가 없어서 오늘은 이에 대해 정리해 보겠다. 



## 1. 베드락 서비스의 기능

| 기능                                 | 설명                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| **LLM 추론 API**                     | Anthropic Claude, Meta Llama, Cohere, AI21, Amazon Titan 등 다양한 모델 제공 |
| **RAG 구성**                         | 벡터 DB, Embedding, 검색 → LLM 연계로 문서 기반 QA 구현      |
| **시각적 워크플로우 (Bedrock Flow)** | LLM 활용 파이프라인을 드래그 앤 드롭으로 설계                |
| **프롬프트 관리**                    | Prompt Template 및 버전 관리 기능                            |
| **보안 & 권한 관리**                 | IAM, VPC, KMS 연동 등 기업용 보안 제어                       |
| **서버리스 & 비용 최적화**           | 인프라 관리 없이 API 호출만으로 비용 청구                    |



## 2. 베드락 주요 컴포넌트

| 구성 요소                    | 설명                                                        |
| ---------------------------- | ----------------------------------------------------------- |
| **Foundational Models (FM)** | Claude, Mistral, Meta Llama, Cohere 등 다양한 LLM 선택 가능 |
| **Model Invocation API**     | 프롬프트를 API로 전송해 응답 받는 추론 엔드포인트           |
| **Agents for Bedrock**       | Tool-Using Agent 시스템 지원 (예: API 호출, DB 검색 등)     |
| **Knowledge Bases**          | S3에 저장된 문서 → Embedding → 벡터 검색 → LLM 응답         |
| **Prompt Management**        | 프롬프트 템플릿 및 버전 관리 (기업 내 협업 기능)            |
| **Bedrock Flow**             | 시각적 워크플로우 설계 도구 (노코드)                        |



## 3. 베드락 서비스에서 지원하는 모델

| 제공 모델                | 제공사                                                       |
| ------------------------ | ------------------------------------------------------------ |
| **Claude 3 시리즈**      | Anthropic. **Claude 3.5 Sonnet**: 대규모 텍스트 생성에 사용. **Claude 3 Haiku**: 간단한 작업을 위한 경량 NLP 모델 |
| **Mistral 7B / Mixtral** | Mistral                                                      |
| **Llama 3**              | Meta                                                         |
| **Command R / R+**       | Cohere                                                       |
| **Jurassic-2**           | AI21 Labs                                                    |
| **Titan Text/Image**     | Amazon 자체 모델. 의미 분석에 특화.                          |



## 4. 베드락 서비스의 장점

| 장점                   | 설명                                                         |
| ---------------------- | ------------------------------------------------------------ |
| **모델 선택 자유도**   | 다양한 기업의 최신 모델을 하나의 API로 이용. 복잡한 인프라 관리 없이 높은 성능의 AI 모델을 쉽게 통합 |
| **서버리스**           | 모델 호스팅 불필요, 인프라 자동 관리. 별도의 서버 관리가 필요 없어 개발자는 애플리케이션 로직에 집중함. |
| **기업 친화적**        | IAM, 보안, 모니터링, 결제 통합                               |
| **통합형 워크플로우**  | 프롬프트 설계부터 배포까지 AWS 안에서 가능                   |
| **비용 효율성**        | 사용자는 모델에 따라 요금을 지불할 수 있으며, 온디맨드 방식으로 필요한 만큼만 사용 |
| **보안 및 프라이버시** | 요청 및 응답 데이터는 암호화되며, IAM 정책을 통한 권한 제어가 가능 |



## 5. 베드락 서비스의 단점

* **비용 발생 불확실성**: 사용량에 따라 비용이 달라질 수 있어 예측하기 어려운 경향이 있음
* **모델 선택의 어려움**: 다양한 모델 중에서 어떤 것을 선택해야할지 고민할 수 있습니다. 잘못된 모델 선택이 비효율성을 야기할 수 있음
* **개발자 전문성 필요**: 초기 설정이나 복잡한 기능을 사용하는 데 있어 일부 기술적 전문성이 요구됨



## 6. 결론

* Amazon Bedrock은 AWS의 생성형 AI 서비스로, 쉽게 다양한 AI 모델을 사용할 수 있는 혜택을 제공함.
* 기능 저항성이 뛰어나고 비용 효율적인 서버리스 구조를 통해 기업들이 AI를 손쉽게 통합할 수 있음.
* 불확실한 비용 구조와 모델 선택의 어려움은 신중하게 고려해야 할 요소로 뽑힘.
* 기업은 필요와 예산에 맞는 전략적 접근이 필요함.



## 7. 참고 자료

* DevelopersIO. (2024). [Amazon Bedrock 서울 리전  공식 출시](https://dev.classmethod.jp/articles/jw-amazon-bedrock-is-now-available-in-seoul-region/)
* Jibinary. (2024). [AWS 주간 소식 모음: Amazon Bedrock 및 Amazon Q의 주요 기능들 출시](https://jibinary.tistory.com/714)
* Caylent. (2023). [Amazon Bedrock Pricing Explained | Caylent](https://caylent.com/blog/amazon-bedrock-pricing-explained/)
* NDS Cloud Tech Blog. (2023). [AWS의 생성형 AI 서비스 Amazon Bedrock이란?](https://m.blog.naver.com/gabianow/223881948359)
* CloudApper. (2024). [What Are the Amazon Bedrock Development Challenges?](https://www.cloudapper.ai/ai-technology/what-are-the-amazon-bedrock-development-challenges/)
* AWS Skill Builder: [Amazon Bedrock Getting Started](https://skillbuilder.aws/learn/63KTRM86DQ/amazon-bedrock-getting-started/SC2Y3HMAUE)