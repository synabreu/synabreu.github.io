---
title: "[실습] OpenAI 관리형 RAG API 워크샵(1)"
date: 2026-08-26
tags: [오픈AI, OpenAI, GPT-5.6, agenticai, aiagent, openrouter, docker, powershell, fastapi, swagger-ui, visual studio code, Linux, Windows11, Kubernetes, On-Premises ]
typora-root-url: ../
toc: true
categories: [openai]
---

국내 대기업에 다양한 분야의 RAG 와 Agent 개발 경험을 바탕으로 해 기업용 문서(markdown 파일, PDF 문서) 등을 **OpenAI의 관리형 RAG API**를 직접 호출해 검색()Retrieval)과 생성(Generation)의 차이를 이해하도록 만든 초보자들도 따라해 볼 수 있는 파이썬 언어로 만든 핸즈온 워크삽이다.

# 1. 워크삽 핵심 학습 목표

- RAG의 검색(Retrieval)과 생성(Generation)을 구분한다.
- OpenAI의 벡터 스토어(Vector Store)를 생성하는 사용법을 배운다. 
- **OpenAI Retrieval API**를 직접 호출하는 방법을 배운다.
- `vector_stores.search()` 결과와 유사성 스코어(similarity score)를 확인한다.
- Responses API의 `file_search`로 관리형 RAG를 구현한다.
- FastAPI로 `/retrieve`, `/ask` 엔드포인트를 만든다.
- 전체 흐름을 기업에서 바로 쓸 수 있는 응용 AI 아키텍처 관점에서 이해한다.

## 2. 워크삽 프로젝트 구조

```text
openai-rag-workshop/
├── .env.example                         # 환경 변수 작성 예시
├── .gitignore
├── README.md                            # 프로젝트 소개와 빠른 실행 방법
├── requirements.txt                    # Python 패키지 목록
├── data/
│   └── vector_store.json               # Lab 1 실행 후 Vector Store ID 저장
├── docs/
│   ├── 01_concepts.md                   # RAG 핵심 개념
│   ├── 02_workshop.md                   # Lab 1~4 실습 과정
│   └── 03_applied_ai_architecture.md    # Applied AI 아키텍처 설명
├── knowledge_files/
│   ├── 230918_금융소비자용 금소법 안내자료_f.pdf
│   ├── company_ai_policy.md
│   └── gpu_operations_guide.md
└── src/
    ├── __init__.py                      # Python 패키지 선언
    ├── api.py                           # FastAPI 엔드포인트
    ├── ask_rag.py                       # Responses API와 File Search 실행
    ├── config.py                        # 환경 변수와 Vector Store ID 관리
    ├── index_documents.py               # 지식 문서 업로드 및 인덱싱
    └── search_retrieval.py              # Retrieval API 직접 검색
```

`knowledge_files/`의 Markdown 문서 2개와 PDF 문서 1개가 하나의 Vector
Store에 인덱싱된다. 이후 예제에서는 새로 추가된 금융소비자보호법 안내 PDF를
검색 대상으로 사용한다.

# 3. 워크삽 프로젝트에서 사용하는 OpenAI 기능

```text
OpenAI Files / Documents
        |
        v
OpenAI Vector Stores
        |
        +-----------------------------+
        |                             |
        v                             v
Retrieval API                  Responses API
vector_stores.search()         + file_search
        |                             |
        v                             v
검색 chunk 확인                 근거 기반 답변 생성
        |                             |
        +--------------+--------------+
                       |
                       v
                    FastAPI
```

# 4. 워크삽 프로젝트 목차

* [[실습] OpenAI 관리형 RAG API 워크샵(2)-RAG](https://synabreu.github.io/openai/%EC%8B%A4%EC%8A%B5-OpenAI-%EA%B4%80%EB%A6%AC%ED%98%95-RAG-API-%EC%9B%8C%ED%81%AC%EC%83%B5(2)-RAG/)
* [[실습] OpenAI 관리형 RAG API 워크샵(3)-VectorStore](https://synabreu.github.io/openai/%EC%8B%A4%EC%8A%B5-OpenAI-%EA%B4%80%EB%A6%AC%ED%98%95-RAG-API-%EC%9B%8C%ED%81%AC%EC%83%B5(3)-VectorStore/)



# 5. 오픈소스 

이 워크삽에서 진행하는 파이썬 소스는 오픈소스로 모두 공개했다.



# 6. 참고자료

