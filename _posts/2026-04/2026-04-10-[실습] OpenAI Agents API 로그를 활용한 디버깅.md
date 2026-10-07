---
title: "[실습] OpenAI Agents API 로그를 활용한 디버깅"
date: 2026-04-10
tags: [openai, devday, samaltman, chatgpt, codex, linux, codexmobile, meta, muse, spacexai, grotbot, dots, ChatGPT Spaces]
typora-root-url: ../
toc: true
categories: [openai]
---

이번 블로그는 기본적이지만 OpenAI 개발자들에게는 많이 사용하는 OpenAI Agents API 로그를 활용한 디버깅이다. 사용하는 하는 언어는 언제나 Python 언어(Python 3.10 이상 권장)이다. 따라서, 이 예제는 Python 가상환경에서 Responses API를 호출하고, OpenAI Platform Logs에서 요청과 답변을 확인하는 초보자용 예제이다. 

# 1. 왜 개발자는 로그를 찍는 이유는 무엇인가?

개발자가 로그를 남기는 이유는 프로그램에서 무슨 일이 일어났는지 확인하기 위해서이다. 또한 화면에 결과만 보이면 그 결과가 나온 과정이나 실패 원인을 알기 어렵기 때문이다. 

* 오류 원인 찾기: 어떤 작업에서 왜 실패했는지 확인함
* 동작 확인: 요청이 들어왔고 작업이 끝났는지 확인함
* 성능 확인: 어느 단계에서 시간이 오래 걸리는지 찾음
* 문제 재현: 사용자가 겪은 문제를 당시 기록으로 조사함
* 사용량 확인: API 호출 횟수와 토큰 사용량 등을 살펴봄

예를 들어 AI 답변이 나오지 않았을 때, 로그를 보면 API 키 오류인지 사용 한도 문제인지, 도구 실행 실패인지 구분할 수 있다.그래서 로그는 개발 중에도 필요하지만 서비스를 실제로 운영할 때 특히 중요한다. 다만 API 키나 비밀번호 같은 비밀 정보는 로그에 남기지 않아야 한다. 

# 2. Python OpenAI Logs 실습 프로젝트 포함 파일

```text
openai-logs-demo/
├── main.py           # API 호출 및 설정 확인
├── requirements.txt  # 필요한 라이브러리
├── .env.example      # 환경 설정 샘플
├── .gitignore        # 키 및 가상환경을 Git에서 제외
└── README.md         # 설치 및 실행 안내
```

`.env`와 `.venv`는 아래 과정을 따라 로컬에서 생성한다. 실제 키는 제공된 파일에 포함되어 있지 않다. 

# 3. main.py 소스 분석

```python
import argparse
import os
from pathlib import Path

from dotenv import load_dotenv
from openai import APIConnectionError, APIStatusError, AuthenticationError, OpenAI, RateLimitError

# 실행 폴더가 달라도 설정 파일을 찾도록 현재 소스 파일의 폴더를 기준으로 함
PROJECT_DIR = Path(__file__).resolve().parent

# 요청 내역을 확인할 OpenAI 플랫폼의 Responses Logs 주소
LOGS_URL = "https://platform.openai.com/logs?api=responses"


def main() -> int:
    # 설정 확인, API 요청, 결과 출력을 담당하는 진입 함수
    # 정상 종료 시 0, 입력·설정 또는 API 오류 발생 시 1을 반환함

    # 커맨드라인에서 질문과 설정 확인 여부를 받고, 질문이 없으면 기본 문장을 사용함
    parser = argparse.ArgumentParser(description="OpenAI Responses Logs 실습")
    parser.add_argument("--prompt", default="파이썬을 처음 배우는 사람에게 응원 한 문장을 한국어로 써줘.")
    parser.add_argument("--check", action="store_true", help="API 요청 없이 설정만 확인")
    args = parser.parse_args()

    # 실행 위치와 관계없이 main.py 옆의 .env를 읽음
    # 이 실습에서는 .env 값이 기존 환경 변수보다 우선함
    load_dotenv(PROJECT_DIR / ".env", override=True, encoding="utf-8-sig")

    # 키와 모델 이름의 앞뒤 공백을 제거하고 모델 설정이 없으면 기본값을 사용함
    api_key = os.getenv("OPENAI_API_KEY", "").strip()
    model = os.getenv("OPENAI_MODEL", "gpt-4.1-mini").strip()

    # 실제 요청 전에 비어 있는 값과 예제용 API 키를 검사함
    if not api_key or api_key == "your_api_key_here":
        print("설정 오류: .env의 OPENAI_API_KEY에 실제 API 키를 입력하세요.")
        return 1
    if not model:
        print("설정 오류: .env의 OPENAI_MODEL을 입력하세요.")
        return 1
    if not args.prompt.strip():
        print("입력 오류: 질문을 입력하세요.")
        return 1
    # --check는 설정 값만 검사하며 API 요청이나 키 인증을 수행하지 않습니다.
    if args.check:
        print(f"설정 확인 완료 / 모델: {model}")
        print("API 키 값은 표시하지 않습니다. 유효성 및 API 접속은 확인하지 않았습니다.")
        return 0

    try:
        # 클라이언트는 블록 종료 시 정리하며, 제한 시간은 60초로 설정함
        # 자동 재시도는 끄고 오류가 나면 사용자가 결과를 확인하도록 함
        with OpenAI(api_key=api_key, timeout=60.0, max_retries=0) as client:
            # 선택한 모델에 질문을 전달하고 응답 저장을 요청합니다.
            response = client.responses.create(
                model=model,
                input=args.prompt,
                store=True,
            )
    except AuthenticationError:
        # API 키 인증에 실패한 경우
        print("인증 오류: API 키가 유효한지 확인하세요.")
        return 1
    except RateLimitError:
        # 사용량 또는 요청 속도 한도에 걸린 경우
        print("한도 오류: API 크레딧, 사용량 한도 또는 요청 속도를 확인하세요.")
        return 1
    except APIConnectionError:
        # 연결 실패 시 서버 처리 여부가 불명확할 수 있어 Logs 확인을 안내함
        print("연결 오류: 인터넷 연결을 확인하세요. 요청 결과가 불명확하면 재실행 전 Logs를 확인하세요.")
        return 1
    except APIStatusError as error:
        # 그 밖의 API 상태 오류는 HTTP 상태 코드와 함께 안내함
        print(f"API 오류 (HTTP {error.status_code}): 모델 권한 및 프로젝트 설정을 확인하세요.")
        return 1

    # 답변 텍스트를 출력하고 텍스트가 없으면 안내 문구로 대체함
    print("AI 답변:")
    print(response.output_text or "텍스트 답변이 없습니다.")

    # 응답 ID와 상태, Logs 주소를 출력해 플랫폼에서 요청을 찾을 수 있게 함
    print(f"\n응답 ID: {response.id}")
    print(f"상태: {response.status}")
    print(f"Logs: {LOGS_URL}")
    print("API 키를 만든 조직/프로젝트를 선택하여 요청을 확인하세요.")

    return 0


if __name__ == "__main__":
    # 직접 실행할 때 main을 호출하고 반환값을 프로세스 종료 코드로 전달함
    raise SystemExit(main())
```

# 4. Windows PowerShell에서 프로젝트 폴더 열기

공개된 github 프로젝트에서 다음과 같이 `git clone'을 사용해서 복제할 수 있다.

```powershell
git clone https://github.com/synabreu/openai-logs-demo.git
```

그런 후 PowerShell 또는 VS Code 터미널을 연다. 

```powershell
python --version
python -m venv .venv
```

# 5. 가상환경 활성화 및 설치

```powershell
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

앞에 `(.venv)`가 나타나면 활성화된 것이다. 가상환경에는 이 프로젝트의 라이브러리가 별도로 설치된다. PowerShell에서 스크립트 실행이 차단되면 활성화 없이 아래처럼 가상환경의 Python을 직접 사용할 수 있고 시스템 정책을 변경할 필요가 없다.

```powershell
.\.venv\Scripts\python.exe -m pip install --upgrade pip
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

이 방식을 선택했다면 아래 실행 명령의 `python`도 `.\.venv\Scripts\python.exe`로 바꿔라. 

# 6. .env 만들기

```powershell
Copy-Item .env.example .env
notepad .env
```

이미 `.env`를 만들었다면 복사 명령을 다시 실행하지 말고 기존 파일을 편집하라. 아래 값에서 `your_api_key_here`만 실제 API 키로 변경하고 저장한다. 

```dotenv
OPENAI_API_KEY=your_api_key_here
OPENAI_MODEL=gpt-4.1-mini
```

키는 [OpenAI Platform API Keys](https://platform.openai.com/api-keys)에서 확인하거나 생성할 수 있다. 키를 채팅이나 GitHub에 올리지 마라. API 사용에는 별도의 API 결제 설정이 필요하며 사용 요금이 발생할 수 있다.

`main.py`는 `python-dotenv`로 자신의 폴더에 있는 `.env`를 읽는다. 이 실습은 `override=True`를 사용하여 `.env`의 값이 기존 환경 변수보다 우선하도록 구성했다. `.env`는 일반 텍스트 파일이며 `.gitignore`는 암호화가 아닌 Git 업로드 제외 설정했다. 

# 7. 설정만 확인하기 — API 호출 없음

```powershell
python main.py --check
```

키 값은 출력하지 않는다. 또한, 이 명령은 설정 여부만 확인하며 키의 실제 유효성, 크레딧 또는 모델 권한은 확인하지 않는다. 아래와 같은 그림에 메시지가 나온다면 정상이다.

![그림 1. main check]({{ '/images/2026-04/openai-log-01.png' | relative_url }}){: style="width: 80%; height: auto;"}
<br>*[그림 1. Python main --check]*

# 8. 실제 OpenAI API 호출하기

아래와 같이 실행한다. 

```powershell
python main.py
```

그러면 다음과 같은 결과 화면이 그림에 나온다. 

![그림 2. main 실행]({{ '/images/2026-04/openai-log-02.png' | relative_url }}){: style="width: 80%; height: auto;"}
<br>*[그림 2. Python main 실행]*

다른 질문을 보내려면:

```powershell
python main.py --prompt "OpenAI API가 로깅 방법을 알려줘"
```

![그림 3. main argument 실행]({{ '/images/2026-04/openai-log-05.png' | relative_url }}){: style="width: 80%; height: auto;"}
<br>*[그림 3. main argument 실행]*


답변에 `resp_...` 형태의 응답 ID, 응답 상태, Logs 주소가 표시된다. 재실행할 때마다 새로운 API 요청을 보내고 이 예제는 자동 재시도를 꺼두었으며 요청 제한 시간은 60초이다. 

# 9. Platform Logs 확인하기

1. [Responses Logs](https://platform.openai.com/logs?api=responses)를 연다. 
2. API 키를 만든 조직과 프로젝트를 선택한다. 
3. 날짜 범위와 검색 조건을 확인하고 새로고침한다. 
4. 방금 보낸 요청을 열어 입력과 출력을 확인한다. 

![그림 4. OpenAI 로그 화면]({{ '/images/2026-04/openai-log-03.png' | relative_url }}){: style="width: 80%; height: auto;"}
<br>*[그림 4. OpenAI 로그 화면]*

![그림 5. OpenAI 로그 상세 화면]({{ '/images/2026-04/openai-log-04.png' | relative_url }}){: style="width: 80%; height: auto;"}
<br>*[그림 5. Python 로그 상세 화면]*


파이썬 소스에서 `client.responses.create(..., store=True)`가 응답을 저장하도록 요청한다. `print()`는 터미널 표시용이다. 공식 문서는 Response 객체의 기본 저장 기간을 30일로 안내한다. 조직이나 프로젝트에 Zero Data Retention이 적용되어 있으면 `store=True`도 `false`로 처리된다. 

# 10. macOS 또는 Linux 운영체제에서 실행하려면?

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
cp .env.example .env
# 편집기로 .env를 열고 API 키 입력
python main.py --check
python main.py
```

# 11. 자주 만나는 문제

| 문제 | 해결 방법 |
|---|---|
| `No module named openai` 또는 `dotenv` | 가상환경의 Python으로 requirements.txt 설치 |
| API 키 설정 오류 | `.env.txt`가 아닌 `.env`인지, 실제 키를 입력했는지 확인 |
| 인증 오류 | 만료되거나 삭제된 키인지 확인 |
| 한도 오류 | API 크레딧, 사용 한도 또는 요청 속도 확인 |
| 모델 관련 오류 | `.env`의 모델 이름과 프로젝트 사용 권한 확인 |
| 연결 오류 | 인터넷 연결 확인. 결과가 불명확하면 재실행 전 Logs 확인 |
| Logs가 비어 있음 | 조직, 프로젝트, 날짜 필터와 데이터 보존 설정 확인 |

가상환경을 종료하려면 활성화한 터미널에서 `deactivate`를 실행합니다.

# 12. 참고 문서

- [OpenAI Python SDK 및 .env 안내](https://developers.openai.com/api/reference/python)
- [Responses 저장과 Logs](https://developers.openai.com/api/docs/guides/conversation-state)
- [데이터 보존 설정](https://developers.openai.com/api/docs/guides/your-data)
- [python-dotenv 문서](https://bbc2.github.io/python-dotenv/)
