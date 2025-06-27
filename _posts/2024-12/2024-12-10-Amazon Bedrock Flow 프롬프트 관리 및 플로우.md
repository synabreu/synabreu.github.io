---
title: "Amazon Bedrock Flow: 프롬프트 관리 및 플로우"
date: 2024-12-10
tags: [오픈AI, OpenAI, Open Agents SDK, Swarm, Visual Studio Code, Multi-Agents, 멀티 에이전트]
typora-root-url: ../
toc: true
categories: [AWS]
---

지난 여름에 AWS는 베드락(Bedrock) 서비스에서 **Prompt Management 및 Prompt Flows**를 발표했다. 추후 이  기능은 Bedrock Flows 라는 이름으로 불리게 되었다.  현재 베드락 플로우 서비스는 전 세계 리전에 모두 사용하게 되었다. 

**Amazon Bedrock**은 다양한 AI 모델을 **서버리스 API로 제공**하여, **프롬프트 기반 생성형 AI 애플리케이션을 손쉽게 구축**할 수 있게 해주는 AWS 플랫폼이다. 따라서, Bedrock Flows 는 Amazon Bedrock 의 부속 서비스라고 부를 수 있다. 그렇다면, 오늘은 아마존 베드락 플로우에 대해 한 번 정리해 보도록 하겠다. 



## 1. 베드락 플로우 주요 특징

| 항목                   | 설명                                                         |
| ---------------------- | ------------------------------------------------------------ |
| **시각적 UI**          | 블록 기반 인터페이스로 LLM, Embedding, 분기, 반복, 조건 등 구성 |
| **지원 모델**          | Claude, Mistral, Meta Llama, Amazon Titan 등 Bedrock 내 모델 |
| **워크플로우 기능**    | 입력 → 추론 → 조건 분기 → 외부 호출 → 출력까지 시각적 구성   |
| **RAG 구성 지원**      | Retriever + Prompt Template + LLM 응답 흐름 제공             |
| **No-code / Low-code** | 코드 없이도 실무형 AI 애플리케이션 설계 가능                 |
| **보안/통합**          | IAM, VPC, CloudTrail, S3, Lambda 등 AWS 서비스와 연동 쉬움   |
| **프롬프트 관리**      | 시각적 Prompt 편집기 제공 + Prompt Template 구성 가능        |
| **실행/디버깅**        | UI 상에서 실시간 실행 및 디버깅 결과 확인 가능               |



## 2. 고객 문제 해결

| 기존 문제                                                    | 해결책                                                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 기존에는 LLM 기반 앱을 만들기 위해, <br />* OpenAI, Anthropic 등 API 연결<br />* 프롬프트 설계<br />* 메모리 상태 관리<br />* 분기/조건 흐름 구현<br />* 결과 포맷팅 및 배포 | **시각적 구성**으로 단순화하여 **비개발자도 쉽게 생성형 AI 워크플로우를 구축**할 수 있도록 제공 |



## 3. Amazon Bedrock Flow vs. LangGraph 비교

| 항목                  | **Amazon Bedrock Flow**                                      | **LangGraph (LangChain Flow)**                               |
| --------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **출처**              | AWS 공식 서비스                                              | LangChain (Python 기반 프레임워크)                           |
| **목적**              | **GUI 기반 워크플로우 생성 및 실행 (No-code/Low-code)**      | **코드 기반 복잡한 state-machine LLM 설계 (개발자용)**       |
| **인터페이스**        | AWS Console 기반 시각적 노드 에디터                          | 코드 기반 (Python) + DAG 시각화 도구 (Graphviz 등)           |
| **대상 사용자**       | 비개발자 또는 엔터프라이즈용 사용자                          | LLM 에이전트 및 툴체인 설계가 필요한 개발자                  |
| **실행 환경**         | AWS 클라우드 전용 (Bedrock 모델만 사용 가능)                 | 로컬, 클라우드 모두 가능 (OpenAI, Anthropic 등 자유 사용)    |
| **LLM 모델**          | Amazon Bedrock 지원 모델만 사용 가능 (Claude, Mistral, Titan 등) | OpenAI, Anthropic, HuggingFace 등 **다양한 LLM 자유롭게 연동** |
| **워크플로우 복잡도** | 직관적 GUI로 간단한 흐름 구성 (e.g. QnA → RAG → Output)      | 에이전트 간 메시지 전달, 조건 분기, 루프 등 고난이도 제어 가능 |
| **RAG 구성**          | 사전 템플릿 제공 (e.g. Retrieve → Prompt → Respond)          | 사용자가 자유롭게 정의해야 함 (보다 유연하고 확장성 있음)    |
| **멀티에이전트**      | 공식 지원 없음 (단일 흐름 위주)                              | **멀티에이전트 설계 및 상호작용 제어** 가능                  |
| **시각화 수준**       | 노드/블록 기반 GUI (노코드)                                  | Graphviz 기반 state machine 시각화                           |
| **배포 연계**         | Lambda, S3, API Gateway 등 AWS 서비스와 통합 쉬움            | FastAPI, Streamlit, LangServe 등으로 직접 배포해야 함        |
| **난이도**            | 쉬움 (GUI로 설계 가능)                                       | 높음 (코드 기반 설계 필요)                                   |



## 4.  LangChain Expression Language + LangGraph

* **LangGraph** (또는 **LangChain Flow**라고도 불리며, **LangGraph** 기반으로 동작)

* LangGraph는 **LangChain의 멀티에이전트/워크플로우 설계**를 위해 만든 **state machine 기반 프레임워크**. 복잡한 조건 분기, 루프, 메시지 전달 등을 정의함

* 주요 특징

  | 항목            | 설명                                                         |
  | --------------- | ------------------------------------------------------------ |
  | **엔진**        | Python 기반 Finite-State Machine / Directed Graph            |
  | **시각화 지원** | Flowchart 형태의 **노드 기반 워크플로 시각화** 가능 (Web UI 또는 VS Code 확장) |
  | **주 목적**     | 에이전트 간 대화, 루프, 분기 처리 등 복잡한 reasoning 흐름 설계 |
  | **호환**        | LangChain, OpenAI, Anthropic, HuggingFace, 기타 툴 연동      |

* 사용자 시나리오
  * 사용자가 질문 → RAG → 요약 → 사용자 응답
  * 질문 → 판단 노드 → 검색/계산/대화 노드로 분기
  * 툴이 실패하면 fallback 루프로 되돌리는 에이전트 설계



## 5.  **LangFlow** (비공식 커뮤니티 프로젝트)

* LangChain의 체인 구성 요소를 드래그 앤 드롭으로 시각적으로 설계할 수 있는 **웹 기반 GUI 도구**

* 주요 특징

  | 항목     | 설명                                                      |
  | -------- | --------------------------------------------------------- |
  | **UI**   | 웹브라우저에서 실행되는 **노코드/로우코드 시각적 편집기** |
  | **기능** | Prompt 구성, Memory, LLM, Tool 등 노드로 조립             |
  | **출처** | 커뮤니티 프로젝트 (공식 LangChain 팀 소속은 아님)         |
  | **설치** | `pip install langflow` 후 실행: `langflow run`            |

* 기능
  * 드래그 앤 드롭으로 LLM + Memory + Prompt Template + Output 구조 설계
  * 빠른 프로토타이핑용으로 적합



## 6.  **LangGraph vs. LangFlow**

| 항목      | **LangGraph (공식)**       | **LangFlow (커뮤니티)**         |
| --------- | -------------------------- | ------------------------------- |
| 목적      | 멀티에이전트 FSM 설계      | GUI 기반 시각적 체인 설계       |
| 시각화    | Graph (state machine 기반) | 노드 기반 flowchart             |
| 난이도    | 중급~고급 (개발자 중심)    | 초급~중급 (시각적 프로토타이핑) |
| 안정성    | 공식 지원                  | 커뮤니티 유지 보수              |
| 실행 환경 | Python 코드로 실행         | Web UI로 실행 (Streamlit 기반)  |



## 7. 요약

* 비개발자, AWS 환경 중심, GUI로 빠르게 LLM 워크플로 만들고 싶은 경우 -> Amazon Bedrock Flow
* 개발자, 다양한 LLM 사용, 복잡한 에이전트 논리 및 상태 전이 설계가 필요한 경우 -> LangGraph (LangChain Flow)



## 8. 관련 문서

* [Amazon Bedrock Flow 공식 소개](https://aws.amazon.com/bedrock/) 
* [AWS re:Invent 2023 – Bedrock Flow 발표 세션](https://www.youtube.com/watch?v=jzIZcgaTruA&t=54s)
* **LangGraph 공식 문서**: [https://docs.langchain.com/langgraph/](https://docs.langchain.com/langgraph/)
* **LangFlow GitHub**: [https://github.com/logspace-ai/langflow](https://docs.langchain.com/langgraph/)