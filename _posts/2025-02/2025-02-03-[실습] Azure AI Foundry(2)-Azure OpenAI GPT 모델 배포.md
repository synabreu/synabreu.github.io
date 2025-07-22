---
title: "Azure OpenAI(2)-GPT 모델 배포"
date: 2025-02-03
tags: [마이크로소프트, Microsoft, Build 2025, Azure AI Foundry, Azure, Azure AI Foundry SDK, Azure OpenAI Studio, Azure OpenAI Service, Azure Machine Learning, Azure App Service, Azure Key Vault, Azure Monitor]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

Azure OpenAI 에서 리소스 생성을 완료 했다면, Azure AI Foundry를 통해 GPT 모델을 배포하는 방법을 노트에 정리해 본다. 



## 1. 코리아 센트럴에서 배포 가능한 Azure OpenAI 모델

| 모델 종류                                      | 배포 가능 여부 | 비고                                                         |
| ---------------------------------------------- | -------------- | ------------------------------------------------------------ |
| **GPT-3.5 Turbo**                              | 가능           | 대부분 지역에서 지원되며, 서울 센트럴에서도 사용 가능        |
| **Text-Davinci-003** (GPT-3 계열)              | 가능           | 여전히 일부 응답에 사용됨                                    |
| **Embedding 모델** (`text-embedding-ada-002`)  | 가능           | 검색 및 RAG에 유용                                           |
| Codex (`code-davinci-002`, `code-cushman-001`) | 제한적         | 일부 지역에선 종료 예정                                      |
|                                                |                |                                                              |
| Whisper                                        | 미지원         | 음성 인식                                                    |
| o3 2025-04-16                                  | 지원           | 입력: ₩2,713.50<br/>캐시된 입력: ₩678.38<br/>출력: ₩10,854.00 |
| o4-mini 2025-04-16                             | 지원           | Input: ₩1,492.43<br/>Cached Input: N/A<br/>Output: ₩5,969.70 |
| GPT-4.1-2025-04-14                             | 지원           | Input: ₩2,713.50<br/>Cached Input: ₩678.38<br/>Output: ₩10,854.00 |
| GPT-4.5-Preview-2025-02-27 글로벌              | 지원           | 입력: ₩101,756.25<br/>캐시된 입력: ₩50,878.13<br/>출력: ₩203,512.49 |
| GPT-4 / GPT-4 Turbo                            | 미지원         | 2025년 현재 한국 리전에서는 미지원                           |
| Whisper                                        | 미지원         | 음성 인식                                                    |
| Sora                                           | 미지원         | 멀티모달                                                     |

* 한국에서 GPT-4를 사용하고 싶다면 지연(latency)을 감수하고 일본 동부 리전(japaneast)으로 배포해야 한다. 

* 자세한 가격 및 지원 정보: [https://azure.microsoft.com/ko-kr/pricing/details/cognitive-services/openai-service/](https://azure.microsoft.com/ko-kr/pricing/details/cognitive-services/openai-service/)

  

## 2. GPT 모델 배포

* GPT 모델을 배포하려면,`[Go to Azure AI Foundry Portal]`가야한다.

  ![그림1 - Go To Azure AI Foundry Portal](/../images/2025-02/AzureModel-01.png)

* 새로운 탭이 생성되면서 Azure AI Foundry 웹사이트로 이동한다. 여기에서 `Create a deployment`를 선택한다.  

  ![그림2 - Create a deployment](/../images/2025-02/AzureModel-02.png)

* 수많은 OpenAI의 모델들 중에 검색 창에서 `gpt-35-turbo` 모델을 찾은 후 선택한다. 

  ![그림3 - Select a Model](/../images/2025-02/AzureModel-03.png)

* `gpt-35-turbo` 모델 명을 확인 한 다음, 이제 이 모델을 배포하기 위해  `[Deploy to selected resource]` 버튼을 클릭한다.

  ![그림4 - Deploy to selected resource](/../images/2025-02/AzureModel-04.png)

* gpt-35-turbo 모델 배포가 끝나면, API Key 와 HTTP 엔드포인트(Endpoint) 가  아래 그림과 같이 생성된다.

  ![그림5 - API Key](/../images/2025-02/AzureModel-05.png)

  

## 3. Chat playground 에서 GPT 모델 사용하기

* `Models+Endpoints` 메뉴에서는 그동안 배포한 모델 종류를 한 눈에 볼 수 있다. 

  ![그림6 - 모델 리스트](/../images/2025-02/AzureModel-06.png)

* `Playgrounds` 메뉴에서는 텍스트, 비디오, 이미지, 오디오 등 다양한 플레이그라운드에서 프롬프트를 통해 명령어를 채팅 창에서 실행할 수 있다. 

  ![그림7 - Playgrounds](/../images/2025-02/AzureModel-07.png)

* Deployment 에서`gpt-35-turbo` 모델이 선택된 것을 확인했다면, 여러분들이 궁금한 사항들을 채팅 창에서 입력하기 바란다. 예) 한국 센트럴에서 지원하는 OpenAI의 GPT 모델은 무엇인가?  

  ![그림8 - Deployment](/../images/2025-02/AzureModel-08.png)

* 또한, 위의 그림에서 `[View code]`를 클릭하면, 아래와 같이 Sample Code를 보여주는 데, 좀더 상세하게 소스 코드를 보고 싶으면, `[Open in VS Code]` 버튼을 클릭하면 된다.

  ![그림9 - Sample Code](/../images/2025-02/AzureModel-09.png)

* 그러면, 새로운 Visual Studio Code Online가 새 탭으로 자동적으로 생성되고, 프로젝트에서 소스 코드들이 생성된다. 아래의 그림을 보면, `sh install sh` 명령어가 자동적으로 실행된다. 

  ![](/../images/2025-02/AzureModel-10.png)

* `run_model.py` 파일 소스를 파악한다음, 명령 창에서 `python3 run_model.py`실행하면, JSON 형식으로 답변이 나오는 것을 여러분들의 눈으로 확인할 수 있다. 

  ![](/../images/2025-02/AzureModel-11.png)
