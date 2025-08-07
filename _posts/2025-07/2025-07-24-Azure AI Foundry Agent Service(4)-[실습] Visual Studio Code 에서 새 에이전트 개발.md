---
title: "Azure AI Foundry Agent Service(4)-[실습] Visual Studio Code 에서 새 에이전트 개발"
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
* BASIC-AGENT 프로젝트 파일에서 .env 파일을 생성하고 다음과 같이 설정한다. <> 부분 안의 문자열은 여러분이 애저에서 설정한 값들이다. 보안 관계상 이 노트에서는 값을 표시지 않았다. 

  ```.env
  AZURE_PROJECT_NAME=<YOUR-PROJECT-NAME>
  
  AZURE_MODEL_NAME=gpt-4o
  AZURE_MODEL_DEPLOYMENT_NAME=gpt-4o
  
  AZURE_PROJECT_ENDPOINT=<YOUR-AZURE-PROJECT-ENDPOINT>
  
  AZURE_RESOURCE_GROUP=<YOUR-AZURE-RESOURCE-GROUP>
  AZURE_SUBSCRIPTION_ID=<YOUR-AZURE-SUBSCRIPTION-ID>
  ```
  | 변수명                          | 설명                                                                                  | 용도                                                                                                                                                                                |
  | ------------------------------- | ------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
  | `AZURE_PROJECT_NAME`          | Azure AI Foundry에서 생성한 AI 프로젝트의 이름                              | 에이전트, 모델, 지식 등을 관리하는 단위 프로젝트 이름. SDK에서 프로젝트를 지정할 때 사용함.                                                                                         |
  | `AZURE_MODEL_NAME`            | 사용할 Azure OpenAI 모델 이름 (예: `gpt-4`, `gpt-4o`, `gpt-35-turbo`) | 어떤 언어 모델을 사용할지 지정함. 내부 식별이나 로깅에서 사용되며, 실제 배포된 이름과 다를 수 있음.                                                                                 |
  | `AZURE_MODEL_DEPLOYMENT_NAME` | 실제로 배포된 모델 인스턴스의 이름                                          | Azure OpenAI Studio 또는 Foundry 내에서 모델을 배포할 때 붙이는 고유명. API 호출 시 이 이름을 기반으로 모델을 식별함.                                                               |
  | `URE_PROJECT_ENDPOINT`        | 해당 AI 프로젝트의 REST API Endpoint URL                                    | `AIProjectClient`나 `AgentsClient`가 이 엔드포인트를 통해 에이전트, 프로젝트, 지식 등을 API로 제어함. 예) `https://<subdomain>.services.ai.azure.com/api/projects/<project>` |
  | `AZURE_RESOURCE_GROUP`        | Azure 리소스 그룹 이름                                                      | AI Foundry 프로젝트가 속해 있는 리소스 그룹. 리소스 배포와 관리를 위한 논리적 그룹입니다. 인프라 및 보안 정책에도 영향을 줌.                                                 |
  | `AZURE_SUBSCRIPTION_ID`       | Azure 구독 ID                                                               | Azure 리소스를 소유하고 결제하는 단위인 구독의 고유 식별자로, 프로젝트, 에이전트 등을 만들거나 연결할 때 이 ID가 필요함.                                               |

* `sample_agents.py` 파일을 생성하고 아래의 내용을 추가해 넣어라.

  ```python
  import os
  from azure.identity import DefaultAzureCredential
  from azure.ai.projects import AIProjectClient # Azure AI Foundry SDK (azure-ai-projects)
  from dotenv import load_dotenv
  
  # 환경 설정(.env) 파일의 내용을 읽음.
  load_dotenv()
  
  # 환경 변수에서 프로젝트 설정 로드
  project_name = os.environ["AZURE_PROJECT_NAME"]
  endpoint = os.environ["AZURE_PROJECT_ENDPOINT"]
  model_deployment_name = os.environ["AZURE_MODEL_DEPLOYMENT_NAME"]
  resource_group = os.environ["AZURE_RESOURCE_GROUP"]
  subscription_id = os.environ["AZURE_SUBSCRIPTION_ID"]
  
  # Azure CLI 또는 Azure SDKs 에서 설정한 DefaultAzureCredential 을 사용해 Azure 자격 증명 생성
  with DefaultAzureCredential(exclude_interactive_browser_credential=False) as credential:
  
      # AIProjectClient 인스턴스 생성 - endpoint와 credential 기반으로 프로젝트 연결
      with AIProjectClient(endpoint=endpoint, credential=credential) as project_client:
  
          # [START agents_sample]
          # 정확한 프로젝트명으로 Agent 생성
          agent = project_client.agents.create_agent(
              model=model_deployment_name,
              name="sample_agents",
              instructions="You are helpful agent",
          )
  
          # Agent 생성 후 ID와 이름 출력
          print(f"Created agent, agent ID: {agent.id}")
          print(f"Agent name: {agent.name}")
  
  
          # 작업 후 정리: Agent 삭제  
          # project_client.agents.delete_agent(agent.id)
          # print(f"Deleted agent, agent ID: {agent.id}")
          # [END connection_sample]
  ```

* `python sample_agents.py` 파일을 실행하면, 다음과 같은 결과를 얻는다. `delete agent` 부분은 애저의 Tracing ID를 여러분들께 보여주기 위해 주석 처리했다. 

  ![그림6 - Visual Studio Code에서 sample_agents.py 실행](/../images/2025-07/VSCode-Agent06.png)

* 그리고 나서, My agents 로깅 부분에서 `agent name` 과 `agent id` 명이 `sample_agents.py` 에서 실행한 부분과 일치한 것을 확인할 수 있다.  

  ![그림7 - Visual Studio Code에서 sample_agents.py 실행](/../images/2025-07/VSCode-Agent07.png)

  

