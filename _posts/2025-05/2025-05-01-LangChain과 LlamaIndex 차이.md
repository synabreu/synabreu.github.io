---
title: "LangChain과 LlamaIndex 차이"
date: 2025-05-01
tags: [LangChain, LlamaIndex, OpenSource, Agent, LLM, RAG]
typora-root-url: ../
toc: true
categories: [LangChain, LlamaIndex, OpenSource]
---

그동안 LLM RAG나 Agent 를 구축할 때, LangChain과 LlamaIndex 프레임워크를 많이 사용했다. 하지만 이 두개의 프레임워크가 어떠한 차이점이 궁금했는 데, 이를 한번 정리해 보겠다. 

참고로 LlamaIndex와 LangChain은 **RAG (Retrieval-Augmented Generation)** 아키텍처를 기반으로 한 **LLM 기반 애플리케이션 개발을 도와주는 프레임워크**이지만, 역할이 다르다.



## 1. LangChain과 LlamaIndex 차이

| 항목          | LlamaIndex                                        | LangChain                                           |
| ------------- | ------------------------------------------------- | --------------------------------------------------- |
| **주요 목적** | 비정형 데이터(문서, PDF, DB 등)를 LLM에 쉽게 연결 | 다양한 LLM 앱을 구성하는 체인/워크플로우 프레임워크 |
| **초점**      | **데이터 연결 및 인덱싱**                         | **LLM 기능 조합 및 파이프라인 구성**                |
| **철학**      | "LLM을 위한 데이터 레이어"                        | "LLM을 위한 파이프라인 엔진"                        |



## 2. 기능적 차이

| 기능 영역              | LlamaIndex                                                   | LangChain                                                    |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **데이터 연결**        | 파일, DB, API 등 다양한 소스에서 데이터를 불러오고 인덱싱 (DocumentLoader, VectorStore 등 내장) | 외부 라이브러리를 통해 처리하거나 LlamaIndex와 연동          |
| **인덱싱 & 검색**      | 자체 인덱스 구조 (`VectorStoreIndex`, `TreeIndex`, `ListIndex` 등)로 **문맥 검색 최적화** | 주로 외부 vector DB나 LlamaIndex를 사용                      |
| **LLM 호출 체인 구성** | 단일 프롬프트 기반 RAG 구조에 특화됨                         | 다양한 체인 (`LLMChain`, `AgentChain`, `ToolChain`) 조합 가능 |
| **에이전트 시스템**    | 기본적 또는 외부 LangChain Agent 활용                        | 복잡한 LLM 기반 에이전트 시스템 지원 (Tool, ReAct 등)        |
| **유연성**             | RAG 구조에 특화된 간결한 API                                 | 복잡한 체인 로직, 에이전트 구성 등 유연한 확장 가능          |



## 3. 사용 예시

|      | LlamaIndex                                             | LangChain                                                    |
| ---- | ------------------------------------------------------ | ------------------------------------------------------------ |
|      | PDF, Notion, Google Docs → Embedding → LLM 질의 응답   | 질의 → 검색 → LLM → 요약 → Tool 호출 → 응답                  |
|      | 자체 문서에 최적화된 인덱스 구성으로 빠른 검색 및 응답 | 복잡한 대화형 에이전트 구축 (예: ChatGPT 플러그인 같은 구조) |



## 4. 통합성

* 실제로는 **LlamaIndex + LangChain을 함께 사용하는 경우**도 많음
  * LlamaIndex로 데이터 처리 및 인덱싱
  * LangChain으로 에이전트 및 전체 워크플로우 구성



## 5. 결론

| 정리           | 추천 사용 목적                                            |
| -------------- | --------------------------------------------------------- |
| **LlamaIndex** | RAG 기반 검색 특화, 문서 기반 질문응답 시스템 구축에 탁월 |
| **LangChain**  | 다양한 툴/에이전트/체인을 활용한 복합 LLM 앱 개발에 적합  |