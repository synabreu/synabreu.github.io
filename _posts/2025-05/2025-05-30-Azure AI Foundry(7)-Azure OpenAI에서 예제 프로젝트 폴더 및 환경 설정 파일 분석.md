---
title: "Azure AI Foundry(7)-Azure OpenAI에서 예제 프로젝트 폴더 및 환경 설정 파일 분석"
date: 2025-05-30
tags: [마이크로소프트, Microsoft, Build 2025, Azure AI Foundry, Azure, Azure AI Foundry SDK, Azure OpenAI Studio, Azure OpenAI Service, Azure Machine Learning, Azure App Service, Azure Key Vault, Azure Monitor]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

지난번 Azure AI Foundry 에서 Azure OpenAI 를 사용해서 `gpt-35-turbo`  모델을 배포할 때, 소스 코드를 볼 수 있는`[View code]`부분에서 `[Open in VS Code]` 버튼을 누르면 Visual Studio Code 온라인으로 연결해서 소스 코드를 볼 수 있다. 그러한 코드들이 어떠한 것들이 있으며, 어떤 코드가 포함되는 지 전체 프로젝트 폴더와 환경 설정 파일(env)를 한번 분석해 보자!



## 1. VS Code에서 보이는 프로젝트 폴더 구조

```bash
AZUREOPENAI-GPT35/
└── azuredev-8d0d/
    ├── .env
    ├── .gitignore
    ├── install.sh
    ├── INSTRUCTIONS.md
    ├── requirements.txt
    └── run_model.py

```

* `.env`: 환경 변수 파일
* `.gitignore`: Git에서 무시할 파일 설정
* `install.sh`: 설치용 셸 스크립트
* `INSTRUCTIONS.md`: 프로젝트 사용 설명서
* `requirements.txt`: 필요한 Python 패키지 목록
* `run_model.py`: 모델 실행을 위한 Python 스크립트



## 2. 환경 변수 파일

```bash
AZURE_ENV_NAME="chat-playground-6772"
AZURE_LOCATION="eastus2"
AZURE_SUBSCRIPTION_ID="<YOUR_AZURE_SUBCRIPTION_ID>"
AZURE_EXISTING_AIPROJECT_ENDPOINT="https://synabreu-2461-resource.cognitiveservices.azure.com/"
AZURE_EXISTING_AIPROJECT_RESOURCE_ID="<YOUR AZURE_EXISTING_AIPROJECT_RESOURCE_ID>"
AZD_ALLOW_NON_EMPTY_FOLDER=true
```



## 3. `AZURE_ENV_NAME` 환경 변수

* `AZURE_ENV_NAME="chat-playground-6772"`

  *  **Azure Developer CLI (azd)** 프로젝트에서 사용되는 **환경 이름(Environment Name)** 을 정의하는 환경 변수


  *  이 값은 Azure 리소스를 생성하고 관리하는 데 중요한 역할을 함


  *  `AZURE_ENV_NAME`은 `azd` 명령어를 사용할 때 지정하는 **Azure 환경 이름**이며, 해당 환경 이름은 Azure 리소스를 **그룹화하고 구분**하는 데 사용됨. ex) `.azure` 폴더나 `.env` 파일, 또는 `azure.yaml`과 관련된 설정 파일에서 참조됨


* `"chat-playground-6772"`는 사용자가 정의한 프로젝트 이름 또는 테스트/샌드박스용 이름으로 하나의 프로젝트에서 `dev`, `test`, `prod`와 같이 서로 다른 환경을 `AZURE_ENV_NAME`으로 구분 가능.

  ```bash
  azd init --environment chat-playground-6772
  azd up
  ```

  * Azure에 리소스 그룹, App Service, Cognitive Services 등 모든 관련 리소스가 `"chat-playground-6772"` 환경 이름을 기준으로 구성함



## 3. `AZURE_LOCATION` 환경 변수

* `AZURE_LOCATION="eastus2"`

  * Azure에서 리소스를 **어느 물리적 지역(Region)** 에 생성할지를 지정하는 환경 변수. 이 값은 **성능**, **비용**, **데이터 거버넌스** 등에 영향을 줌

  * `AZURE_LOCATION`은 `azd`, ARM 템플릿, Bicep, Terraform 등의 자동화 도구에서 **리소스를 배포할 위치**를 정의

* `"eastus2"`는 **미국 동부 지역 중 하나**로, 버지니아(Virginia)에 위치한 Azure 데이터 센터

  ```env
  AZURE_LOCATION="eastus2"
  ```

  * `.env` 파일에서 Azure location 정의로 eastus2 지정

  ```bash
  azd up
  # → Azure 리소스가 모두 eastus2 지역에 생성됨
  ```

* 주의 사항

  * 리전별로 사용 가능한 서비스가 다름. 예: `eastus2`는 Azure OpenAI, GPU VM, Azure AI Studio 등을 지원하지만 다른 리전은 제한됨	
  * **데이터 거버넌스 및 컴플라이언스** 요구 사항이 있는 경우 해당 국가나 기업의 정책에 맞는 지역을 선택해야 함
  * 같은 리전에 모든 리소스를 생성하면 **지연(latency)** 이 줄고, **데이터 전송 비용**도 절감됨



## 4. `AZURE_LOCATION` 환경 변수

* `AZURE_SUBSCRIPTION_ID="<YOUR_AZURE_SUBCRIPTION_ID>"`

  * Azure에서 리소스를 **청구 및 관리**할 때 사용하는 고유한 **구독 ID(Subscription ID)** 를 지정하는 환경 변수

  * `AZURE_SUBSCRIPTION_ID`

    * Azure 구독(Subscription)은 **Azure 리소스를 만들고 사용하는 단위**
    * 이 변수는 Azure Developer CLI(`azd`), ARM/Bicep 템플릿, Terraform, SDK 등에서 리소스를 배포하거나 연결할 때 **어느 구독 아래 작업할지**를 지정함
    * `<YOUR_AZURE_SUBCRIPTION_ID>`: 실제 구독 ID 예시 형식이며, 각 사용자는 Azure 포털 또는 CLI에서 본인의 고유 ID를 사용해야 함. 이 값은 공유되면 안되므로 <YOUR_AZURE_SUBCRIPTION_ID> 으로 표시하므로 실제 값과 다르다.

  * Azure Subscription ID가 중요한 이유

    * Azure에는 여러 구독이 있을 수 있기 때문에, **부서별, 프로젝트별, 환경별(dev/test/prod)** 로 구독을 나누는 것이 일반적이며, 특정 구독에만 리소스 생성 권한이 부여될 수도 있음. 그러므로  사용량 및 비용을 구독 단위로 관리함

      ```env
      AZURE_SUBSCRIPTION_ID="<YOUR_AZURE_SUBCRIPTION_ID>"
      ```

      * `azd up`, `az login`, `az deployment` 같은 명령어가 이 값을 사용해 리소스를 생성

      * 배포되는 모든 리소스는 이 ID에 연결된 구독 내에서 요금이 발생함

      * Azure CLI 에서 구독 ID 확인 방법

        ```bash
        az account list --output table
        # 또는 
        az account show --query id
        ```

      * Azure Portal 에서 구독 ID 확인 방법: https://portal.azure.com → 좌측 상단 "모든 서비스" → "구독(Subscriptions)" → ID 확인 가능

      * **권한 부여(RBAC)** 또는 **기업 내 구독 관리 전략**은 별도



## 5. `AZURE_EXISTING_AIRPROJECT_ENDPOINT` 환경 변수

* `AZURE_EXISTING_AIPROJECT_ENDPOINT="https://synabreu-2461-resource.cognitiveservices.azure.com/"`

  * Azure에서 이미 만들어진 **AI 서비스 인스턴스(예: Azure OpenAI, Cognitive Services 등)** 의 **엔드포인트(URL 주소)** 를 지정하는 환경 변수

  * `AZURE_EXISTING_AIPROJECT_ENDPOINT`

    * Azure의 **Cognitive Services 또는 Azure OpenAI 리소스**에 접근하기 위한 **HTTP API 엔드포인트**

    * 이 URL을 통해 애플리케이션이 Azure의 AI 서비스(예: GPT-3.5, GPT-4, 언어 분석, 이미지 분석 등)를 호출

    * 일반적으로 Azure 리소스를 생성할 때 자동으로 부여됨

    * 예시 분석

      ```env
      AZURE_EXISTING_AIPROJECT_ENDPOINT="https://synabreu-2461-resource.cognitiveservices.azure.com/"
      ```

      * `https://` : HTTPS 프로토콜
      * `synabreu-2461-resource` : 사용자가 생성한 Cognitive Services 리소스 이름
      * `cognitiveservices.azure.com` : Azure Cognitive Services의 도메인
      * 전체 URL은 이 리소스에 대한 **공용 REST API 접속 주소**	

* 엔드포인트 사용처

  | 사용처                              | 설명                                                         |
  | ----------------------------------- | ------------------------------------------------------------ |
  | Python SDK (`openai`, `azure-ai-*`) | GPT, 번역, 요약, 이미지 분석 등의 요청을 이 URL로 전송       |
  | REST API                            | 직접 HTTP 요청을 보낼 때 사용                                |
  | `.env` 파일                         | 런타임에서 설정을 주입하여 동적으로 호출 URL을 결정          |
  | `azd` 프로젝트                      | 이미 존재하는 AI 리소스를 사용할 때 재생성 없이 연결하기 위해 사용됨 |

* 엔드포인트 확인 방법

  * Azure Portal: 리소스 그룹 > `synabreu-2461-resource` > **“엔드포인트” 탭** 클릭

  *  **Azure CLI**

    ```bash
    az cognitiveservices account show \
      --name synabreu-2461-resource \
      --resource-group rg-synabreu-2461 \
      --query "endpoint" -o tsv
    ```

    * 이 엔드포인트만으로는 호출 불가 — 반드시 **API Key 또는 Azure AD 토큰**이 함께 필요
    * `.env` 파일에 저장할 때는 Git 등에 **절대 노출하지 않도록 주의**해야 함



## 6. `AZURE_EXISTING_AIPROJECT_RESOURCE_ID` 환경 변수

* `AZURE_EXISTING_AIPROJECT_RESOURCE_ID="<YOUR AZURE_EXISTING_AIPROJECT_RESOURCE_ID>"`

  * Azure에서 **이미 생성된 AI 프로젝트 리소스의 전체 리소스 경로(Resource ID)** 를 지정하는 환경 변수

  * `azd` 또는 Azure 자동화 도구가 **해당 리소스를 참조하거나 연결**할 때 사용함

* 변수 구성 설명

  ```bash
  /subscriptions/<YOUR AZURE_EXISTING_AIPROJECT_RESOURCE_ID>/resourceGroups/rg-synabreu-2461/providers/Microsoft.CognitiveServices/accounts/synabreu-2461-resource/projects/synabreu-2461
  ```

  * 보안상 중요하기 때문에 <YOUR AZURE_EXISTING_AIPROJECT_RESOURCE_ID> 로 표시하고,  Azure 리소스의 **정규 경로(Fully Qualified Resource ID)** 를 의미함. 

  | 구성 요소                                                    | 설명                                                         |
  | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | `/subscriptions/{subscription-id}`                           | 이 리소스가 속한 Azure 구독 ID                               |
  | `/resourceGroups/{resource-group}`                           | 리소스를 포함하고 있는 리소스 그룹 이름                      |
  | `/providers/Microsoft.CognitiveServices/accounts/{account-name}` | Cognitive Services 계정 (예: OpenAI, Vision, Speech 등)      |
  | `/projects/{project-name}`                                   | 해당 Cognitive Service 하위의 특정 프로젝트 (예: Azure AI Studio 프로젝트) |

  * Azure Developer CLI (`azd`) 또는 ARM/Bicep 템플릿을 사용해서 이미 존재하는 리소스를 **재사용**하고자 할 때 사용
  * **리소스를 새로 만들지 않고** 지정된 리소스를 연결하기 위해 필요
  * 특히 Azure AI Studio에서 만든 **AI 프로젝트 (예: chat playground, fine-tuning project)** 를 사용할 때 이 리소스 ID가 반드시 필요함

* 리소스 ID 확인 방법

  * Azure Portal: Cognitive Services 또는 Azure AI Studio에서 해당 프로젝트 선택 → `속성` 또는 `JSON 보기`에서 Resource ID 확인 가능

  * Azure CLI

    ```bash
    az resource show \
      --name <YOUR_RESOURCE_NAME> \
      --resource-group <YOUR_RESOURCE-GROUP> \
      --resource-type "Microsoft.CognitiveServices/accounts/projects" \
      --query "id" -o tsv
    ```

    * `--name`: <YOUR_RESOURCE_NAME>으로 지정됨. 예) synabreu-2461
    * `--resource-group`: <YOUR_RESOURCE-GROUP>으로 지정 됨. 예) rg-synabreu-2461
    * `--resource-type`은 실제 프로젝트 타입에 따라 다를 수 있으며, `az resource list`로 확인 필요



## 7. `AZD_ALLOW_NON_EMPTY_FOLDER` 환경 변수

* `AZD_ALLOW_NON_EMPTY_FOLDER=true`: **Azure Developer CLI (azd)** 를 사용할 때, 현재 디렉토리에 이미 파일이나 폴더가 있어도 프로젝트 초기화나 배포를 **허용**하도록 하는 **환경 변수**

  * 기본값: `false`.  Azure Developer CLI의 명령어들, 예: `azd init`, `azd up` 등은 **안전성 확보**를 위해
    * **현재 폴더가 비어 있지 않으면 실행을 중단**함
    *  기존 파일이 덮어쓰이거나 충돌하는 걸 **방지**하기 위해서

  * true 역할

    ```bash
    AZD_ALLOW_NON_EMPTY_FOLDER=true
    ```

    * 현재 폴더가 비어 있지 않아도 `azd` 명령어들을 **강제로 실행 가능**하게 만듦
    * 예: 이미 일부 코드가 있는 프로젝트 디렉토리에서 `azd init` 을 실행할 때 유용

    ```bash
    AZD_ALLOW_NON_EMPTY_FOLDER=true azd init
    ```

    *  `.env` 파일에 아래와 같이 설정

    ```env
    AZD_ALLOW_NON_EMPTY_FOLDER=true
    ```

  * 사용 시나리오

    | 상황                                                | 설명                     |
    | --------------------------------------------------- | ------------------------ |
    | 기존 프로젝트 폴더에 `azd` 프로젝트 구성 추가       | 초기화 및 배포 가능      |
    | Git으로 클론 받은 코드에 `azd` 설정을 입히려는 경우 | 안전하게 진행 가능       |
    | 기존 파일이 있어도 강제로 `azd up` 실행             | 가능하지만 **주의 필요** |

    * 주의 사항
      * **기존 파일이 덮어쓰기** 될 수 있으므로, 반드시 **백업 또는 Git 커밋**을 권장
      * 특히 `.env`, `azure.yaml`, `.azure` 폴더 등이 덮어쓸 수 있음
      * **멀티 개발자 협업 환경**에서는 사용 여부를 명확히 하고, 일관된 설정이 필요
