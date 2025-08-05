---
title: "Azure AI Foundry Agent Service(3)-[실습] Visual Studio Code 에서 새 에이전트 개발"
date: 2025-07-24
tags: [마이크로소프트, Microsoft, Build 2025, Azure AI Foundry, Azure, Azure AI Foundry SDK, Azure OpenAI Studio, Azure OpenAI Service, Azure Machine Learning, Azure App Service, Azure Key Vault, Azure Monitor, Agentic DevOps, Github Copilot, DevOps, MLOps, Software Factory]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

지난 실습에서 Azure AI Foundry Agent Service 의 Foundry Portal 을 이용하여 새 에이전트 생성할 수 있도록 환경 설정과 Agents Playground 에서 테스트를 했다. 그렇다면, 이번 노트는 윈도우11 운영체제에서 Visual Studio Code 개발 도구를 이용해서 직접 코딩을 통해 에이전트 개발하는 방법에 대해 정리해보겠다. 



## 1. Visual Studio Code 에서 Basic-Agent 프로젝트 설정

* Visual Studio Code 에서 `New WIndows` 메뉴를 실행해 `Basic-Agent` 폴더를 생성한다.  그런 후, 해당 프로젝트 폴더에서 새 `Terminal Window`를 실행시켜  **Azure AI Agent Service 및 프로젝트 관리 기능**을 위한 클라이언트 라이브러리로 `pip install azure-ai-projects` 명령으로 설치한다.  

  ![그림1 - azure-ai-projects 설치](/../images/2025-07/VSCode-Agent01.png)

* Microsoft Azure의 다양한 서비스에 **보안 인증(Authentication)**을 수행하기 위한 Python SDK 라이브러리인 **`azure-identity`**를 설치하기 위해 다음과 같이 Terminal Window 에서 `pip install azure-identity` 실행한다.

  ```bash
  pip install azure-identity
  ```

* **Azure CLI (Command-Line Interface)**에서 사용하는 인증 명령어로, **사용자의 Azure 계정으로 로그인**하여 Azure 리소스에 접근할 수 있도록 **인증 토큰을 발급받는 작업**을 수행하도록 다음과 같이 터미널 윈도우에서 `az login` 명령을 실행한다.

  ![](/../images/2025-07/VSCode-Agent02.png)

* PC 클라이언트에서 Azure 서비스를 사용하기 위해서는 여러분의 계정 로그인을 로그인 화면에서 다음과 같이 인증한다.

  ![](/../images/2025-07/VSCode-Agent03.png)

* 여러분의 Azure 계정이 로그인 성공하고 끝났다면, 여러분의 구독(subscription)을 하나 선택한다. 현재 `Azure-subscription-synabreu` 만 사용하고 있으므로 그냥 `Enter Key` 또는 `1` 번 숫자를 입력하면 된다.

  ![그림4 - Subscription 선택](/../images/2025-07/VSCode-Agent04.png)

  

## 2. Visual Studio Code 에서 .env 파일 환경 설정

* Visual Studio Code 에서 **환경 변수(environment variables)**를 정의하고, 개발 중인 애플리케이션이나 개발 도구가 해당 변수들을 자동으로 불러올 수 있게 하는 설정 파일인 `.env` 파일에서 Azure AI Foundry project endpoint 를 설정하기 위해 아래와 같이 복사한다.

  ![그림5 - Azure AI Foundry project endpoint 설정](/../images/2025-07/VSCode-Agent05.png)

  