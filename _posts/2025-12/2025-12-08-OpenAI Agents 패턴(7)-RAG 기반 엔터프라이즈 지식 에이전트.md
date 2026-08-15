---
title: "OpenAI Agents 패턴(7)-RAG 기반 엔터프라이즈 지식 에이전트"
date: 2025-12-08
tags: [오픈AI, OpenAI, Open Agents SDK, Swarm, Visual Studio Code, Multi-Agents, 멀티 에이전트]
typora-root-url: ../
toc: true
categories: [OpenAI]
---

이 예제는 OpenAI Agents SDK를 사용하여 회사 문서를 검색하고 답변하는 지식 기반 AI Agent(RAG Agent)를 만드는 기본 예제를 만들어 보았다. RAG(Retrieval Augmented Generation)는 LLM이 외부 지식을 검색하여 답변 생성에 활용하는 방식이다. RAG Knowledge Agent는 LLM과 기업 내부 지식을 연결하여 신뢰 가능한 답변을 제공하는 가장 기본적인 Enterprise Agent 패턴이다.

구조:

    사용자 질문
        ↓
    Knowledge Agent
        ↓
    검색 Tool
        ↓
    문서 데이터
        ↓
    답변 생성

---

# 1. 개발 환경

필요 환경:

- Windows 11
- Visual Studio Code
- Python 3.11 이상
- OpenAI API Key

프로젝트 구조:

    rag-agent-example/

    ├── app.py
    ├── knowledge/
    │   └── company_policy.txt
    ├── requirements.txt
    └── .env

---

# 2. 패키지 설치

requirements.txt 파일

```txt
openai-agents
python-dotenv
```

설치 방법:

```bash
pip install -r requirements.txt
```

---

# 3. 문서 데이터 준비

knowledge/company_policy.txt

```text
회사 휴가는 연간 20일이다.
GPU 서버 사용 신청은 AI Infra 팀 승인이 필요하다.
긴급 장애 발생 시 IT 운영팀에 연락한다.
```

---

## 4. Agent 구현

app.py 파일 내용

```python
from agents import Agent, Runner, function_tool


@function_tool
def search_company_document(question: str) -> str:
    """회사 문서를 검색하는 Tool"""

    with open(
        "knowledge/company_policy.txt",
        "r",
        encoding="utf-8"
    ) as f:
        document = f.read()

    return document


knowledge_agent = Agent(
    name="Knowledge Agent",
    instructions="""
    당신은 회사 지식 검색 Agent이다.

    사용자의 질문과 관련된 정보를
    search_company_document Tool에서 검색한 후
    답변한다.

    문서에 없는 내용은 추측하지 않는다.
    """,
    tools=[
        search_company_document
    ]
)


result = Runner.run_sync(
    knowledge_agent,
    "GPU 서버 사용 절차를 알려줘"
)

print(result.final_output)
```

---

# 5. 예제 실행

Visual Studio Code 터미널:

```bash
python app.py
```

예상 결과:

    GPU 서버 사용 신청은 AI Infra 팀 승인이 필요합니다.

---

# 6. 구성 요소 설명

* Agent : 사용자의 요청을 이해하고 Tool 사용 여부를 결정한다.
* Tool : 외부 데이터를 가져오는 함수이다.
* Runner : Agent 실행 흐름을 관리한다.

---

# 7. 기존 Agent 패턴과 차이

```
기본 Tool Agent:

    User
     ↓
    Agent
     ↓
    Tool
     ↓
    Answer

RAG Knowledge Agent:

    User
     ↓
    Knowledge Agent
     ↓
    Search Tool
     ↓
    Company Document
     ↓
    Answer
```

---

# 8. Enterprise 활용 사례

* 고객지원 에이전트 : 제품 매뉴얼 검색 후 고객 답변 생성
* IT 운영 에이전트 : Runbook 검색 후 장애 대응 방법 안내
* AI Infrastructure Agent : GPU 운영 정책과 사용 절차 검색

---

# 9. 다음 확장 단계

- Vector Database 연결
- Embedding 검색
- OpenAI File Search 활용
- Multi-Agent Router 추가
- MCP Tool 연결
