---
title: "[실습] OpenAI 에이전트 도커 워크삽 (4)-FastAPI 서비스"
date: 2026-08-16
tags: [오픈AI, OpenAI, GPT-5.6, agenticai, aiagent, openrouter, docker, powershell, fastapi, swagger-ui, visual studio code, Linux, Windows11, Kubernetes, On-Premises ]
typora-root-url: ../
toc: true
categories: [openai]
---
이 블로그는 윈도우11 운영체제 환경에서 Powershell을 포함한 Visual Studio Code를 이용해

# 1. 실습 6 - FastAPI로 서비스화

`app/main.py`는 Agents SDK를 HTTP API로 노출한다.

주요 엔드포인트:

| Method | Endpoint           | 기능                  |
| ------ | ------------------ | --------------------- |
| GET    | `/health`        | 상태 확인             |
| POST   | `/agents/simple` | Simple Agent          |
| POST   | `/agents/tool`   | Tool Calling          |
| POST   | `/agents/triage` | Handoff / Multi-Agent |
| GET    | `/docs`          | Swagger UI            |

Windows 11 운영체제 환경에서 실행한다.

```powershell
.\scripts\run-native.ps1
```

웹 브라우저:

```text
http://localhost:8010/docs
```

다른 파워셀 창에서 테스트한다.

```powershell
.\scripts\test-api.ps1
```

여기까지는 **Docker를 사용하지 않은 Windows 네이티브 개발 및 디버깅**이다.

## 6-1. run-native.ps1 분석

```python
$ErrorActionPreference = "Stop"

if (-not $env:OPENAI_API_KEY) {
    throw "OPENAI_API_KEY is not set. Example: `$env:OPENAI_API_KEY='sk-...'"
}

if (-not $env:OPENAI_DEFAULT_MODEL) {
    $env:OPENAI_DEFAULT_MODEL = "gpt-5.6-luna"
}

& .\.venv\Scripts\Activate.ps1
uvicorn app.main:app --host 127.0.0.1 --port 8010 --reload
```

위의 소스는 윈도우 11 환경으로 FastAPI 기반 OpenAI Agents SDK 애플리케이션을 실행하는 역할을 한다. OpenAI API Key 존재 여부 확인과 `gpt-5.6-luna`를 기본 모델 지정한다. 여기서 OpenAI API Key 를 보안 관계상 직접 입력하지 말고 파워쉘의 environment 변수를 통해 입력한다.

| 구성요소 | 종류 | 설명 |
|---|---|---|
| `uvicorn` | 실행 명령 | FastAPI 같은 ASGI 애플리케이션을 실행하는 웹 서버이다. |
| `app.main:app` | 애플리케이션 인자 | 실행할 FastAPI 애플리케이션의 위치를 지정한다. |
| `app.main` | Python 모듈 | `app` 디렉터리의 `main.py` 파일을 의미한다. |
| `:app` | 객체 이름 | `main.py` 안에 선언된 `app = FastAPI(...)` 객체를 의미한다. |
| `--host` | 옵션 | 서버가 요청을 받을 네트워크 주소를 지정한다. |
| `127.0.0.1` | `--host`의 인자 | 현재 컴퓨터에서 들어오는 요청만 허용한다. |
| `--port` | 옵션 | 서버가 사용할 포트를 지정한다. |
| `8010` | `--port`의 인자 | 서버가 8010번 포트에서 요청을 받도록 한다. |
| `--reload` | 옵션 | Python 파일이 변경되면 서버를 자동으로 재시작한다. 개발 환경에서 사용한다. |

실행 흐름은 다음과 같다.

```
Uvicorn 실행
    ↓
app/main.py 모듈 불러오기
    ↓
main.py의 app 객체 찾기
    ↓
127.0.0.1:8010에서 서버 실행
    ↓
코드 변경 시 자동 재시작
```

실행 후 Swagger 문서는 다음 주소로 접속한다.

```
http://localhost:8010/docs
```

127.0.0.1은 외부 컴퓨터나 Docker 컨테이너 밖에서 접근하기 어렵다. Docker 컨테이너에서 실행한다면 일반적으로 다음처럼 0.0.0.0을 사용한다.

```
uvicorn app.main:app --host 0.0.0.0 --port 8010
```

--reload는 개발 편의 기능이므로 운영 환경에서는 일반적으로 사용하지 않는다.

## 6-2. test-api.ps1 소스 분석

```Powershell

# PowerShell 명령 실행 중 오류가 발생하면 스크립트를 즉시 중단한다.
$ErrorActionPreference = "Stop"


# ------------------------------------------------------------
# 1. Health Check API 테스트
# ------------------------------------------------------------

# 현재 실행할 테스트의 이름을 화면에 출력한다.
Write-Host "Health check"

# FastAPI 서버의 상태 확인 API를 GET 방식으로 호출한다.
# 서버가 정상적으로 실행 중이면 {"status": "ok"} 응답을 받는다.
Invoke-RestMethod `
    -Uri "http://localhost:8010/health" `
    -Method Get


# ------------------------------------------------------------
# 2. Simple Agent 테스트
# ------------------------------------------------------------

# `n은 줄바꿈을 의미한다.
# 한 줄을 띄운 후 테스트 이름을 화면에 출력한다.
Write-Host "`nSimple Agent"

# Agent에게 전달할 사용자 메시지를 PowerShell 해시 테이블로 작성한다.
# ConvertTo-Json은 해시 테이블을 JSON 문자열로 변환한다.
$body = @{
    message = "Docker image가 무엇인지 초보자에게 한 문장으로 설명해줘."
} | ConvertTo-Json

# simple Agent API를 POST 방식으로 호출한다.
#
# -Uri는 요청을 보낼 API 주소이다.
# -Method Post는 HTTP POST 방식을 사용한다.
# -ContentType은 요청 본문의 형식이 JSON임을 나타낸다.
# -Body는 앞에서 만든 JSON 메시지를 전달한다.
Invoke-RestMethod `
    -Uri "http://localhost:8010/agents/simple" `
    -Method Post `
    -ContentType "application/json" `
    -Body $body


# ------------------------------------------------------------
# 3. Function Tool Agent 테스트
# ------------------------------------------------------------

# 한 줄을 띄운 후 Tool Agent 테스트 이름을 출력한다.
Write-Host "`nTool Agent"

# 상품 가격과 수량 계산을 요청하는 메시지를 JSON으로 변환한다.
$body = @{
    message = "가격이 12500원인 상품 3개의 총액을 계산해줘."
} | ConvertTo-Json

# tool Agent API를 호출한다.
# tool Agent는 계산이 필요하다고 판단하면 calculate_total 도구를 호출한다.
Invoke-RestMethod `
    -Uri "http://localhost:8010/agents/tool" `
    -Method Post `
    -ContentType "application/json" `
    -Body $body


# ------------------------------------------------------------
# 4. Triage 및 Handoff Agent 테스트
# ------------------------------------------------------------

# 한 줄을 띄운 후 Triage/Handoff Agent 테스트 이름을 출력한다.
Write-Host "`nTriage/Handoff Agent"

# 사내 Kubernetes 배포에 관한 사용자 메시지를 JSON으로 변환한다.
$body = @{
    message = "이 컨테이너를 사내 Kubernetes에 배포하려면 무엇을 준비해야 하나?"
} | ConvertTo-Json

# triage Agent API를 호출한다.
#
# triage Agent는 요청 내용을 분석한다.
# 이 요청은 사내 환경에 관한 질문이므로 on_prem_specialist로
# handoff할 가능성이 높다.
Invoke-RestMethod `
    -Uri "http://localhost:8010/agents/triage" `
    -Method Post `
    -ContentType "application/json" `
    -Body $body
```
