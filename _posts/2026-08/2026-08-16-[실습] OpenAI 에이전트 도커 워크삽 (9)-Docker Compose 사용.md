---
title: "[실습] OpenAI 에이전트 도커 워크삽 (9)-Docker Compose 사용"
date: 2026-08-16
tags: [오픈AI, OpenAI, GPT-5.6, agenticai, aiagent, openrouter, docker, powershell, fastapi, swagger-ui, visual studio code, Linux, Windows11, Kubernetes, On-Premises ]
typora-root-url: ../
toc: true
categories: [openai]
---
계속해서 이번에는 Docker Compose 사용법에 대해 알아보도록 하겠다. 앞써 설명했듯이, Docker Compose는 여러 개의 Docker 컨테이너를 하나의 설정 파일로 정의하고 함께 실행·관리하는 도구이다.

보통 `docker-compose.yml` 또는 `compose.yml`에  사용할 이미지 또는 빌드 방법, 포트 연결, 환경 변수, 볼륨, 컨테이너 간 네트워크, 실행 순서와 의존성 등을 작성한다.

# 1. 실습 10 - Docker Compose 사용

PowerShell 환경변수 설정 후 다음을 실행한다.

```powershell
$env:OPENAI_API_KEY="YOUR_OPENAI_API_KEY"
$env:OPENAI_DEFAULT_MODEL="gpt-5.6-luna"

docker compose up --build
```

`docker compose up --build`는 Docker Compose로 애플리케이션을 실행하되, 실행 전에 Docker 이미지를 새로 빌드하라는 뜻이다.

백그라운드에서 실행하려면:

```powershell
docker compose up --build -d
```

완전 종료 - 실행을 중지하고 컨테이너를 제거하려면,

```powershell
docker compose down
```

## 1-1. 도커 이미지가 API Key를 포함하지 않는지 확인

Dockerfile과 image history를 확인한다.

```powershell
.\scripts\verify-image.ps1
```

Dockerfile에 다음 같은 코드를 절대로 넣지 않는다.

```dockerfile
ENV OPENAI_API_KEY=sk-xxxxxxxx
```

또한 다음 파일도 도커 이미지(image)에 복사하지 않는다.

```text
.env
```

`.dockerignore`에서 제외되어 있다.

## 1-2. verify-image.ps1 분석

```powershell
# PowerShell 에러 처리 설정 : 오류가 발생하면 즉시 스크립트를 중단함
$ErrorActionPreference = "Stop"

# Docker 이미지 플랫폼 정보를 출력함
# 예: linux/amd64, linux/arm64
Write-Host "Image platform:"

# Docker 이미지의 OS와 CPU Architecture 정보를 조회함
# --format 옵션으로 필요한 정보만 출력함
docker image inspect openai-agents-workshop:1.0 `
    --format '{{.Os}}/{{.Architecture}}'

# 빈 줄을 추가하고 Image size 제목 출력
# `n 은 PowerShell에서 newline(줄바꿈)을 의미함
Write-Host "`nImage size:"

# Docker 이미지 크기를 출력함
# .Size 값은 byte 단위로 표시됨
docker image inspect openai-agents-workshop:1.0 `
    --format '{{.Size}} bytes'
```

Docker 이미지가 정상 생성되었는지 확인하는 용도이다.

* linux/amd64 → Intel/AMD 서버, 대부분의 AWS EC2, Docker Desktop 환경
* linux/arm64 → Apple Silicon(M1/M2/M3), ARM 서버
