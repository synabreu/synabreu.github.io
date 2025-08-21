---
title: "Azure AI Foundry Agent Service(5)-[실습] 에이전트 비동기화 호출"
date: 2025-07-25
tags: [마이크로소프트, Microsoft, Build 2025, Azure AI Foundry, Azure, Azure AI Foundry SDK, Azure OpenAI Studio, Azure OpenAI Service, Azure Machine Learning, Azure App Service, Azure Key Vault, Azure Monitor, Agentic DevOps, Github Copilot, DevOps, MLOps, Software Factory]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

지난 실습에서 윈도우11 운영체제에서 Visual Studio Code 개발 도구를 이용해서 직접 코딩을 통해 에이전트 개발하는 방법에 대해 정리해보았다. 이번 노트는 Azure AI Foundry에서 Azure AI Projects 에이전트 프로젝트와 연결된 비동기 AgentsClient를 사용하는 방법을 보여주도록 하겠다. 또한 어떤 차이가 있는지 정리하겠다. 



## 1. 필수 컴포넌트 설치 

* 필수 컴포넌트 설치는 아래와 같고, 컴포넌트 설치 주석은 아래 부분에 설명해 놓았다.

  ```bash
  pip install azure-ai-projects # Python 환경에 Azure AI Foundry 프로젝트를 제어 및 관리할 수 있는 Azure SDK 패키지를 설치
  pip azure-identity # Azure 서비스에 연결할 때 필요한 인증 및 자격 증명 기능을 제공하는 Azure Python SDK 패키지를 설치
  pip install aiohttp # Python에서 비동기 HTTP 요청과 웹 서버 기능을 제공하는 aiohttp 라이브러리를 설치
  pip install python-dotenv # .env 파일에 저장된 환경 변수를 Python 코드에서 쉽게 불러올 수 있게 해주는 라이브러리를 설치
  ```

  

## 2. Agent 호출 변수 설정

* `sample_agents_async.py` 파일에서 사용하는 .env 파일은 지난번 `sample_agents.py` 예제와 동일하다. 참고로 <> 부분 안의 문자열은 여러분이 애저에서 설정한 값들이다. 보안 관계상 이 노트에서는 값을 표시지 않았다. 

  ```python
  AZURE_PROJECT_NAME=<YOUR-PROJECT-NAME>
  
  AZURE_MODEL_NAME=gpt-4o
  AZURE_MODEL_DEPLOYMENT_NAME=gpt-4o
  
  AZURE_PROJECT_ENDPOINT=<YOUR-AZURE-PROJECT-ENDPOINT>
  
  AZURE_RESOURCE_GROUP=<YOUR-AZURE-RESOURCE-GROUP>
  AZURE_SUBSCRIPTION_ID=<YOUR-AZURE-SUBSCRIPTION-ID>
  ```

* `.env`에서 설정을 불러와 Azure AI Foundry에 비동기 인증 후, 프로젝트 자원에 접근하는 준비 단계

  ```python
  import os # .env 환경 변수 읽기(os.environ), 파일 경로 처리
  import asyncio # 비동기(async/await) 프로그래밍을 지원해 이벤트 루프를 실행하고 비동기 함수(async def)를 관리
  from dotenv import load_dotenv # API 키나 프로젝트 설정을 코드에 직접 하드코딩하지 않고, .env 파일의 환경 변수를 로드하여 os.environ에 추가
  
  # 비동기 전용 클라이언트/크레덴셜
  from azure.identity.aio import DefaultAzureCredential # Azure SDK 비동기 인증 클래스로 로컬 개발 환경, Azure CLI 로그인, Managed Identity 등 다양한 인증 방식을 자동으로 탐색해 aio 버전을 사용하므로 비동기 HTTP 통신함. 
  from azure.ai.projects.aio import AIProjectClient # Azure AI Foundry 프로젝트 비동기 클라이언트. Azure AI Foundry에서 프로젝트, 에이전트, 모델 등 자원에 비동기 방식으로 접근/관리해 async/await로 호출해야 함.
  ```



## 3.  동기와 비동기 에이전트 실행 차이

* 우선 소스를 분석하기 전에, 동기(sync)와 비동기(async) 프로그램 실행방식에 대해 정리하자면 다음과 같다. 

  * **동기(async) 실행 방식:** 코드가 순차적으로 실행되며, `create_agent`와 같이 하나의 API 호출이 끝날 때까지 다음 명령을 기다린다. 예) `project_client.agents.create_agent(...)` → 완료될 때까지 프로그램이 블로킹(blocking)됨.
  
    ```python
    credential = DefaultAzureCredential()
    with AIProjectClient(endpoint=endpoint, credential=credential) as project_client:
        agent = project_client.agents.create_agent(
            model=model_deployment_name,
            name="sample_agents_sync",
            instructions="You are helpful agent",
        )
        print(agent.id)
        project_client.agents.delete_agent(agent.id)
    ```
  
  * **비동기(async) 실행 방식:**  Azure AI Foundry SDK (AIProjectClient)를 비동기(async) 방식으로 사용할 때, API 호출이 네트워크 요청과 같은 **대기 상태** 에 들어가면, 제어권을 이벤트 루프(event loop)에 반환하고, 다른 작업들이 동시에 진행될 수 있다. `await` 키워드로 결과를 기다리지만, 그 동안 다른 코루틴(coroutine)이 실행하는 방식이다. 
  
    ```python
    credential = DefaultAzureCredential()
    async with AIProjectClient(endpoint=endpoint, credential=credential) as project_client:
        agent = await project_client.agents.create_agent(
            model=model_deployment_name,
            name="sample_agents_async",
            instructions="You are helpful agent",
        )
        print(agent.id)
        await project_client.agents.delete_agent(agent.id)
    ```
  
  * 실제 동작 차이: 
  
    * 동기(async): 요청 → 응답 대기 → 다음 줄 실행. 단일 요청만 처리할 때 단순하고 직관적. 
    * 비동기(async): 네트워크 I/O (예: Azure API 호출)이 많은 경우 유리. 여러 개의 에이전트 생성/삭제 같은 작업을 동시에 실행 가능. CPU 연산이 아닌, I/O 대기 시간이 긴 환경에서 성능을 크게 개선.
  
  * 비동기 방식은, 수십 ~ 수백 개의 에이전트를 생성/삭제하는 **대량 API 호출** 처리, 또는 Azure 서비스와의 **네트워크 통신**을 동시에 여러 개 실행하고 싶을 때, 그리고 FastAPI, aiohttp 같은 **비동기 웹 프레임워크**와 연동할 때 자연스럽게 사용한다.



## 4.  비동기 에이전트 메인루틴 분석

* Azure SDK가 인증 토큰을 얻는 방법 정의

  ```python
  credential = DefaultAzureCredential(exclude_interactive_browser_credential=False)
  ```

  * `DefaultAzureCredential` 의 기본 역할

    * `DefaultAzureCredential` 은 Azure SDK가 제공하는 **인증 체인(Credential chain)** 임
    * 실행 환경(로컬 개발, VM, 컨테이너, Azure Functions 등)에 따라 **여러 가지 인증 방식**을 자동으로 시도함
    * 우선순위 예시
      * 환경 변수 기반 인증: (`AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, `AZURE_CLIENT_SECRET` 등)
      * Azure Managed Identity: (Azure VM, App Service, Container Apps 등에서 제공되는 MSI)
      * Visual Studio Code 로그인
      * Azure CLI 로그인 (`az login`)
      * InteractiveBrowserCredential (브라우저 로그인) 

  * `exclude_interactive_browser_credential` 옵션의 의미

    * 이 파라미터는 **브라우저를 통한 사용자 로그인(Interactive Browser)** 을 쓸지 말지를 결정함
    * `exclude_interactive_browser_credential=True` (기본값)
      * 브라우저 로그인은 **시도하지 않음**.
      * 서버/자동화 환경(CI/CD, 클라우드 배포)에서 유용 (브라우저 UI가 없기 때문)
    * `exclude_interactive_browser_credential=False`
      * 브라우저 로그인도 허용.
      * **로컬 개발 환경**에서 로그인 UI가 뜨면서 Azure 계정 인증 가능.

  * 이 코드는 "가능하면 환경 변수, CLI, Managed Identity 등을 쓰고, 그래도 실패하면 **브라우저 창을 열어서 로그인까지 허용**한다"는 뜻이며, 즉, **개발자가 로컬 환경에서 테스트할 때** 인증이 안 되면 마지막 수단으로 **Azure 로그인 페이지를 띄워준다**는 역할

    

## 5.  참고 자료: 전체 소스

```python
 # 비동기 크레덴셜과 클라이언트는 async context manager 사용
    credential = DefaultAzureCredential(exclude_interactive_browser_credential=False)
    async with credential:
        async with AIProjectClient(endpoint=endpoint, credential=credential) as project_client:

            # [START agents_sample_async]
            # 에이전트 생성 (비동기)
            agent = await project_client.agents.create_agent(
                model=model_deployment_name,
                name="sample_agents_async",
                instructions="You are helpful agent",
            )

            # 에이전트 생성 후 ID와 이름을 LOG 출력
            print(f"Created agent, agent ID: {agent.id}")
            print(f"Agent name: {agent.name}")

            # 비동기 방식 - 에이전트 삭제
            await project_client.agents.delete_agent(agent.id)
            print(f"Deleted agent, agent ID: {agent.id}")
            # [END agents_sample_async]
```

