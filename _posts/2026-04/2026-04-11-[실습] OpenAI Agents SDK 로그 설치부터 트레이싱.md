---
title: "[실습] OpenAI Agents SDK 로그 설치부터 트레이싱"
date: 2026-04-11
tags: [openai, devday, samaltman, chatgpt, codex, codexmobile, agent, agenticai, trace, debugging]
typora-root-url: ../
toc: true
categories: [openai]
---

이번 블로그에는 agents sdk 상에서 어떻게 로깅하고 트레이싱하는 지 보여주는 예제이다. `agents-sdk-trace-demo` 프로젝트는 주문 상담과 답변 검토 과정을 OpenAI Agents SDK의 워크플로 trace로 기록하는 Python 실습 프로젝트다. 

다시 말해 Agents SDK로 두 에이전트를 실행하고 OpenAI의 Developer Site에서 Logs 그룹을 눌러 Agents SDK탭에서 하나의 워크플로 trace를 확인하는 과정이다. 

# 1. 프로젝트 아키텍처

![그림 1. 아키텍처 플로]({{ '/images/2026-04/agents-sdk-01.png' | relative_url }}){: style="width: 70%; height: auto;"}
<br>*[그림 1. 아키텍처 플로]*

아키텍처 플로우에 대해 설명하자면 다음과 같다. 사용자의 질문부터 도구 호출, 답변 생성, 검토까지 이어지는 과정을 Agents SDK 탭에서 확인할 수 있다. 주문 정보는 코드에 정의한 가상 데이터를 사용한다. 사용자가 “DEMO-1001 주문의 배송 상태를 알려줘”라고 질문하면 상담 에이전트가 `lookup_order` 도구를 호출한다. 도구는 해당 주문의 배송 상태와 예상 도착 정보를 반환하고, 상담 에이전트는 이를 바탕으로 답변을 작성한다. 이어서 검토 에이전트가 상담 답변과 주문 정보를 비교해 검토 결과를 출력한다.

이 과정에서 모델에 전달한 입력과 출력, 도구 호출에 사용한 인자와 반환 결과, 각 단계의 실행 시간과 기록된 오류를 확인할 수 있다. 입력을 준비하는 과정도 사용자 정의 단계인 custom span으로 남긴다. 두 에이전트의 실행은 하나의 `trace()` 안에 묶여 있어 전체 작업 흐름을 함께 살펴볼 수 있다.

로깅은 개별 단계에서 발생한 일을 기록하는 것이고, 트레이싱은 그 기록을 연결해 전체 실행 흐름을 보여주는 것이다. 이 프로젝트에서는 어떤 주문을 조회했는지뿐 아니라, 조회 결과가 상담 답변으로 이어지고 다시 검토되는 과정까지 추적한다.

터미널에 표시되는 답변과 Trace ID는 `print()`로 출력한 내용이다. OpenAI Platform의 Agents SDK 탭에 나타나는 실행 기록은 SDK의 tracing 기능이 전송한다. 따라서 대시보드 기록은 콘솔 출력을 그대로 저장한 것이 아니라, 실행 단계를 구조화한 기록이다.

검토 에이전트는 상담 답변을 자동으로 수정하지 않는다. 상담 답변과 근거가 일치하는지 확인하고 검토 의견을 출력한다. 이 예제는 두 에이전트를 순서대로 실행하면서, 각 단계가 하나의 워크플로에 어떻게 기록되는지 살펴보는 데 목적이 있다.

# 2. main.py 소스 분석

```python
import argparse
import asyncio
import os
import sys
from pathlib import Path
from uuid import uuid4

from dotenv import load_dotenv

# 실행 위치와 관계없이 소스 파일 옆의 .env를 찾기 위한 기준 폴더
ROOT = Path(__file__).resolve().parent

# trace 확인 주소와 대시보드에 표시할 워크플로 명
DASHBOARD = "https://platform.openai.com/logs?api=traces"
WORKFLOW = "Python 주문 상담 워크플로"


def settings() -> tuple[str, str]:
    # API 키와 모델 설정을 검사하고 (키, 모델) 튜플을 반환하고 
    # 이미 설정된 셸 환경 변수는 로컬 .env 값보다 우선함 
    load_dotenv(ROOT / ".env", override=False)

    # 설정 값의 앞뒤 공백을 제거하고 모델이 설정되지 않았으면 기본값을 사용함 
    key = os.getenv("OPENAI_API_KEY", "").strip()
    model = os.getenv("OPENAI_MODEL", "gpt-4.1-mini").strip()

    # 빈 키, 예제용 키, 빈 모델, tracing 비활성화 설정은 실행 전에 거부함 
    if not key or key == "your_api_key_here":
        raise ValueError("기존 API 키를 .env의 OPENAI_API_KEY 또는 환경 변수에 설정하세요.")
    if not model:
        raise ValueError("OPENAI_MODEL이 비어 있습니다.")
    if os.getenv("OPENAI_AGENTS_DISABLE_TRACING", "").lower() in {"1", "true"}:
        raise ValueError("trace 실습을 위해 OPENAI_AGENTS_DISABLE_TRACING 설정을 해제하세요.")
    return key, model


async def workflow(key: str, model: str, prompt: str) -> None:
    # 상담 → 답변 검토를 비동기로 실행하고 trace 내보내기를 요청함 
    # 실제 실행에 필요한 SDK는 여기서 불러와 --check에서는 사용하지 않음 
    from openai import AsyncOpenAI
    from agents import (
        Agent, Runner, RunConfig, custom_span, function_tool,
        gen_trace_id, set_default_openai_client,
        set_tracing_export_api_key, trace,
    )
    from agents.tracing import get_trace_provider

    @function_tool
    def lookup_order(order_id: str) -> str:
        """Look up a demo order by order ID, such as DEMO-1001."""
        # 에이전트가 호출할 함수 도구입니다. 외부 시스템 없이 가상 주문을 조회함 
        # 주문 ID는 대소문자를 구분하지 않고 비교함 
        if order_id.upper() == "DEMO-1001":
            return "DEMO-1001: 배송 중. 예상 도착: 주문 후 3영업일. 상품: 데모 키보드."
        return "데모 데이터에 해당 주문이 없습니다. 사용 가능한 주문: DEMO-1001."

    # 실행마다 고유한 trace ID와 그룹 ID를 생성함 
    trace_id = gen_trace_id()
    run_group = f"demo_{uuid4().hex}"

    # 요청 제한 시간은 60초이며 자동 재시도는 하지 않음 
    client = AsyncOpenAI(api_key=key, timeout=60.0, max_retries=0)

    # 에이전트의 모델 요청과 trace 전송에 사용할 클라이언트 및 키를 설정함 
    set_default_openai_client(client)
    set_tracing_export_api_key(key)
    
    # trace를 활성화하고 모델·도구의 입력/출력을 포함하고 실습에는 샘플 질문과 가상 주문 데이터를 사용함
    config = RunConfig(tracing_disabled=False, trace_include_sensitive_data=True)

    # 상담 에이전트는 주문 조회 도구를 사용하고 도구가 반환한 사실로 답함 
    support = Agent(
        name="주문 상담 에이전트", model=model, tools=[lookup_order],
        instructions=("한국어로 짧게 답하세요. 주문 상태 질문에는 반드시 lookup_order 도구를 "
                      "사용하세요. 도구가 반환한 데모 데이터만 사실로 사용하세요."),
    )

    # 검토 에이전트는 다음 단계에서 상담 답변과 데모 근거를 비교함 
    reviewer = Agent(
        name="답변 검토 에이전트", model=model,
        instructions="상담 답변이 제공된 근거와 일치하는지 검토하고 한국어로 짧게 결과를 쓰세요.",
    )

    # 대시보드에서 이번 실행을 찾을 수 있도록 이름과 trace ID를 먼저 출력함 
    print(f"워크플로: {WORKFLOW}\nTrace ID: {trace_id}\nLogs: {DASHBOARD}")
    try:
        # 바깥 trace 컨텍스트가 입력 준비와 두 에이전트 실행을 하나로 묶음 
        with trace(WORKFLOW, trace_id=trace_id, group_id=run_group,
                   metadata={"demo": "agents-sdk-trace", "language": "python"}):
            
            # 입력 정리 과정을 별도 span으로 기록하고 빈 질문을 검사함 
            with custom_span("입력 준비", data={"demo_order_id": "DEMO-1001"}):
                prepared = prompt.strip()
                if not prepared:
                    raise ValueError("질문을 입력하세요.")
                
            # 먼저 상담을 실행하고 모델과 도구 호출을 포함해 최대 5턴을 허용함 
            answer = await Runner.run(support, prepared, run_config=config, max_turns=5)

            # 상담 결과와 고정된 데모 근거를 검토 에이전트에 전달함(최대 3턴).
            review = await Runner.run(
                reviewer,
                f"사용자 질문: {prepared}\n상담 답변: {answer.final_output}\n"
                "데모 근거: DEMO-1001은 배송 중, 예상 도착은 주문 후 3영업일, 상품은 데모 키보드.",
                run_config=config, max_turns=3,
            )

            # 두 에이전트의 최종 출력을 콘솔에 표시함 
            print(f"\n상담 답변:\n{answer.final_output}\n\n검토 결과:\n{review.final_output}")
    finally:
        # 에이전트 실행이 실패해도 trace 컨텍스트 종료 후 내보내기를 요청함 
        # 동기 flush는 별도 스레드에서 실행해 이벤트 루프를 막지 않음 
        # 내보내기 요청은 대시보드 수신 성공을 보장하지 않음 
        await asyncio.to_thread(get_trace_provider().force_flush)

        # 비동기 API 클라이언트의 연결 자원을 정리함 
        await client.close()
        print("\ntrace 내보내기를 요청했습니다. 같은 API 프로젝트의 Agents SDK 탭에서 확인하세요.")


def main() -> int:
    # 명령줄 처리, 설정 검사, 비동기 워크플로 실행을 담당하는 진입 함수
    # 정상 종료는 0, 설정 또는 실행 오류는 1을 반환함 

    # --check는 설정만 확인하고, --prompt는 상담 질문을 지정함 
    parser = argparse.ArgumentParser(description="Agents SDK 워크플로 trace 실습")
    parser.add_argument("--check", action="store_true", help="API 호출 없이 설정 확인")
    parser.add_argument("--prompt", default="DEMO-1001 주문의 배송 상태를 알려줘.")
    args = parser.parse_args()
    try:
        # API 요청 전에 키·모델·tracing 설정을 확인함 
        key, model = settings()
        if args.check:
            # 키 값은 숨기며 API 호출이나 인증·권한 검증 없이 종료함 
            print(f"API 키: 설정됨 (값 숨김)\n모델: {model}\nLogs: {DASHBOARD}")
            print("설정만 확인했습니다. 키 유효성 및 API 권한은 확인하지 않았습니다.")
            return 0
        # 이벤트 루프를 만들어 비동기 워크플로가 끝날 때까지 실행함 
        asyncio.run(workflow(key, model, args.prompt))
        return 0
    except Exception as exc:
        # 예외 유형과 안내를 표준 오류로 출력하고 원본 API 예외 메시지, 헤더, 인증 정보는 출력하지 않음 
        print(f"실행 실패: {type(exc).__name__}. .env, API 권한, 네트워크와 Logs를 확인하세요.", file=sys.stderr)
        if isinstance(exc, ValueError):
            # 설정·입력 검사에서 발생한 ValueError는 구체적인 메시지도 안내함 
            print(str(exc), file=sys.stderr)
        return 1


if __name__ == "__main__":
    # 직접 실행할 때만 main을 호출하고 반환값을 프로세스 종료 코드로 사용함 
    raise SystemExit(main())
```

# 3. 프로젝트 파일 설치

[agents sdk trace demo](https://github.com/synabreu/agents-sdk-trace-demo)에 공개적으로 올려 놓은 예제 소스를 다음과 같이 복사한다.

```powersehll
git clone https://github.com/synabreu/agents-sdk-trace-demo.git
```

agents-sdk-trace-demo 프로젝트 폴더에서 PowerShell을 열고 실행한다. 

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

# 4. OpenAI Key 설정

```powershell
Copy-Item .env.example .env
notepad .env
```

여러분의 OpenAI API 키를 your_api_key_here에 넣어라. 

```dotenv
OPENAI_API_KEY=your_api_key_here
OPENAI_MODEL=gpt-4.1-mini
```

셸에 OPENAI_API_KEY가 설정되어 있으면 해당 값이 .env보다 우선한다. .env는 main.py와 같은 폴더에서 읽는다. 모델은 프로젝트에서 사용 가능한 모델로 변경할 수 있다. 실제 실행에는 API 사용 요금이 발생한다. 

# 5. 설정 확인 — API 호출 없음

```powershell
.\.venv\Scripts\python.exe main.py --check
```

키 값을 출력하지 않는다. 참고로 키의 유효성, 결제 상태, 모델 권한, trace 전송 성공을 검증하는 명령은 아니다.

![그림 2. main 파일 체크]({{ '/images/2026-04/agents-sdk-02.png' | relative_url }}){: style="width: 70%; height: auto;"}
<br>*[그림 2. main 파일 체크]*

# 6. 워크플로 실행

```powershell
.\.venv\Scripts\python.exe main.py
# 다른 질문
.\.venv\Scripts\python.exe main.py --prompt "DEMO-1001은 언제 도착하나요?"
```

![그림 3. main 파일 체크]({{ '/images/2026-04/agents-sdk-03.png' | relative_url }}){: style="width: 70%; height: auto;"}
<br>*[그림 2. 프롬프트 실행]*


기본 질문에서 기대하는 trace 구조는 다음과 같다. 모델 판단에 따라 모델 호출 수는 달라질 수 있다.

```text
Python 주문 상담 워크플로 (trace_...)
├── 입력 준비 (custom span)
├── 주문 상담 에이전트
│   ├── 모델 호출
│   ├── lookup_order (function tool)
│   └── 모델 호출 / 상담 답변
└── 답변 검토 에이전트
    └── 모델 호출 / 검토 결과
```

주문 정보는 코드에 정의한 가상 데이터이다. 외부 주문 시스템에는 연결하지 않았다. 두 Runner.run()을 바깥쪽 trace()로 묶으므로 하나의 워크플로로 기록한다. 검토(review)는 명시적인 순차 실행이며 handoff 예제는 아니다. 

# 7. Agents SDK 탭 확인

1) [Platform Logs](https://platform.openai.com/logs?api=traces)를 연다. 
2) 기존 API 키가 속한 조직·프로젝트를 선택한다.
3) **Agents SDK** 탭에서 `Python 주문 상담 워크플로` 또는 터미널의 `trace_...` ID를 찾아본다. 
4) trace를 열고 모델 입력·출력, 도구 호출·결과, 실행 시간과 오류를 확인한다. 

SDK가 trace를 생성하고 전송한다. `print()`는 콘솔 출력용이며 `store=True`로 Responses를 저장하는 것과는 별도의 tracing 경로이다. 종료 시 trace 컨텍스트를 닫은 뒤 force_flush()로 내보내기를 요청한다. 이 메시지만으로 대시보드 저장 성공이 확인되는 것은 아니기 떄문에 전송 오류는 SDK 로그를 확인하라. 

![그림 4. Agents 로그]({{ '/images/2026-04/agents-sdk-04.png' | relative_url }}){: style="width: 70%; height: auto;"}
<br>*[그림 4. Agents 로그]*

이 실습은 입출력 내용을 trace에 포함한다(`trace_include_sensitive_data=True`). 샘플 질문과 가상 주문 데이터로 실습하라. 이를 False로 바꾸면 해당 모델·도구 입력/출력 기록을 줄일 수 있으나 커스텀 스팬(custom span)에 직접 넣은 데이터까지 자동 제거하지는 않는다. 

# 8. Agents SDK 탭 확인

OpenAI Platform에서 Agents SDK 실행 기록을 열면 오른쪽 위에 Evaluate 버튼이 표시된다. 

## 8.1 Evaluate 

Evaluate 버튼은 에이전트의 실행 기록인 trace를 대상으로 실행 과정과 결과가 평가 기준을 충족했는지 확인하는 기능이다. Trace에는 최종 답변뿐 아니라 모델 호출, 도구 호출, 각 단계의 입력과 출력 등이 기록된다. 이를 활용하면 답변이 정확했는지와 함께, 그 답변을 만들기 위해 적절한 도구와 절차를 사용했는지도 평가할 수 있다.

![그림 5. Agents 로그 상세]({{ '/images/2026-04/agents-sdk-05.png' | relative_url }}){: style="width: 70%; height: auto;"}
<br>*[그림 5. Agents 로그 상세]*

참고로 Evaluate는 trace 평가를 설정하고 실행하는 흐름으로 연결되는 진입점이고 평가 절차는 다음과 같다.

1) 평가할 trace를 선택한다.
2) 평가 기준을 담은 **grader**를 만들거나 선택한다.
3) 선택한 trace에 대해 평가를 실행한다.
4) 점수나 통과·실패 등의 결과를 확인한다.

채점기(grader)는 실행 기록을 기준에 따라 평가하고 결과를 구조화된 점수나 라벨로 남긴다. 개발자는 이 결과와 trace를 함께 살펴보며 프롬프트, 도구 정의, 실행 흐름 등을 개선할 수 있다.

## 8.2 Grade all

`Grade all`버튼은 여러 개의 실행 기록을 한 번에 평가하는 기능이다. 개별 trace를 하나씩 열어 결과를 확인하는 대신, 현재 선택된 trace 전체에 동일한 grader 기준을 적용해 일괄적으로 검증한다. 예를 들어 주문 상담 에이전트라면 사용자가 전달한 주문번호를 올바르게 사용했는지, `lookup_order` 같은 필수 도구를 정상적으로 호출했는지, 조회된 결과와 최종 답변이 일치하는지 등을 자동으로 확인할 수 있다.

![그림 6. Grade all 클릭]({{ '/images/2026-04/agents-sdk-06.png' | relative_url }}){: style="width: 70%; height: auto;"}
<br>*[그림 6. Grade all 클릭]*

평가 기준은 사용자가 직접 정의할 수 있다. 단순히 최종 답변이 맞는지만 확인하는 것이 아니라 모델 호출, 도구 사용, 에이전트 간 작업 흐름, 응답 근거 등 trace에 기록된 전체 실행 과정을 대상으로 평가할 수 있다. 각 trace에는 Pass 또는 Fail과 같은 결과가 부여되며, 실패한 경우에는 어떤 단계에서 문제가 발생했는지 추적할 수 있다.

`Grade all`의 장점은 반복적인 수작업 검증을 줄일 수 있다는 점이다. 테스트해야 할 trace가 몇 개일 때는 사람이 직접 확인할 수 있지만 수십 개, 수백 개로 늘어나면 같은 기준으로 일관되게 평가하기 어렵다. 이때 grader를 이용하면 동일한 평가 기준을 전체 trace에 적용할 수 있고, 문제가 있는 실행만 골라서 분석할 수 있다. 또한 `Grade all`은 에이전트 워크플로를 다시 실행하는 기능이 아니다. 이미 생성된 trace를 대상으로 평가를 수행한다. 따라서 실제 시스템 실행과 평가 과정을 분리할 수 있으며, 에이전트의 동작 품질을 반복적으로 측정하고 개선하는 데 활용할 수 있다.

실제 사용에서는 먼저 몇 개의 대표 trace에 grader를 실행해 평가 기준이 의도대로 동작하는지 확인한 뒤, `Grade all`을 이용해 전체 trace를 평가하는 방식이 적절하다. 이를 통해 AI 에이전트의 품질 평가를 사람의 주관적인 확인 과정에서 벗어나 보다 체계적이고 반복 가능한 평가 과정으로 만들 수 있다.

## 8.3 더 보기 메뉴(More Options Menu)

더 보기 메뉴(ellipsis menu)를 누르면 다음 그림과 같이 나타난다. 

![그림 7. 더 보기 메뉴 클릭]({{ '/images/2026-04/agents-sdk-07.png' | relative_url }}){: style="width: 70%; height: auto;"}
<br>*[그림 7. 더 보기 메뉴 클릭]*

그러한 특징들에 대해 정리하자면 다음과 같다. 

| 메뉴 | 의미 | 이 화면에서의 역할 |
|---|---|---|
| **Inspect results** | 결과 상세 검사 | 해당 Eval Run의 개별 trace별 평가 결과를 자세히 확인한다. 어떤 trace가 Pass/Fail인지, grader가 어떤 판단을 했는지, 점수와 평가 근거 등을 살펴볼 때 사용한다. |
| **View data source** | 데이터 소스 보기 | 이 평가에 사용된 원본 데이터 집합을 확인한다. Trace Eval이라면 평가 대상으로 들어간 trace들이 무엇인지 확인하는 용도다. |
| **Rename** | 이름 변경 | 해당 Eval Run의 이름을 알아보기 쉽게 변경한다. 이름만 바뀌며 기존 평가 결과나 점수에는 영향을 주지 않는다. |
| **Delete** | 삭제 | 해당 Eval Run과 그 결과 기록을 삭제한다. 일반적으로 평가에 사용했던 원본 trace 자체를 삭제하는 것과는 별개의 작업이다. |

특히 Inspect results가 가장 자주 사용하는 메뉴다. 화면에 표시된 66%라는 전체 점수만으로는 어떤 trace가 실패했는지 알기 어렵기 때문이다. Inspect results를 선택하면 개별 평가 결과로 들어가 실패한 trace와 grader의 판단 내용을 확인할 수 있다. 흐름으로 보면 다음과 같다.

```
Grade all
   ↓
Eval Run 생성
   ↓
전체 점수 66%
   ↓
… More options
   ↓
Inspect results
   ↓
각 Trace별 Pass / Fail과 평가 이유 확인
```

예를 들어, 3 개의 주문 상담 trace를 평가해서 두 개가 성공하고 하나가 실패했다면 전체 결과가 약 **66%**가 될 수 있다. 이때 Inspect results에서 실패한 하나를 찾아 주문번호를 잘못 사용했는지, lookup_order를 호출하지 않았는지, 도구의 반환값과 답변이 일치하지 않았는지 등을 확인한다.

## 8.4 Inspect results

![그림 8. Inspect results]({{ '/images/2026-04/agents-sdk-08.png' | relative_url }}){: style="width: 70%; height: auto;"}
<br>*[그림 8. Inspect results]*

[그림 8]의 `Inspect results`는 Eval Run에 포함된 개별 Trace의 평가 결과를 자세히 확인하는 화면이다. Report 화면이 전체 성공률을 보여준다면, Inspect results는 어떤 Trace가 `Pass`이고 어떤 Trace가 `Fail`인지 구체적으로 보여준다. 화면에는 각 실행의 `trace_id`, 생성 시점인 `created_at`, 부가 정보인 `metadata`, 워크플로 실행 내용인 `output`, 그리고 최종 평가 결과인 `Pass/Fail`이 표시된다. 예제에서는 3개의 Trace 중 2개가 Pass, 1개가 Fail이므로 전체 성공률이 67%로 나타난다.

가장 중요한 용도는 실패한 Trace의 원인을 찾는 것이다. Fail로 표시된 항목을 열어 주문번호가 올바르게 전달됐는지, 필요한 도구를 정상적으로 호출했는지, 도구의 반환값과 최종 답변이 일치하는지 등을 실행 단계별로 확인할 수 있다. 다시 말해, `Inspect results`는 **전체 평가 점수에서 한 단계 더 들어가 실패한 실행을 찾아 원인을 분석하는 화면**이다.

# 9. 문제 해결

| 증상 | 확인 사항 |
|---|---|
| 모듈을 찾을 수 없음 | 가상환경 Python으로 requirements.txt 설치 |
| 키 설정 오류 | .env.txt가 아닌 .env인지, 샘플 값을 바꿨는지 확인 |
| 인증·한도·모델 오류 | 키 유효성, API 결제, 프로젝트 모델 권한 확인 |
| trace가 나타나지 않음 | Agents SDK 탭, 조직·프로젝트, 날짜 필터, 전송 지연, 네트워크 확인 |
| tracing 비활성화 | OPENAI_AGENTS_DISABLE_TRACING 설정 해제 |
| 조직의 데이터 정책 제한 | Zero Data Retention 등 조직 정책에 따라 tracing을 사용할 수 없을 수 있음 |

자동 재시도는 끄고 요청 제한 시간은 60초로 설정했다. 네트워크 오류 뒤 재실행하면 새 실행이 생성되므로 먼저 Logs를 확인하라. 

# 10. macOS 또는 Linux 에서 실행 

```bash
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
cp .env.example .env
# 편집기로 .env에 기존 키를 설정
.venv/bin/python main.py --check
.venv/bin/python main.py
```

# 11. 예제 소스 및 공식 문서

- [agents sdk trace demo](https://github.com/synabreu/agents-sdk-trace-demo)
- [Agents SDK](https://developers.openai.com/api/docs/guides/agents/sdk)
- [워크플로 tracing](https://developers.openai.com/api/docs/guides/agents/integrations-observability#tracing)
