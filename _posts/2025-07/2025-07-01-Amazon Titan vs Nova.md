---
title: "Amazon Titan vs Nova"
date: 2025-07-01
tags: [아마존, Amazon, AWS, Amazon Bedrock, Agent, 생성형 AI, Generative AI, Agentic AI, Strands Agents SDK, Amazon Nova, Amazon Nova Canvas, Amazon Titan]
typora-root-url: ../
toc: true
categories: [AWS]
---

아마존 타이탄(Amazon Titan)은 간단하고 **경제적인 텍스트 중심** 작업에 적합하고, **Nova**는 **멀티모달/고성능/에이전트형 AI**가 필요한 기업용 차세대 모델이다. Amazon Bedrock에서는 이 두 모델을 **서로 보완적으로 사용**할 수 있으며, Titan으로 문서 임베딩 후, Nova로 자연어 Q&A 생성 및 추론하는 방식도 가능하다. 

**Amazon Titan**과 **Amazon Nova**는 모두 AWS에서 제공하는 **파운데이션 모델(FM, Foundation Model)**이지만, **목적**, **설계 철학**, **기술 범위**, **활용 시나리오** 면에서 다음과 같이 **차이점과 공통점**이 있다.



## 1.  Amazon Titan

* Amazon Titan은 **Amazon Bedrock**에서 제공되는 **파운데이션 모델(FM)** 시리즈이다. 
* **용도**: 텍스트 생성, 텍스트 임베딩, 검색, 분류 등 다양한 **NLP 작업**에 사용
* **모델 종류**:
  * `Titan Text` (예: Titan Text G1): 텍스트 생성 및 이해
  * `Titan Embeddings`: 검색/추천/분류를 위한 임베딩 제공
* **특징**: AWS에서 직접 개발한 모델이며, 기업용 안전성과 성능에 최적화되어 있음.



## 2.  Amazon Nova

* Amazon Nova는 **AWS의 새로운 AI 인프라** 또는 **모델/서비스 브랜드 이름**으로 최근 등장한 용어
* **용도**:
  * **Nova 인스턴스 타입**: 예를 들어, `Amazon EC2 Trn1n`, `Inf2`, 또는 `HyperPod`와 연계된 AI 인프라에 해당할 수 있음
  * 또는 내부적으로 LLM 기반 API나 AI assistant 이름으로도 사용될 가능성 있음 (예: Amazon Q와 비슷한 서비스)
* AWS Bedrock에서 제공되는 **차세대 파운데이션 모델 시리즈**로, **최첨단 성능과 뛰어난 가격 대비 효율**을 제공gka
  * **통합 플랫폼**: Bedrock API와 완전히 통합되어, 개발자가 쉽게 호출 및 실험 가능
  * **안전 및 책임성**: 콘텐츠 필터링, 워터마킹 등의 책임감 있는 AI 설계 포함 
  * **언어 및 모달 범위가 넓음**: 200개 이상의 언어 지원, 멀티모달 입력(텍스트·이미지·비디오·음성) 수용 



## 3.  Amazon Nova 모델 구성 요소

* Nova는 크게 **이해형**, **창조형**, **음성형** 모델로 나눈다.

* 이해형(Understanding) 모델

  | 이름         | 설명                                                         |
  | ------------ | ------------------------------------------------------------ |
  | Nova Micro   | * 텍스트 전용, 초저지연, 비용 최적화<br />* 128K 토큰, 200+ 언어 지원, 빠른 수백 토큰/초 생성<br /> |
  | Nova Lite    | * 멀티모달(텍스트·이미지·비디오 입력), 경량형, 낮은 비용<br />* 300K 토큰, 고속 처리, 많은 작업에 적합<br /> |
  | Nova Pro     | * 멀티모달, 정확도, 속도, 비용 균형 최적화<br />* 1M 토큰, 모델 증류용 *Teacher* 역할 수행, 고성능 커스텀 모델 생성 가능<br /> |
  | Nova Premier | * 최고급 복잡 작업 대응<br />* 1M 토큰, 모델 증류용 *Teacher* 역할 수행, 고성능 커스텀 모델 생성 가능<br /> |

* 창조형(Creative) 모델

  | 특징        | 설명                                                         |
  | ----------- | ------------------------------------------------------------ |
  | Nova Canvas | * 텍스트/이미지 → 고품질 이미지 생성<br />* 편집, 시각적 조정, 워터마킹 포함<br /> |
  | Nova Reel   | * 텍스트/이미지 → 동영상 생성<br />* 카메라 동작 및 스타일 제어, 워터마킹·필터링 포함<br /> |

* 음성형(Speech-to-Speech) 모델

  | 특징       | 설명                                                         |
  | ---------- | ------------------------------------------------------------ |
  | Nova Sonic | * 음성 → 텍스트+음성 대화 생성<br />* 실시간 스트리밍, 함수 호출, RAG, 영어(미국+영국 액센트) 지원<br /> |

  

## 3.  Amazon Nova 기능

* **멀티모달 지원**: 200개 언어 이상, 이미지·비디오·음성 등 입력 가능
* **긴 문맥 처리**: 최대 1M 토큰 (Nova Premier 기준)
* **책임성 확보**: 워터마킹 및 콘텐츠 필터링 내장
* **비용 및 속도 최적화**: 경쟁 제품 대비 75% 이상 저렴, 빠른 응답성
* **Agentic & RAG 역량**: 다단계 워크플로우, 함수 호출, 도메인 지식 기반 응답 가능
* **모델 증류 및 커스터마이징**: Premier를 Teacher 모델로 활용하여 경량화 및 고효율 모델 생성 가능



## 4.  Amazon Nova 활용 사례

* **RAG 기반 Q&A 및 문서 요약** – Nova Lite/Pro + Bedrock Knowledge Base
* **코딩 지원** – Micro의 빠른 텍스트 세대 활용
* **멀티모달 에이전트** – 웹 브라우저 조작용 Nova Act SDK 연구용 프리뷰 제공
* **광고 콘텐츠 제작** – Shutterstock, Dentsu 등에서 이미지/비디오 제작에 Canvas/Reel 활용
* **콜센터 음성봇** – Nova Sonic의 실시간 자연어 대화 기능 활용 (예: 텔레콤 사례)



## 5.  가용 지역

* **Nova Micro/Lite/Pro**: US East(N. Virginia), US West(Oregon), US East(Ohio), AWS GovCloud(US-West)
* **Canvas/Reel**: US East(N. Virginia), EU(Ireland), APAC(Tokyo)
* **Premier** 및 Sonic는 특정 리전에 단계적으로 출시 중이며, Bedrock 콘솔에서 확인 가능



## 6.  Amazon Nova 시리즈

| 모델         | 입력 가능 모달리티        | 최대 토큰/길이 | 주요 사용처                                    |
| ------------ | ------------------------- | -------------- | ---------------------------------------------- |
| Nova Micro   | 텍스트                    | 128K           | 빠른 응답, 코딩, 계산, 챗봇                    |
| Nova Lite    | 텍스트·이미지·비디오      | 300K           | 문서/비디오 요약, RAG, 저비용 멀티모달 응용    |
| Nova Pro     | 텍스트·이미지·비디오      | 300K           | 정확·효율적 멀티모달 작업, 에이전트 워크플로우 |
| Nova Premier | 텍스트·이미지·비디오      | 1M             | 복잡한 문맥, 모델 증류, 중대형 워크플로우      |
| Nova Canvas  | 텍스트·이미지             | –              | 광고·마케팅 이미지 제작                        |
| Nova Reel    | 텍스트·이미지             | –              | 마케팅/트레이닝용 짧은 동영상 생성             |
| Nova Sonic   | 음성 ↔ 텍스트·음성 쌍방향 | –              | 실시간 음성 대화, 콜센터, 함수 호출            |



## 7.  Amazon Titan vs. Nova

| 항목               | **Amazon Titan**                             | **Amazon Nova**                                              |
| ------------------ | -------------------------------------------- | ------------------------------------------------------------ |
| **출시 시기**      | 2023년 (Bedrock 초창기부터)                  | 2024년 말 ~ 2025년 (최신 세대)                               |
| **제공처**         | AWS 자체 개발 (Bedrock)                      | AWS 자체 개발 (Bedrock)                                      |
| **주요 모델 종류** | - Titan Text G1- Titan Embeddings G1         | - Nova Micro- Nova Lite- Nova Pro- Nova Premier- Nova Canvas- Nova Reel- Nova Sonic |
| **모달리티**       | 텍스트 전용 (텍스트 생성, 임베딩)            | **멀티모달** (텍스트, 이미지, 비디오, 음성 포함)             |
| **지원 기능**      | - 텍스트 생성- 임베딩- 검색, 분류, RAG 등    | - 생성 + 이해 + 멀티모달 작업- 고속 추론, 긴 컨텍스트 처리- 실시간 음성- 이미지/비디오 생성 |
| **컨텍스트 길이**  | ~8K–32K (모델별 상이)                        | 최대 **1M 토큰** (Nova Premier)                              |
| **특화 역량**      | - **문서 임베딩**- 저렴한 비용의 텍스트 모델 | - **에이전트/멀티모달 RAG**- 실시간 앱용 음성/이미지 생성- LLM 파이프라인 통합 최적화 |
| **비용/속도 효율** | **비용 효율 중시**, 상대적으로 가벼움        | **성능과 유연성 중시**, 고성능/대규모 워크플로우 대응        |
| **RAG 적합도**     | Titan Embeddings + 다른 모델 조합            | Nova Pro + Knowledge Base 기반 고급 RAG 최적화               |
| **적용 사례**      | - 검색 시스템- 추천엔진- FAQ 생성            | - 대화형 AI- 코드 보조- 다국어 RAG 에이전트- 마케팅용 이미지/영상 제작- 실시간 음성 비서 |



## 8.  결론

* **텍스트 중심이면**: 빠른 Micro
* **멀티모달 이해형이면**: Lite 또는 Pro
* **복잡한 맥락이나 모델 증류면**: Premier
* **크리에이티브 컨텐츠면**: Canvas (이미지) / Reel (비디오)
* **음성 대화면**: Sonic



## 9.  참고 기사

* Amazon Nova Foundation Models: [Amazon Nova Foundation Models](https://aws.amazon.com/ai/generative-ai/nova/)

* AWS News Blog: [Introducing Amazon Nova foundation models: Frontier intelligence and industry leading price performance](https://aws.amazon.com/blogs/aws/introducing-amazon-nova-frontier-intelligence-and-industry-leading-price-performance/?utm_source=chatgpt.com)

* Amazon: [Amazon makes it easier for developers and tech enthusiasts to explore Amazon Nova, its advanced Gen AI models](https://www.aboutamazon.com/news/innovation-at-amazon/amazon-nova-website-sdk?utm_source=chatgpt.com)

* Time: [Why Amazon Web Services CEO Matt Garman Is Playing the Long Game on AI](https://time.com/7225660/amazon-aws-matt-garman-interview/?utm_source=chatgpt.com)

* TheVerge: [Amazon announces its own set of Nova AI models](https://www.theverge.com/2024/12/3/24312260/amazon-nova-foundation-ai-models-anthropic?utm_source=chatgpt.com)

  

