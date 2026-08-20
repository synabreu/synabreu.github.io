---
title: "[실습] OpenAI 에이전트 도커 워크삽 (6)-Dockerfile 과 docker-compose.yml 작성"
date: 2026-08-16
tags: [오픈AI, OpenAI, GPT-5.6, agenticai, aiagent, openrouter, docker, powershell, fastapi, swagger-ui, visual studio code, Linux, Windows11, Kubernetes, On-Premises ]
typora-root-url: ../
toc: true
categories: [openai]
---
이제 본격적으로 OpenAI Agents SDK로 만든 웹 서비스를 도커로 배포하기 위해 Dockerfile 작성하고 리눅스용 이미지 빌드하는 실습을 해보겠다.

# 1. 실습 7 - Dockerfile 작성하기

## 1-1. Dockerfile 작성하기

계속해서 이어 도커파일을 작성하기 위해 Dockerfile의 Base Image는 다음과 같이 선언할 수 있다. 이 파일은 루트 디렉토리에 Dockerfile 이다.

```dockerfile
FROM python:3.12-slim
```

이 한 줄이 매우 중요하다. Windows PC에서 `docker build`를 실행하더라도 이 프로젝트의 최종 application image는 리눅스 기반이 된다.

전체 흐름은,

```text
Windows Source Code
       ↓
Dockerfile
       ↓
python:3.12-slim
       ↓
Linux filesystem + Python + app
       ↓
Linux container image
```

API Key는 Dockerfile에 들어 있지 않다.  참고로 전체 내용을 주석처리해서 보여주면 다음과 같다.

```dockerfile
# Python 3.12가 설치된 경량 Debian 기반 이미지를 애플리케이션의 기본 이미지로 사용한다.
FROM python:3.12-slim


# Python과 pip에서 사용할 환경변수를 설정한다.
#
# PYTHONDONTWRITEBYTECODE=1:
# Python이 __pycache__ 디렉터리와 .pyc 파일을 생성하지 않도록 한다.
#
# PYTHONUNBUFFERED=1:
# Python의 출력 버퍼링을 비활성화하여 로그가 즉시 출력되도록 한다.
#
# PIP_NO_CACHE_DIR=1:
# pip가 패키지 설치 파일을 캐시에 보관하지 않도록 하여 Docker 이미지 크기를 줄인다.
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1


# 컨테이너 내부의 작업 디렉터리를 /app으로 설정한다.
# 이후 COPY, RUN, CMD 명령은 기본적으로 이 디렉터리를 기준으로 실행된다.
WORKDIR /app


# 로컬의 requirements.txt 파일을 컨테이너의 /app/requirements.txt 위치로 복사한다.
COPY requirements.txt .


# pip를 최신 버전으로 업그레이드한다.
# requirements.txt에 정의된 Python 패키지를 설치한다.
#
# 두 명령을 하나의 RUN 명령으로 연결하여 Docker 이미지 레이어 수를 줄인다.
RUN pip install --upgrade pip \
    && pip install -r requirements.txt


# 로컬 app 디렉터리의 소스 코드를 컨테이너의 /app/app 디렉터리로 복사한다.
COPY app ./app


# 이 컨테이너의 애플리케이션이 8000번 포트를 사용한다는 정보를 명시한다.
#
# EXPOSE는 포트를 외부에 직접 공개하지 않는다.
# 실제 외부 연결은 docker run의 -p 옵션이나 docker-compose.yml의 ports 설정으로 구성해야 한다.
EXPOSE 8000


# 컨테이너가 시작될 때 실행할 기본 명령을 정의한다.
#
# uvicorn:
# FastAPI 애플리케이션을 실행하는 ASGI 웹 서버이다.
#
# app.main:app:
# app/main.py 모듈 안의 FastAPI app 객체를 실행한다.
#
# --host 0.0.0.0:
# 컨테이너 외부에서 들어오는 연결을 받을 수 있도록 설정한다.
#
# --port 8000:
# Uvicorn 서버를 컨테이너 내부 8000번 포트에서 실행한다.
CMD [
    "uvicorn",
    "app.main:app",
    "--host",
    "0.0.0.0",
    "--port",
    "8000"
]
```

## 1-2. Dockerfile 빌드

전체 빌드 및 실행 흐름은 다음과 같다.

```
Python 3.12 경량 이미지 준비
    ↓
Python 및 pip 환경변수 설정
    ↓
/app 작업 디렉터리 생성 및 이동
    ↓
requirements.txt 복사
    ↓
Python 패키지 설치
    ↓
애플리케이션 소스 복사
    ↓
8000번 포트 사용 명시
    ↓
Uvicorn으로 FastAPI 실행
```

그렇다면, 이제 dockerfile 을 정의했다면, 이미지를 빌드하자!

```powershell
docker build -t openai-agents-workshop:1.0 .
```

그런 후 이제 컨테이너 실행한다. 예를 들어, 컨테이너의 8000번 포트를 호스트의 8010번 포트에 연결하려면 다음과 같이 실행한다.

```powershell
docker run --rm -p 8010:8000 openai-agents-workshop:1.0
```

이미지가 로컬에 없으면 docker run은 Docker Hub 같은 레지스트리에서 이미지를 내려받으려고 시도하지만, 현재 디렉터리의 Dockerfile을 자동으로 빌드하지는 않는다. 이미지를 이미 빌드했다면 다시 빌드하지 않고 실행만 하면 된다.

웹 브라우저에서는 다음 주소로 접속해서 LLM의 Agent 가 수행하는 프롬프트를 최종적으로 확인할 수 있다.

```
http://localhost:8010/docs
```

## 1-3. docker-compose.yml 분석

docker-compose.yml은 하나 이상의 컨테이너에 필요한 이미지 빌드, 포트 연결, 환경변수, 볼륨, 네트워크 등의 실행 설정을 YAML 형식으로 정의하는 파일이다.

| 파일                   | 역할                                                                        |
| ---------------------- | --------------------------------------------------------------------------- |
| `Dockerfile`         | 애플리케이션 이미지를 어떻게 만들 것인지 정의한다.                          |
| `docker-compose.yml` | 만들어진 이미지를 어떤 설정으로 실행하고 다른 컨테이너와 연결할지 정의한다. |

```yaml
# Docker Compose에서 실행할 컨테이너 서비스 정의
services:

  # 서비스 이름을 agents-api로 지정한다.
  agents-api:

    # 현재 디렉터리의 Dockerfile을 사용하여 이미지를 빌드한다.
    build: .

    # 빌드된 Docker 이미지의 이름과 태그를 지정한다.
    # 이미지 이름은 openai-agents-workshop이고 태그는 1.0이다.
    image: openai-agents-workshop:1.0

    # 호스트와 컨테이너의 포트를 연결한다.
    ports:

      # 호스트의 8000번 포트를 컨테이너의 8000번 포트에 연결한다.
      # 브라우저에서는 http://localhost:8000으로 접근한다.
      - "8010:8000"

    # 컨테이너 내부에서 사용할 환경변수를 정의한다.
    environment:

      # 호스트 환경 또는 .env 파일의 OPENAI_API_KEY 값을
      # 컨테이너의 OPENAI_API_KEY 환경변수로 전달한다.
      OPENAI_API_KEY: ${OPENAI_API_KEY}

      # OPENAI_DEFAULT_MODEL 값을 컨테이너에 전달한다.
      # 환경변수가 없거나 비어 있으면 gpt-5.6-luna를 기본값으로 사용한다.
      OPENAI_DEFAULT_MODEL: ${OPENAI_DEFAULT_MODEL:-gpt-5.6-luna}
```

그래서 실행 흐름은 다음과 같다.

```
docker compose up --build
    ↓
현재 디렉터리의 Dockerfile로 이미지 빌드
    ↓
openai-agents-workshop:1.0 이름으로 저장
    ↓
OPENAI_API_KEY와 모델 설정 전달
    ↓
호스트 8000 → 컨테이너 8000 연결
    ↓
agents-api 컨테이너 실행
```

.env 파일은 다음과 같이 구성할 수 있다,

```dotnet
OPENAI_API_KEY=실제_API_키
OPENAI_DEFAULT_MODEL=gpt-5.6-luna
```

실제 .env 파일은 Git 저장소에 커밋하지 않고 .gitignore에 포함해야 한다.

그러면, Docker Compose에서는 Docker 이미지 빌드와 빌드된 이미지로 컨테이너 생성 및 실행 등 두 작업을 한 명령으로 다음과 같이 처리할 수 있다.

```powershell
docker compose up --build
```

> Dockerfile만 작성한 뒤 docker build와 docker run을 실행해도 된다. docker-compose.yml은 필수가 아니다. Docker Compose는 복잡하고 긴 docker run 명령을 파일로 저장하여 누구나 동일한 환경을 간단하게 실행하도록 하기 위해 사용한다.

| 상황                               | 적합한 방법                                  |
| ---------------------------------- | -------------------------------------------- |
| 컨테이너 하나를 간단히 실행        | Dockerfile +`docker build`, `docker run` |
| 긴 실행 옵션을 반복해서 사용       | Docker Compose                               |
| 여러 컨테이너를 함께 실행          | Docker Compose                               |
| 팀원에게 동일한 실행 설정 제공     | Docker Compose                               |
| 포트·환경변수·볼륨을 파일로 관리 | Docker Compose                               |
