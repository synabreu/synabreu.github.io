---
title: "[실습] OpenAI 에이전트 도커 워크삽 (4)-main 소스분석"
date: 2026-08-16
tags: [오픈AI, OpenAI, GPT-5.6, agenticai, aiagent, openrouter, docker, powershell, fastapi, swagger-ui, visual studio code, Linux, Windows11, Kubernetes, On-Premises ]
typora-root-url: ../
toc: true
categories: [openai]
---
이 블로그는 main.py 파일 소스를 분석하고 내부의 get 과 post 데코레이터 함수를 어떻게 사용하는 지 분석한다. 

# 1. main.py 파일 소스 분석

```python
# FastAPI는 웹 API 애플리케이션을 생성하는 클래스
# HTTPException은 클라이언트에 HTTP 오류 응답을 반환하는 예외 클래스
from fastapi import FastAPI, HTTPException

# BaseModel은 요청과 응답 데이터의 구조를 정의하는 Pydantic 기본 클래스
# Field는 필드의 유효성 검사 조건과 문서 예시를 설정하는 함수
from pydantic import BaseModel, Field

# 지정한 Agent를 실행하고 결과를 반환하는 함수를 가져온다.
from app.agents_app import run_agent


# FastAPI 애플리케이션 객체를 생성하고, 설정한 정보는 Swagger UI와 OpenAPI 문서에 표시된다.
app = FastAPI(
    # API 문서에 표시되는 애플리케이션 이름
    title="OpenAI Agents SDK Docker Workshop",

    # API 버전 정보
    version="1.0.0",

    # 애플리케이션의 목적과 실행 환경
    description=(
        "Windows Native development -> "
        "Linux Docker image hands-on workshop"
    ),
)


# Agent API가 전달받을 요청 본문의 데이터 구조를 정의함
class ChatRequest(BaseModel):
    # 사용자가 Agent에게 보낼 메시지이며, 최소 한 글자 이상 입력하도록 검사한다.
    # examples는 Swagger UI에 표시할 요청 예시이다.
    message: str = Field(
        min_length=1,
        examples=["안녕하세요. 이 앱을 설명해줘."],
    )


# HTTP GET 방식의 루트 경로를 등록함
@app.get("/")
async def root():
    """
    애플리케이션의 기본 정보와 주요 API 경로를 반환한다.
    """

    return {
        # 애플리케이션 이름 반환
        "name": "OpenAI Agents SDK Docker Workshop",

        # Swagger API 문서의 경로 반환
        "docs": "/docs",

        # 서버 상태 확인 경로 반환
        "health": "/health",

        # 실행할 수 있는 에이전트 이름 반환
        "agents": ["simple", "tool", "triage"],
    }


# HTTP GET 방식의 서버 상태 확인 경로를 등록
@app.get("/health")
async def health():
    """
    애플리케이션이 정상적으로 실행 중인지 확인한다.
    """

    return {"status": "ok"}


# HTTP GET 방식으로 특정 에이전트를 실행하는 경로를 등록함
# {agent_name}에는 simple, tool, triage 등의 Agent 이름이 들어간다.
@app.get("/agents/{agent_name}")
async def chat(agent_name: str, request: ChatRequest):
    """
    URL에서 Agent 이름을 받고 요청 본문에서 메시지를 받아 Agent를 실행한다.
    """

    try:
        # 지정된 Agent를 비동기로 실행하고 결과를 반환함
        return await run_agent(
            agent_name,
            request.message,
        )

    except ValueError as exc:
        # 존재하지 않는 Agent 이름이 전달되면 404 오류를 반환함
        raise HTTPException(
            status_code=404,
            detail=str(exc),
        ) from exc

    except Exception as exc:
        # Agent 실행 중 예상하지 못한 오류가 발생하면 500 오류를 반환함
        raise HTTPException(
            status_code=500,
            detail=f"Agent execution failed: {exc}",
        ) from exc


# HTTP POST 방식으로 특정 Agent를 실행하는 경로를 등록한다.
@app.post("/agents/{agent_name}")
async def chat(agent_name: str, request: ChatRequest):
    """
    URL에서 Agent 이름을 받고 JSON 요청 본문에서 메시지를 받아 Agent를 실행한다.
    """

    try:
        # 지정된 Agent를 비동기로 실행하고 결과를 반환함
        return await run_agent(
            agent_name,
            request.message,
        )

    except ValueError as exc:
        # 존재하지 않는 Agent 이름이 전달되면 404 오류를 반환함
        raise HTTPException(
            status_code=404,
            detail=str(exc),
        ) from exc

    except Exception as exc:
        # Agent 실행 중 예상하지 못한 오류가 발생하면 500 오류를 반환함
        raise HTTPException(
            status_code=500,
            detail=f"Agent execution failed: {exc}",
        ) from exc
```

# 2. get/post 함수

GET과 POST 함수 이름이 모두 chat으로 동일하다. FastAPI 경로는 데코레이터가 등록하므로 실행될 수 있지만, 코드 가독성과 API 문서의 작업 식별을 위해 다음처럼 서로 다른 이름을 사용하는 것이 좋다.

```python
@app.get("/agents/{agent_name}")
async def chat_get(agent_name: str, request: ChatRequest):
    ...


@app.post("/agents/{agent_name}")
async def chat_post(agent_name: str, request: ChatRequest):
    ...
```

또한 요청 본문을 전달하는 API는 일반적으로 GET보다 POST 방식을 사용한다. 따라서 실제 서비스에서는 POST 경로만 제공하는 구성이 더 자연스럽다.
