---
title: "[실습] 파운드리 로컬에서 LangChain과 OpenWebUI 사용"
date: 2025-06-13
tags: [Microsoft, 마이크로소프트, Windows Foundry, Foundry Local, Windows Azure, Azure AI Foundry, AI Agent, Ollama, OpenAI, Visual Studio Code, LangChain, Open Web UI]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

이번 실습에서는 LangChain 과 통합하는 방법과 Open Web UI와 함께 사용하는 방법에 대해 사용자들이 쉽게 사용하기 위해 단계별로 자습서 노트를 정리해보자! 



## 1. LangChain 과 통합 

* 정의:  초대형 언어 모델(LLM)을 앱에 쉽게 통합할 수 있도록 도와주는 Python/JavaScript 기반 오픈소스 프레임워크

* Langchain 주요 특징

  | 항목                                 | 설명                                                         |
  | ------------------------------------ | ------------------------------------------------------------ |
  | **체인(Chain)**                      | 여러 LLM 호출 및 작업을 순차적으로 연결해 복잡한 워크플로우 구성 가능 |
  | **프롬프트 템플릿(Prompt Template)** | 다양한 입력값을 받아 재사용 가능한 프롬프트 작성 가능        |
  | **에이전트(Agent)**                  | 도구(tool) 사용과 결정을 LLM에게 위임하여 **동적 작업 수행** 가능 |
  | **메모리(Memory)**                   | 이전 대화를 기억해 문맥을 유지하는 **대화형 앱** 제작 가능   |
  | **도구 통합(Tools Integration)**     | 검색(Search), 계산기, 데이터베이스, API 등 다양한 외부 도구 연결 가능 |
  | **Document QA**                      | PDF, 노션, 웹페이지 등에서 문서를 불러와 **LLM을 통한 질의응답** 가능 |

* 예제: 로컬 모델을 사용하여 한 언어에서 다른 언어로 텍스트를 번역하는 번역 애플리케이션 개발

  

  * Python 패키지 설치

    ```Bash
    pip install langchain[openai] 
    pip install foundry-local-sdk
    ```

  * `foundry_langchain.py` 소스 파일 분석

    ```python
    import os
    from langchain_openai import ChatOpenAI
    from langchain_core.prompts import ChatPromptTemplate
    from foundry_local import FoundryLocalManager
    
    
    # 별칭(alias) 사용. 사용 가능한 모델 목록을 확인하려면,  foundry model list
    alias = "phi-3-mini-4k"
    
    # FoundryLocalManager 인스턴스 생성 
    # Foundry Local 서비스가 아직 실행 중이 아니라면 서비스를 시작하고, 지정된 모델을 로드함
    manager = FoundryLocalManager(alias)
    
    # ChatOpenAI를 로컬 PC/Mac에서 실행 중인 모델을 사용하도록 구성함
    llm = ChatOpenAI(
        model=manager.get_model_info(alias).id,
        base_url=manager.endpoint, 
        api_key=manager.api_key,
        temperature=0.8,
        streaming=False
    )
    
    # 번역 프롬프트 템플릿 생성
    prompt = ChatPromptTemplate.from_messages([
        (
            "system",
            "You are a helpful assistant that translates {input_language} to {output_language}."
        ),
        ("human", "{input}")
    ])
    
    # 프롬프트를 언어 모델에 연결하여 간단한 체인 생성
    chain = prompt | llm
    
    input = "J'aime coder."
    print(f"'{input}'를 우리나라 말로 번역중...")
    
    # 입력값을 사용해 체인 실행
    ai_msg = chain.invoke({
        "input_language": "French",
        "output_language": "Korean",
        "input": input
    })
    
    # 결과 컨텐트 출력
    print(f"응답 결과: {ai_msg.content}")
    ```

  * `python foundry_langchain.py` 실행

    ![그림 1 - foundry_lanchain.py 실행](/../images/2025-06/foundry-local-12.png)

* Foundry Local 실행 장점
  * **사용자의 하드웨어에 가장 적합한 모델 버전을 자동으로 선택**해줌
    * 사용자가 **GPU**를 보유한 경우 → **GPU용 모델**을 다운로드하고,
    * 사용자가 **NPU(신경처리장치)**를 보유한 경우 → **NPU용 모델**을 다운로드하며,
    * **GPU나 NPU가 없는 경우**에는 → **CPU용 모델**을 다운로드함



## 2. Open WebUI 설치 및 사용 

* 정의
  * 로컬 또는 클라우드에서 실행되는 오픈소스 기반의 LLM 사용자 인터페이스
  * 웹 브라우저를 통해 LLM과 채팅하거나 명령을 실행할 수 있게 해주는 도구
  * 자체 호스팅 가능하며, 프라이버시와 보안에 유리
  * 멀티 사용자 지원, 다중 채팅 세션, 플러그인 기능 등 제공

* 주요 특징

  * Open-source ChatGPT 스타일 웹 UI
  * **LocalAI**, **Ollama**, **LM Studio** 등과 연동 가능
  * 자체 호스팅 가능하며, 프라이버시와 보안에 유리
  * 멀티 사용자 지원, 다중 채팅 세션, 플러그인 기능 등 제공
  * 웹 브라우저에서 실행되며 직관적인 인터페이스
  * `Docker`, `pip`, `ollama` 등 다양한 방식으로 설치 가능
  * 로컬에서 작동하므로 API Key 없이도 모델 실행 가능 (Ollama 등과 연동 시)

* 깃허브 소스: [https://github.com/open-webui/open-webui](https://github.com/open-webui/open-webui)

* Open WebUI 사용하려면, 먼저 open-webui 설치해야 한다.

  ```bash
  pip install open-webui
  ```

  ![그림 2 - open webui 설치](/../images/2025-06/foundry-local-13.PNG)

* Open WebUI가 설치완료 하면,  `open-webui servee` 명령어로 open-webui 를 실행할 수 있다.  그러면, 아래의 그림과 같이 `http://localhost:8080/auth?redirect=%2F` 를 자동적으로 실행한다. 그렇지 않으면 수동으로 웹 브라우저를 띄워 웹 주소를 넣어 주면 된다. 

  ![그림 3 - OpenWebUI 초기화면](/../images/2025-06/foundry-local-14.PNG)

* 그런 후, 위의 그림에서 Get started 를 마우스로 클릭하면, Open WebUI 메인으로 동작한다.

  ![그림 4 - OpenWebUI 메인 화면](/../images/2025-06/foundry-local-15.PNG)

*  `Select a model` 를 선택하면,  `Phi-3-mini-4k-instruct-generic-cpu` 모델을 선택한다.

  ![그림5 - 모델 선택](/../images/2025-06/foundry-local-16.PNG)

* 이제 프롬프트 창에서 ,  `what is Foundry Local?` 입력하면 다음과 같은 결과가 나타난다.

  ![그림6 - 프롬프트 실행](/../images/2025-06/foundry-local-17.PNG)



## 3. 참고

* 깃허브 소스: foundry_langchain.py

  