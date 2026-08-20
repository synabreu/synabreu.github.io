---
title: "[실습] OpenAI 에이전트 도커 워크삽 (3)-agents 분석"
date: 2026-08-16
tags: [오픈AI, OpenAI, GPT-5.6, agenticai, aiagent, openrouter, docker, powershell, fastapi, swagger-ui, visual studio code, Linux, Windows11, Kubernetes, On-Premises ]
typora-root-url: ../
toc: true
categories: [openai]
---
두번째 실습에서는 OpenAI Key 설정하고 간단한 Simple Agent 를 분석하고 만든 다음, Function Tool 작성을 해 보자!

# 1. 실습 2 - OpenAI Key 설정

현재 파워셀 세션에서만 환경변수를 설정한다.

```powershell
$env:OPENAI_API_KEY="YOUR_OPENAI_API_KEY"
$env:OPENAI_DEFAULT_MODEL="gpt-5.6-luna"
```

> 일반 실습에서는 비용을 낮추기 위해 `gpt-5.6-luna`를 기본값으로 사용한다. 필요하면 `gpt-5.6-terra` 또는 `gpt-5.6-sol`로 바꿀 수 있다.

확인할 때 API Key 전체를 출력하지 않는 편이 안전하다.

```powershell
if ($env:OPENAI_API_KEY) { "OPENAI_API_KEY is set" }
```

보안을 위해 `.env`, Dockerfile, Git repository에 실제 API Key를 넣지 않는다.

# 2. 실습 3 - Simple Agent 분석

`app/agents_app.py`의 첫 번째 에어전트는 다음 구조다.

```python
simple_agent = Agent(
    name="Simple Assistant",
    model=MODEL,
    instructions="You are a beginner-friendly assistant...",
)
```

실행은 다음 코드가 담당한다.

```python
result = await Runner.run(AGENTS[agent_name], message)
```

`Runner.run()`은 비동기 환경에서 에이전트(Agent loop)를 수행하고 최종 결과를 반환한다.

# 3. 실습 4 - Function Tool 작성

파이썬 함수를 에어전트가 호출할 수 있는 도구(Tool)로 만든다.

```python
@function_tool
def calculate_total(price: float, quantity: int) -> str:
    total = price * quantity
    return f"total={total:.2f}"
```

그리고 Agent 클래스에 등록한다. `tool_agent` 안에 tools 변수에 `calculate_total' 함수명을 넣는다.

```python
tool_agent = Agent(
    name="Shopping Assistant",
    model=MODEL,
    tools=[calculate_total],
)
```

전체 핵심 흐름은 아래의 예와 같다.

```text
"12,500원 상품 3개 총액"
          ↓
       Agent
          ↓
  calculate_total Tool 호출
          ↓
       37,500
          ↓
      최종 응답
```

# 4. 실습 5 - Handoff / 멀티 에이전트 작성

두 전문 Agent를 만든다.

```text
Deployment Triage Agent
        │
        ├── Public Cloud 질문 → Cloud Specialist
        │
        └── On-Prem 질문     → On-Prem Specialist
```

코드에서는 다음처럼 연결한다.

```python
triage_agent = Agent(
    name="Deployment Triage Agent",
    model=MODEL,
    instructions="...",
    handoffs=[cloud_agent, onprem_agent],
)
```

Handoff는 단순 함수 호출과 다르다. 전문 Agent가 현재 turn의 active agent가 되어 응답을 이어갈 수 있다. 다시 말해, `handoff`는 현재 에이전트가 사용자의 요청을 더 적합한 전문 에이전트에게 넘기고, 그 에이전트가 이후 실행의 주체가 되도록 하는 기능입니다. 위의 설정은 다음과 같이 동작합니다.

* `Runner`가 먼저 `triage_agent`를 실행함.
* `cloud_agent`와 `onprem_agent`로 전환할 수 있는 handoff 도구를 모델에 제공함.
* `triage_agent`는 사용자 요청과 `instructions`를 바탕으로 적절한 에이전트를 선택함
* 선택된 에이전트로 실행 제어권과 대화 문맥이 전달됨
* `Runner`는 새 에이전트를 계속 실행하고 그 결과를 최종 출력으로 반환함

예를 들면:

| 사용자 요청                                 | 예상 동작                                       |
| ------------------------------------------- | ----------------------------------------------- |
| “AWS에 Docker 이미지를 배포하고 싶어요.”  | `cloud_agent`로 handoff                       |
| “사내 서버에 컨테이너를 설치하고 싶어요.” | `onprem_agent`로 handoff                      |
| 요청이 불분명함                             | `triage_agent`가 추가 질문을 하거나 직접 응답 |

흐름으로 표현하면 다음과 같다.

```text
사용자 요청
    ↓
Deployment Triage Agent
    ├─ 클라우드 배포 요청 → cloud_agent
    └─ 사내 서버 배포 요청 → onprem_agent
                              ↓
                     전문 에이전트가 응답
                              ↓
                  RunResult.final_output
```

중요한 점은 `handoffs=[...]`에 등록했다고 해서 두 에이전트가 모두 실행되는 것은 아니라는 것이다. 모델이 요청에 맞는 에이전트를 선택했을 때만 handoff가 발생하며, 발생하지 않으면 `triage_agent`가 직접 응답할 수도 있다.

또한 handoff는 단순히 하위 작업을 부탁하고 결과를 돌려받는 것이 아니라, 대화의 주도권 자체를 대상 에이전트로 넘기는 방식이다. OpenAI 공식 예제에서도 triage 에이전트가 요청 언어를 판별해 적절한 언어 전문 에이전트로 전환한다.

# 5. 전체 소스

| 사용자 요청       | 예상 동작                                                                                                              |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------- |
| __init__.py | 아무런 내용이 없는 데, 이는 오류나 미완성 파일이 아니라 “이 폴더는 Python 패키지이며 별도의 초기화 동작은 없다”는 뜻 |
| agents_app.py     | agent 클래스 관련 파일                                                                                                 |
| main.py           | 파이썬 main 함수 실행                                                                                                  |

## 5.1 agents_app.py 파일 전체 분석

```python
import os # 운영체제의 환경변수를 읽기 위한 Python 표준 라이브러리

# Agent:
# 모델, 지침, 도구, handoff 등을 구성하는 에이전트 클래스
#
# Runner:
# Agent를 실행하고 모델 호출, 도구 실행, handoff 과정을 관리하는 실행기
#
# function_tool:
# 일반 Python 함수를 Agent가 호출할 수 있는 Function Tool로 변환하는 데코레이터
from agents import Agent, Runner, function_tool


# OPENAI_DEFAULT_MODEL 환경변수의 값을 가져옴
# 환경변수가 없으면 기본값으로 "gpt-5.6-luna"를 사용함
MODEL = os.getenv("OPENAI_DEFAULT_MODEL", "gpt-5.6-luna")


# 이 데코레이터(@)는 calculate_total()을 Agent가 호출할 수 있는 Function Tool로 변환
@function_tool
def calculate_total(price: float, quantity: int) -> str:
    """
    상품 가격과 수량을 곱하여 총액을 계산함.

    Args:
        price: 상품 하나의 가격
        quantity: 상품 수량

    Returns:
        계산된 총액을 문자열로 반환
    """

    # 가격과 수량을 곱하여 총액 계산
    total = price * quantity

    # 소수점 둘째 자리까지 표시한 문자열 반환
    return f"total={total:.2f}"


# 일반적인 질문에 답변하는 기본 에이전트 객체
simple_agent = Agent(
    # Agent를 식별하는 이름
    name="simple_assistant",

    # Agent가 사용할 모델
    model=MODEL,

    # Agent의 역할과 응답 방식을 정의하는 지침
    instructions=(
        "You are a beginner-friendly assistant. "
        "Answer clearly and briefly."
    ),
)


# 상품 가격 계산을 지원하는 에이전트 객체
tool_agent = Agent(
    name="shopping_assistant",
    model=MODEL,

    # 가격과 수량의 곱셈이 필요한 경우 calculate_total 도구를 사용하도록 지시함
    instructions=(
        "You help users with simple shopping calculations. "
        "Use the calculate_total tool whenever multiplication "
        "of price and quantity is needed."
    ),

    # 이 Agent가 사용할 수 있는 Function Tool 목록
    tools=[calculate_total],
)


# AWS, Azure, GCP 등 퍼블릭 클라우드 배포를 담당하는 전문 에이전트
cloud_agent = Agent(
    name="cloud_specialist",
    model=MODEL,
    instructions=(
        "You are a cloud deployment specialist. "
        "Explain container deployment concepts for AWS, Azure, "
        "and GCP in beginner-friendly Korean."
    ),
)


# 사내 서버와 데이터센터 배포를 담당하는 전문 에이전트
onprem_agent = Agent(
    name="on_prem_specialist",
    model=MODEL,
    instructions=(
        "You are an on-premises container deployment specialist. "
        "Explain Docker and Kubernetes deployment concepts "
        "in beginner-friendly Korean."
    ),
)


# 사용자의 배포 질문을 분류하는 Triage 에이전트
triage_agent = Agent(
    name="deployment_triage_agent",
    model=MODEL,

    # 사용자 요청을 분석하여 적절한 전문 Agent를 선택하도록 지시함
    instructions=(
        "Classify the user's deployment question. "
        "If it is mainly about a public cloud, "
        "hand off to Cloud Specialist. "
        "If it is mainly about an on-premises environment, "
        "hand off to On-Prem Specialist. "
        "If neither is needed, answer directly."
    ),

    # Triage Agent가 실행 제어권을 넘길 수 있는 Agent 목록
    #
    # 클라우드 질문이면 cloud_agent,
    # 사내 환경 질문이면 onprem_agent로 handoff할 수 있음
    handoffs=[cloud_agent, onprem_agent],
)


# 외부에서 전달받은 문자열 이름을 실제 Agent 객체와 연결하는 딕셔너리
#
# 예:
# "simple" → simple_agent
# "tool"   → tool_agent
# "triage" → triage_agent
AGENTS = {
    "simple": simple_agent,
    "tool": tool_agent,
    "triage": triage_agent,
}


# 선택된 Agent를 비동기로 실행하는 애플리케이션 함수
async def run_agent(agent_name: str, message: str) -> dict:
    """
    Agent 이름과 사용자 메시지를 받아 해당 Agent를 실행합니다.

    Args:
        agent_name: 실행할 Agent 이름
        message: 사용자가 입력한 메시지

    Returns:
        최초 Agent 이름, 마지막으로 응답한 Agent 이름,
        최종 응답을 포함한 딕셔너리
    """

    # 등록되지 않은 Agent 이름이 전달되면 실행을 중단합니다.
    if agent_name not in AGENTS:
        raise ValueError(f"Unknown agent: {agent_name}")

    # 선택된 Agent와 사용자 메시지를 Runner에 전달함
    #
    # Runner는 다음 과정을 관리함
    # 1. 모델 호출
    # 2. Function Tool 실행
    # 3. Agent handoff
    # 4. 최종 응답 생성
    result = await Runner.run(
        AGENTS[agent_name],
        message,
    )

    # API에서 사용하기 편한 딕셔너리 형식으로 결과를 반환함
    return {
        # 사용자가 처음 선택한 에이전트
        "agent": agent_name,

        # handoff를 포함한 전체 실행에서 마지막으로 동작한 에이전트
        "last_agent": (
            result.last_agent.name
            if result.last_agent
            else None
        ),

        # 사용자에게 전달할 최종 응답
        "response": result.final_output,
    }
```

# 6. 참고 자료

[OpenAI Developer Quickstart](https://platform.openai.com/docs/quickstart/make-your-first-api-request)
