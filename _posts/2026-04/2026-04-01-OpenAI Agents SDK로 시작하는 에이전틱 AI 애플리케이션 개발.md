---
title: "OpenAI Agents SDK로 시작하는 에이전틱 AI 애플리케이션 개발"
date: 2026-05-01
tags: [ChatGPT, OpenAI, AgenticAI, 에이전틱AI, ]
typora-root-url: ../
toc: true
categories: [OpenAI]
---
ChatGPT 등장 이후 대규모 언어 모델(LLM)은 자연어 이해와 생성 능력에서 놀라운 발전을 보여주었다. 하지만 실제 업무 환경에서는 단순히 질문에 답하는 모델을 넘어, 스스로 목표를 이해하고 필요한 도구를 호출하며 여러 작업을 조합하는 **에이전틱(Agentic) AI**가 요구되고 있다.

예를 들어, 기업용 AI 비서는 사용자의 요구사항 분석,  데이터 검색,  외부 API 호출, 문서 작성,  다른 전문 에이전트에게 업무 위임, 결과 검증 및 사용자 승인 요청 등과 같은 작업을 수헹헤야 한다.

이러한 복잡한 워크플로우를 직접 구현하려면 많은 코드와 상태 관리 로직이 필요하다. **OpenAI Agents SDK**는 이러한 문제를 해결하기 위해 등장한 경량 에이전트 개발 프레임워크이다.

#### 1. OpenAI Agents SDK란?

OpenAI Agents SDK는 멀티 에이전트 워크플로우를 구축하기 위한 Python 기반 프레임워크이다. 핵심 철학은 복잡한 추상화를 최소화하면서도 실제 서비스 수준의 에이전트 애플리케이션을 만들 수 있도록 하는 것이다.

SDK는 다음과 같은 핵심 구성 요소를 제공한다.

| 구성 요소         | 역할                                                     |
| ----------------- | -------------------------------------------------------- |
| Agent             | LLM에 Instructions, Tools, Guardrails를 결합한 실행 단위 |
| Tools             | Agent가 외부 함수, MCP, Hosted Tool 등을 호출하는 기능   |
| Handoff           | 하나의 Agent가 다른 전문 Agent에게 작업 전달             |
| Guardrails        | 입력과 출력 검증 및 안전 제어                            |
| Sessions          | 대화 상태 관리                                           |
| Human in the Loop | 필요 시 사람 승인 과정 추가                              |
| Tracing           | Agent 실행 과정 분석 및 디버깅                           |

Agents SDK는 단순한 LLM 호출 라이브러리가 아니라, **LLM 기반 업무 자동화 시스템을 설계하기 위한 실행 프레임워크**이다.

#### 2. 프로젝트 구조 이해하기

이 Github 저장소에서는 OpenAI가 개발한 예제들을 모은 것으로써 OpenAI Agents SDK의 기본 사용 방법을 Python 코드 중심으로 학습할 수 있도록 구성되어 있다. 따라서, 개발자가 가장 먼저 이해해야 할 실행 흐름은 다음과 같다.

```
사용자 요청 -> Agent 선택 -> Reasoning 및 Tool 호출 -> 필요 시 다른 Agent에게 Handoff -> 결과 생성 -> Tracing 및 평가
```

기존 Chat Completion 방식에서는 개발자가 직접 Tool 호출 판단, Function Calling 처리, 대화 상태 저장, 오류 처리, Agent 간 연결 등 로직을 구현해야 했다. 그러나 Agents SDK에서는 이러한 실행 흐름을 Runner와 Agent 객체 기반으로 관리하는 것이 차이점이다.

#### 3. 개발 환경

Agents SDK는 Python 환경에서 실행되며 Python 3.10 이상을 요구한다. 기본 설치 과정은 다음과 같다.

```
python -m venv .venv

source .venv/bin/activate

pip install openai-agents
```

또는 최근 Python 개발자 사이에서 많이 사용하는 uv 환경에서는 다음과 같이 설치할 수 있다.

```
uv init

uv add openai-agents
```

#### 4. 첫 번째 Agent 만들기

가장 기본적인 Agent는 다음 구조를 가진다.

```Python
from agents import Agent, Runner

agent = Agent(
    name="Assistant",
    instructions="You are a helpful assistant."
)

result = Runner.run_sync(
    agent,
    "OpenAI Agents SDK를 설명해줘"
)

print(result.final_output)
```

여기서 중요한 개념은 Agent와 Runner의 역할 분리이다.  Agent는 이름(name),  행동 지침(instructions), 사용할 도구(Tool), 가드레일(Guardrail)과 다른 에이전트 연결 정보 등을 포함한다. 다시 말해, Agent는 "어떤 역할을 수행하는 AI 직원"이라고 생각할 수 있다. 반면, Runner는 에이전트 실행 엔진이다. Runner는 Agent 실행, Tool 호출, Handoff 처리와 결과 반환 등 작업을 관리한다.

#### 5. 멀티 에이전트(Multi-Agent) 구조 설계


```
![llmd-11]({{ '/images/2026-04/openai-agents-01.jpg' | relative_url }})
```

실제 기업 환경에서는 하나의 에이전트가 모든 업무를 처리하지 않는다. 예를 들어, 금융 AI 시스템이라면 다음과 같이 구성할 수 있다. Router Agent는 사용자의 요청을 분석하고 적절한 전문 Agent에게 업무를 전달한다. Agents SDK에서는 Handoff 기능을 이용해 이러한 구조를 구현할 수 있다.

#### 6. Tool Calling: Agent가 실제 세계와 연결되는 방법

LLM은 기본적으로 텍스트 생성 모델이다. 하지만 Agent는 Tool을 통해 실제 작업을 수행한다. Agents SDK에서는 Tools를 이용해 Agent에게 실행 능력을 추가할 수 있다.

```
Agent
|
+-- 날씨 API 호출
|
+-- 데이터베이스 검색
|
+-- Python 함수 실행
|
+-- MCP 서버 연결
```

#### 7. Sandbox Agent: 코드 실행형 에이전트의 등장

최근 Agent 개발에서 중요한 변화는 단순 대화형 Agent에서 작업 수행형 Agent로 이동하고 있다는 점이다. Agents SDK는 Sandbox Agent 개념을 제공한다. Sandbox Agent는 파일 분석, 코드 실행, Repository 탐색, 장시간 작업 상태 유지 등과 같은 작업이 가능하다. 예를 들어, Software Engineering Agent는 다음과 같은 workflow를 수행할 수 있다.

```
Git Repository 분석 -> 코드 이해 - 버그 수정 -> Test 실행 -> Patch 생성
```

#### 7. Tracing: 에이전트 시대의 관측성

전통적인 애플리케이션에서는 로그와 평가 지수(Metric)으로 시스템을 분석한다. 하지만 에이전트 시스템에서는 다음 정보를 추적해야 한다.

* 어떤 Agent가 실행되었는가?
* 어떤 Tool을 호출했는가?
* 어떤 판단 과정을 거쳤는가?
* 어느 단계에서 실패했는가?

Agents SDK는 내장 Tracing 기능을 제공해 Agent 실행 흐름을 확인하고 디버깅할 수 있다. 이는 Production Agent 시스템 운영에서 매우 중요한 기능이다.

#### 8. Tracing: 에이전트 시대의 관측성

LLM 애플리케이션 개발 방식은 빠르게 변화하고 있다.

과거:

```
Prompt - LLM - Response
```

현재:

```
User Request - Agent - Reasoning - Tools - Other Agents - Business Workflow
```

앞으로의 AI 개발자는 단순히 Prompt를 작성하는 사람이 아니라, Agent Architecture 설계, Tool Integration,
Workflow Orchestration, Evaluation, Observability 까지 고려해야 한다.

#### 9. 개발자가 앞으로 학습해야 할 영역

OpenAI Agents SDK를 기반으로 다음 영역을 함께 학습하면 실제 서비스 개발 역량을 키울 수 있다.

1)에이전트 패턴
Router Agent
Planner Agent
Executor Agent
Critic Agent

2)기업 통합
RAG
Vector Database
MCP
API Gateway
Kubernetes Deployment

3)프로덕션 운영
Tracing
Evaluation
Guardrails
Cost Optimization
Security

#### 10. 마무리

OpenAI Agents SDK는 단순히 LLM API를 쉽게 호출하는 라이브러리가 아니다. 이는 LLM을 실제 업무 프로세스에 연결하기 위한 Agent Engineering Framework이다. 앞으로 AI 애플리케이션 개발의 중심은 "더 큰 모델을 호출하는 것"에서 "더 좋은 Agent 시스템을 설계하는 것"으로 이동할 것이다. 개발자는 이제 Prompt Engineering을 넘어 Agent Architecture Engineering을 배워야 한다. 그러므로 OpenAI Agents SDK는 이러한 새로운 AI 개발 패러다임을 학습하기 위한 좋은 출발점이다.
