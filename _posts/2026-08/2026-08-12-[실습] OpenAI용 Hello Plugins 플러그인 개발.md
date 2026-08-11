---
title: "2026-08-12-[실습] OpenAI용 Hello Plugins 플러그인 개발"
date: 2026-08-13
tags: [오픈AI, OpenAI, ChatGPT, MCP, Agent, Plugin,  ]
typora-root-url: ../
toc: true
categories: [openai]
---
오픈AI용 플러그인 첫 프로젝트인 Hello Plugins 플러그인 아키텍처를 분석하고 파이썬으로 개발하고 오픈AI Sites 에 배포해서 실행하도록 하는 실습을 해보도록 하겠다. 따라서 “ChatGPT/Codex가 호출할 수 있는 Python MCP 도구”를 구현한 최소 OpenAI Plugin 으로 ChatGPT/Codex의 에이전트가 Plugin의 `hello_plugins` 도구를 발견하고 호출하는 구조이다. 

# 1. 전체 아키텍처

![hello-plugins-01]({{ '/images/2026-08/hello-plugins-01.jpg' | relative_url }})

* 사용자가 “플러그인 인사말을 출력해줘”라고 요청한다.
* ChatGPT/Codex가 Plugin의 도구 목록과 설명을 확인한다.
* hello_plugins가 적합하다고 판단되면 /mcp로 도구 호출을 보낸다.
* Python 서버가 hello_plugins()를 실행한다.
* 고정 문자열 Hello, Plugins!가 반환된다.
* ChatGPT/Codex가 결과를 사용자에게 보여준다.

# 2. 프로젝트 구조

```
hello-plugins-python/
├── server.py          # 실제 MCP 서버와 도구 구현
├── test_server.py     # 함수 단위 테스트
├── verify_mcp.py      # 실제 HTTP MCP 통합 테스트
├── requirements.txt  # pip 설치용 의존성
├── pyproject.toml     # Python 프로젝트 메타데이터
├── Dockerfile         # 컨테이너 이미지 정의
├── .dockerignore      # Docker 빌드 제외 파일
└── README.md          # 실행 및 연결 가이드
```

프로젝트는 크게 네 영역으로 구분된다.

| 영역     | 파일                                         | 역할                |
| ------ | ------------------------------------------ | ----------------- |
| 애플리케이션 | `server.py`                                | MCP 서버 및 도구 정의    |
| 테스트    | `test_server.py`, `verify_mcp.py`          | 함수 및 HTTP 통합 검증   |
| 패키징    | `requirements.txt`, `pyproject.toml`       | Python 버전과 패키지 정의 |
| 배포·문서  | `Dockerfile`, `.dockerignore`, `README.md` | 컨테이너 실행과 사용자 안내   |

# 3. server.py 분석

## 3.1 모듈과 의존성, 그리고 MCP 서버 생성

```python
import os # HOST, PORT 환경변수를 읽음

# FastMCP: Python 함수를 MCP 도구로 노출하고 HTTP 서버를 실행함
from mcp.server.fastmcp import FastMCP

# MCP 서버 생성
mcp = FastMCP(
    "Hello Plugins",
    instructions="Call hello_plugins when the user asks for the plugin greeting.",
    host=os.getenv("HOST", "0.0.0.0"),
    port=int(os.getenv("PORT", "8000")),
    stateless_http=True,
    json_response=True,
)
```

MCP 서버 생성에 대한 각 설정의 의미는 다음과 같다.

| 설정                    | 의미                                 |
| --------------------- | ---------------------------------- |
| `"Hello Plugins"`     | MCP 서버 이름                          |
| `instructions`        | ChatGPT/Codex가 도구를 언제 호출할지 판단하는 지침 |
| `host`                | 서버가 수신할 네트워크 인터페이스                 |
| `port`                | 서버 포트                              |
| `stateless_http=True` | 요청 사이에 서버 세션 상태를 유지하지 않음           |
| `json_response=True`  | MCP 응답을 JSON 형식으로 반환               |

* 환경변수가 없다면 `http://0.0.0.0:8000/mcp` 로 실행 - `0.0.0.0`은 서버가 모든 네트워크 인터페이스에서 요청을 받도록 하는 바인딩 주소이며, 브라우저에 직접 입력하는 목적지는 아니다.
* 사용자는 일반적으로 `http://localhost:8000/mcp` 주소로 접근  


## 3.2 MCP 도구 등록