---
title: "[실습] OpenAI 관리형 RAG API 워크샵(3)-개발환경준비"
date: 2026-08-26
tags: [오픈AI, OpenAI, GPT-5.6, agenticai, aiagent, openrouter, docker, powershell, fastapi, swagger-ui, visual studio code, Linux, Windows11, Kubernetes, On-Premises ]
typora-root-url: ../
toc: true
categories: [openai]
---
전체 워크삽 실습은 기업용 문서 파일을 OpenAI에 있는 클라우드 서비스에 있는 벡터 스토어(Vector Store)에 저장하고 검색 가능한 지식 기반 시스템을 구축하는 것이다. 전체적으로 Retrieval API와 Responses API의 File Search를 직접 사용하며 검색 결과가 근거 기반 답변으로 연결되는 과정을 단계별로 익힌다. 마지막에는 FastAPI를 활용해 완성된 RAG 시스템을 웹 API 서비스로 구현해 배포하는 워크삽이다.

우선 첫번쨰 핸즈온 워크삽으로 윈도우 11, macOS 또는 Linux 운영체제에서 이 워크삽을 실행할 수 있는 준비 환경을 아래와 같이 따라해 본다.

# 1. 워크삽 목표

이 워크삽을 끝내면 다음 흐름을 직접 설명하고 실행할 수 있다.

```text
Document
  -> Vector Store
  -> Retrieval
  -> File Search
  -> Responses API
  -> RAG Answer
  -> FastAPI
```

# 2. 실습 - 개발 환경 준비

## 2.1 준비사항

- Windows 11, macOS 또는 Linux 운영체제
- Python 3.11 이상 권장
- Visual Studio Code 권장
- OpenAI API 사용이 가능한 계정과 API Key

## 2-2. 파워셀창에서 폴더 생성 및 Visual Studio Code 실행

파워쉘(Powershell)을 실행한다음 워크삽 프로젝트 폴더를 만들고, Visual Studio Code를 실행하기 위해 하기 위해서는 다음과 같이 실행한다.

```
mkdir openai-rag-workshop
cd openai-rag-workshop
code .
```

## 2-3. Terminal 창 실행

Visual Studio Code 메뉴에서 `Terminal` 을 선택해서 `New Terminal` 을 선택하면 된다. 만약 Terminal 창이 Powershell이 아니면 아래와 같이 변경해준다.

![openai-rag]({{ '/images/2026-08/openai-rag-01.png' | relative_url }})

터미널 창의 툴바에서 `아래 화살표`를 누르면 팝업 메뉴가 뜨는 데 그때 `powershell` 리스트 박스를 선택하면 된다.


## 2-4. Python 가상환경 생성

```powershell
python -m venv .venv
```

활성화:

```powershell
.\.venv\Scripts\Activate.ps1
```

macOS/Linux:

```bash
source .venv/bin/activate
```

## 2-5. 패키지 설치

```powershell
python -m pip install --upgrade pip
pip install -r requirements.txt
```

## 2-6. OpenAI의 API를 사용하기 위한 환경 변수 준비

`.env.example`을 `.env`로 복사한다.

PowerShell:

```powershell
Copy-Item .env.example .env
```

`.env`에 본인의 API key를 설정한다.

주의: `.env` 파일은 Git에 commit하지 않는다.
