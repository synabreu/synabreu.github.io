---
title: "[실습] OpenAI용 Hello Plugins 플러그인 개발"
date: 2026-08-13
tags: [오픈AI, OpenAI, ChatGPT, MCP, Agent, Plugin,  ]
typora-root-url: ../
toc: true
categories: [openai]
---
오픈AI용 플러그인 첫 프로젝트인 Hello Plugins 플러그인 아키텍처를 분석하고 파이썬으로 개발하고 오픈AI Sites 에 배포해서 실행하도록 하는 실습을 해보도록 하겠다. 따라서 “ChatGPT/Codex가 호출할 수 있는 Python MCP 도구”를 구현한 최소 OpenAI Plugin 으로 ChatGPT/Codex의 에이전트가 Plugin의 `hello_plugins` 도구를 발견하고 호출하는 구조이다. 

# 1. 전체 아키텍처

![hello-plugins-01]({{ '/images/2026-08/hello-plugins-01.png' | relative_url }}){: width="50%"}

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

```
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

```
@mcp.tool(
    name="hello_plugins",
    description="Return the exact greeting 'Hello, Plugins!'. This tool is read-only.",
    annotations={
        "title": "Say hello to plugins",
        "readOnlyHint": True,
        "destructiveHint": False,
        "openWorldHint": False,
    },
)
```

@mcp.tool 데코레이터는 아래의 Python 함수를 MCP 도구로 등록한다. 도구 메타데이터는 단순 설명이 아니라 ChatGPT/Codex의 도구 선택과 안전 판단에 사용된다. 이 도구는 외부 API, 데이터베이스 또는 파일을 사용하지 않으므로 설정이 정확하다.

| 항목                      | 의미                         |
| ----------------------- | -------------------------- |
| `name`                  | 실제 MCP 도구 이름               |
| `description`           | 도구 사용 목적                   |
| `title`                 | 사람이 읽기 쉬운 도구 제목            |
| `readOnlyHint=True`     | 데이터를 변경하지 않는 읽기 전용 도구      |
| `destructiveHint=False` | 삭제·덮어쓰기 같은 위험 동작 없음        |
| `openWorldHint=False`   | 외부 시스템이나 공개 데이터에 영향을 주지 않음 |


## 3.3 비즈니스 로직

```
def hello_plugins() -> str:
    """Return the plugin's fixed greeting."""
    return "Hello, Plugins!"
```

입력 매개변수가 없고 항상 동일한 문자열을 반환한다. MCP SDK는 Python 반환값을 MCP 텍스트 콘텐츠로 변환한다. 실제 MCP 호출 결과에서는 대략 다음과 같은 형태로 처리된다.

```
{
  "content": [
    {
      "type": "text",
      "text": "Hello, Plugins!"
    }
  ]
}
```

## 3.4 서버 실행

```
if __name__ == "__main__":
    mcp.run(transport="streamable-http")
```

python server.py로 직접 실행했을 때만 서버가 시작된다. streamable-http는 ChatGPT/Codex Plugin 연결에 필요한 HTTP 기반 MCP 전송 방식이다. MCP 엔드포인트는 기본적으로 /mcp에 생성된다.

# 4. test_server.py 분석

`test_server.py`는 가장 작은 단위 테스트이다.

```
from server import hello_plugins

def test_hello_plugins_returns_exact_greeting() -> None:
    assert hello_plugins() == "Hello, Plugins!"
```

* 검증하는 항목은 함수가 정상적으로 import되는가? 실행 중 예외가 발생하지 않는가? 반환 문자열이 정확한가? 쉼표, 공백, 느낌표까지 동일한가 등이다.

* 실행은 다음과 같다. 

```
python -m pytest -q
```

이 테스트는 빠르지만 다음 항목은 검증하지 않다.

* MCP 서버가 실제로 시작되는가
* /mcp가 연결되는가
* 도구가 MCP 목록에 등록되는가
* MCP 요청을 통해 도구가 호출되는가

이 부분은 `verify_mcp.py`가 담당한다. 

# 5. verify_mcp.py 분석

verify_mcp.py는 실제 서버와 MCP 클라이언트를 함께 실행하는 종단 간 테스트이다. 

## 5.1 MCP 클라이언트 연결

```
async with streamablehttp_client(
    "http://127.0.0.1:8000/mcp"
) as (read, write, _):
```

Streamable HTTP 방식으로 실제 MCP 엔드포인트에 연결한다.

## 5.2 MCP 초기화

```
async with ClientSession(read, write) as session:
    await session.initialize()
```

이 과정에서 클라이언트와 서버는 MCP 프로토콜 버전, 서버 이름과 기능, 지원하는 도구 및 기능, 서버 지침 등 정보를 교환한다. 

## 5.3 도구 검색 검증

```
tools = await session.list_tools()
assert [tool.name for tool in tools.tools] == ["hello_plugins"]
```

서버가 정확히 하나의 도구를 광고하고 있으며, 그 이름이 hello_plugins인지 확인합니다.

## 5.4 실제 도구 호출

```
result = await session.call_tool("hello_plugins", {})
assert result.content[0].text == "Hello, Plugins!"
```

빈 입력 객체 {}로 도구를 호출한 후 첫 번째 텍스트 콘텐츠를 검사한다. 이 테스트는 다음 전체 경로를 검증한다. 

```
서버 시작
→ HTTP 연결
→ MCP 초기화
→ 도구 목록 조회
→ 도구 호출
→ 결과 확인
→ 서버 종료
```

## 5.5 프록시 환경변수 제거

```
proxy_keys = (
    "ALL_PROXY",
    "HTTPS_PROXY",
    "HTTP_PROXY",
    ...
)
```

일부 개발 환경에서는 로컬 요청까지 회사 프록시나 SOCKS 프록시로 전달될 수 있다. 테스트 중 해당 환경변수를 잠시 제거하여 `127.0.0.1`에 직접 연결한다. 테스트가 끝나면 기존 값을 복구한다.

```
os.environ.update(saved_proxy_env)
```

## 5.6 서버 프로세스 관리

```
server = subprocess.Popen(
    [sys.executable, "server.py"],
    ...
)
```

현재 가상환경과 동일한 Python 인터프리터로 서버를 실행한다.

```
finally:
    server.terminate()
    server.wait(timeout=5)
```

테스트 성공 여부와 관계없이 서버를 종료하므로 테스트 후 백그라운드 프로세스가 남는 것을 방지한다.

## 5.7 현재 구현의 개선 가능점

time.sleep(1)은 서버가 1초 안에 준비된다고 가정한다.

```
time.sleep(1)
```

느린 PC나 CI 환경에서는 서버 시작에 1초 이상 걸릴 수 있다. 실무적으로는 /mcp 연결이 성공할 때까지 제한된 횟수만 재시도하는 readiness 확인 방식이 더 안정적이다. 또한 서버 로그를 모두 버리고 있다.

```
stdout=subprocess.DEVNULL
stderr=subprocess.DEVNULL
```

테스트가 실패했을 때 원인 분석이 어렵기 때문에 실패 시에만 로그를 출력하도록 개선할 수 있다.

# 6. 의존성 분석

`requirements.txt`

```
mcp[cli]==1.26.0
pytest==8.4.2
mcp[cli]: MCP Python SDK와 MCP 개발용 CLI
pytest: 단위 테스트 프레임워크
```

버전을 고정했기 때문에 다른 환경에서도 동일한 버전이 설치된다. 다만 `requirements.txt`에는 운영 의존성과 개발 의존성이 함께 들어 있다. 그 결과 도커 이미지에도 pytest와 MCP CLI가 설치된다.

운영 환경을 가볍게 만들려면 다음처럼 분리할 수 있다.

```
requirements.txt
requirements-dev.txt
```

예:
```
# requirements.txt
mcp==1.26.0
# requirements-dev.txt
-r requirements.txt
mcp[cli]==1.26.0
pytest==8.4.2
pyproject.toml
```

pyproject.toml은 Python 패키지의 표준 메타데이터이다.

```
[project]
name = "hello-plugins-openai-plugin"
version = "1.0.0"
requires-python = ">=3.10"
```

Python 3.10 이상을 지원한다고 선언한다.

```
dependencies = [
  "mcp[cli]==1.26.0",
]
```

운영 의존성을 정의한다.

```
[project.optional-dependencies]
dev = ["pytest==8.4.2"]
```

개발용 의존성을 분리해 놓았습니다. 이 구조를 사용하면 다음처럼 설치할 수 있습니다.

```
pip install -e ".[dev]"
```

현재는 requirements.txt와 pyproject.toml에서 동일한 의존성을 중복 관리하고 있다. 버전 변경 시 두 파일이 달라질 위험이 있으므로 장기적으로는 pyproject.toml을 기준 파일로 사용하는 것이 좋다.

# 7. Docker 분석

Dockerfile은 프로젝트를 컨테이너로 실행한다.

```
FROM python:3.12-slim # 가벼운 Python 3.12 Debian 이미지를 사용함
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
```

의존성 파일을 먼저 복사하고 패키지를 설치한다. 소스 코드만 변경될 경우 Docker의 의존성 설치 레이어를 재사용할 수 있다.

```
COPY server.py .
```

운영에 필요한 server.py만 이미지에 포함한다. 테스트와 README가 이미지에 포함되지 않는 점은 적절하다.

```
ENV HOST=0.0.0.0
ENV PORT=8000
EXPOSE 8000
```

컨테이너 외부에서 접근할 수 있도록 모든 인터페이스의 8000번 포트에서 수신한다. 

```
CMD ["python", "server.py"]
```

컨테이너가 시작되면 MCP 서버가 실행된다. 
실행:

```
docker build -t hello-plugins .
docker run --rm -p 8000:8000 hello-plugins
```

## 7.1 Docker 개선점

현재 컨테이너가 기본 root 사용자로 실행된다. 공개 서비스라면 별도의 비권한 사용자를 만드는 것이 좋다. 또한 requirements.txt에 pytest가 포함되어 있어 운영 이미지가 불필요하게 커진다.

# 8. .dockerignore 분석

```
.venv/
__pycache__/
.pytest_cache/
*.pyc
```

Docker 빌드 컨텍스트에서 로컬 가상환경, Python 바이트코드, pytest 캐시, 자동 생성된 .pyc 등 파일을 제외한다. 적절한 최소 설정이며, 실무에서는 다음도 추가할 수 있습니다.

```
.git/
.gitignore
README.md
test_*.py
verify_mcp.py
```

현재 Dockerfile이 server.py와 requirements.txt만 선택적으로 복사하므로 필수 개선은 아니다.

# 9. Plugin과 Agent의 정확한 관계

| 개념                | 이 프로젝트의 적용 여부 |
| ----------------- | ------------- |
| OpenAI Plugin     | 적용            |
| MCP 서버            | 적용            |
| MCP 도구            | 적용            |
| OpenAI Agents SDK | 사용하지 않음       |
| 자체 LLM 호출         | 없음            |
| Skill             | 없음            |
| 사용자 인증            | 없음            |
| 사용자 정의 UI         | 없음            |

“Agent Plugin”이라는 표현은 에이전트가 사용할 수 있는 Plugin이라는 의미로는 맞다. 그러나 독립적인 Agent를 Python으로 실행하는 프로젝트는 아니다. 독립적인 Python Agent를 만들려면 별도로 다음 구조를 필요로 한다.

```
OpenAI Agents SDK Agent
→ MCP 서버 연결
→ hello_plugins 호출
→ 최종 답변 생성
```

현재 구조에서는 ChatGPT Work 또는 Codex가 Agent 역할을 담당한다.

# 10. 보안과 운영성 평가

현재 도구는 다음 특징을 가진다.

* 외부 데이터를 읽지 않음
* 파일을 수정하지 않음
* 사용자 데이터를 받지 않음
* 인증정보를 사용하지 않음
* 고정 문자열만 반환
* 상태를 저장하지 않음

따라서 학습용으로는 위험이 매우 낮고, 공개 Plugin으로 배포하려면 추가로 고려해야 합니다.

* 공개 HTTPS 엔드포인트
* 안정적인 도메인
* 요청 로그와 오류 모니터링
* 비권한 Docker 사용자
* 헬스 체크
* 타임아웃과 요청 제한
* Plugin 제출용 설명과 개인정보 처리 안내

현재처럼 정적 문자열만 반환하는 개인 Plugin에는 OAuth 인증이 필요하지 않는다.

# 12. 전체 정리

이 프로젝트는 다음을 학습하기에 좋은 최소 예제이다.

* Python 함수를 MCP 도구로 만드는 방법
* 도구 메타데이터와 안전 annotation 설정
* Streamable HTTP MCP 서버 실행
* MCP 클라이언트로 도구 검색 및 호출
* ChatGPT Work Plugin 연결
* Docker 기반 배포

완성도는 “학습용 최소 Plugin” 기준으로 충분하며, 단위 테스트와 실제 MCP 통합 테스트가 모두 포함되어 있다는 점이 특히 좋습니다. 실무 수준으로 확장할 때 우선순위는 다음과 같다.

* 테스트의 고정 sleep(1)을 readiness 재시도로 변경
* 운영·개발 의존성 분리
* Docker 비권한 사용자 적용
* 실패 시 서버 로그 출력
* 헬스 체크 추가
* 공개 HTTPS 환경 배포
* 필요하면 Skill 또는 Agents SDK Agent 추가

