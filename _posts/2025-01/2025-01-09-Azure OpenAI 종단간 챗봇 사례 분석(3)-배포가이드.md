---
title: "Azure OpenAI 종단간 챗봇 사례 분석(3)-배포가이드"
date: 2025-01-09
tags: [마이크로소프트, Microsoft, Build 2025, Azure AI Foundry, Azure, Azure AI Foundry SDK, Azure OpenAI Studio, Azure OpenAI Service, Azure Machine Learning, Azure App Service, Azure Key Vault, Azure Monitor]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

자! 이제 Azure OpenAI 종단 간 챗봇 사례 분석으로  **Azure 구독에 배포하고**,**배포한 내용을 직접 테스트**해보며, 마지막으로 **사용한 리소스를 정리(clean up)**하는 방법을 노트에 정리해 보겠다. 



## 1. 배포 하기 전 사전 요구 사항

* 애저 구독(Azure Subscription)
  * 이 배포에 사용되는 모든 리소스 공급자(Resource Providers)가 등록되어 있어야 함
    * `Microsoft.CognitiveServices`
    * `Microsoft.Insights`
    * `Microsoft.ManagedIdentity`
    * `Microsoft.OperationalInsights`
    * `Microsoft.Storage`
  * 선택한 리전(Region)에 사용할 수 있는 다음과 같은 할당량(Quota)이 있어야 함
    * **App Service Plans**: P1v3(AZ) 인스턴스 3개
    * **OpenAI 모델**: GPT-4o 모델 배포가 가능하며, 분당 **50,000 토큰(TPM)** 처리 용량
  * 배포를 수행하는 사용자는 구독 범위에서 다음 권한을 가지고 있어야 함
    * 새로 생성되는 리소스 그룹 및 리소스에 **Azure 역할을 할당할 수 있는 권한**
       (예: **User Access Administrator** 또는 **Owner**)
    * 삭제된 AI 서비스 리소스를 **완전히 삭제(purge)** 할 수 있는 권한
       (예: **Contributor** 또는 **Cognitive Services Contributor**)
  * Azure CLI 설치
    * WSL(Windows Subsystem for Linux) 환경에서 이 작업을 실행하는 경우, Azure CLI가 **WSL 안에 설치**되어야 하며,**Windows에 설치된 버전이 사용되지 않도록 주의**해야 함
    * `which az` 명령어 실행 시 출력이 `/usr/bin/az`이어야 함



## 2. 애저 인프라스트럭처에 챗봇 배포

* 쉘(Shell)에서 이 리포지토리를 클론한 후, 리포지토리의 **루트 디렉터리(root directory)**로 이동하라.

```bash
git clone https://github.com/Azure-Samples/openai-end-to-end-basic
cd openai-end-to-end-basic
```

* 애저에 로그인하고, 여러분의 타겟 서브스크립을 설정하라.

```bash
az login
az account set --subscription <your-subscription-name>
```

* 배포 위치를 **구독에서 할당량(quota)이 충분한 리전**으로 설정하라.
  * 이 배포는 `australiaeast`, `eastus`, `eastus2`, `francecentral`, `japaneast`, `southcentralus`, `swedencentral`, `switzerlandnorth`, `uksouth` 등 리전에서 테스트 됨
  * 다른 리전에서도 배포가 성공할 수 있지만, 위 리전에서는 정상 동작이 검증되었음

```bash
LOCATION=eastus2
```

* 이 솔루션에서 배포되는 Azure 리소스들의 이름에 사용될 기본 이름 값을 설정하라.

```bash
BASE_NAME=<base resource name, between 6 and 8 lowercase characters, all DNS names will include this text, so it must be unique.>
```

* 리소스 그룹을 생성하고 인프라를 배포하라
  * 이 배포에는 선택적인 추적 ID(tracking ID)가 포함되어 있습니다. 해당 기능을 사용하지 않으려면, 아래 배포 코드에 다음 매개변수를 추가하라.  **`-p telemetryOptOut true`**

```bash
RESOURCE_GROUP=rg-chat-basic-${BASE_NAME}
az group create -l $LOCATION -n $RESOURCE_GROUP

PRINCIPAL_ID=$(az ad signed-in-user show --query id -o tsv)

# This takes about 10 minutes to run.
az deployment group create -f ./infra-as-code/bicep/main.bicep \
  -g $RESOURCE_GROUP \
  -p baseName=${BASE_NAME} \
  -p yourPrincipalId=$PRINCIPAL_ID
```



## 3. Azure AI Foundry Agent 서비스에 에이전트 배포

* 주요 개요
  * 챗봇 시나리오를 테스트하기 위해 **AI 에이전트**를 배포하게 됨
  * 에이전트는 **GPT 모델**과 함께 **Bing 검색을 통한 그라운딩 데이터**를 사용하는 구조
  * AI 에이전트를 배포하려면 **Azure AI Foundry의 데이터 플레인 접근 권한**이 필요함
  * 챗봇 아키텍처에서는 인터넷을 통해 **Azure AI Foundry 포털과 리소스**에 접근
* 배포 순서
  * **Azure 포털에 접속**하여 본인의 구독을 연다.
  * 리소스 그룹 내에서 `projchat`이라는 이름의 **Azure AI Foundry 프로젝트**로 이동함
  * **[Go to Azure AI Foundry portal]** 버튼을 클릭하여 Azure AI Foundry 포털을 염.
    * 이 버튼을 클릭하면 자동으로 `'Chat project'`로 이동
    *  https://ai.azure.com 에서 모든 Foundry 계정과 프로젝트에 접근할 수 있으므로, Azure 포털을 통해 접근할 필요는 없음
  * 왼쪽 사이드 메뉴에서 **[Agents]** 를 클릭함
  * 상단의 **[+ New agent]** 버튼을 클릭함
  * **Setup 패널**에서 에이전트 이름을 **'Baseline Chatbot Agent'** 로 변경함
  * **Knowledge 섹션**에서 **[+ Add]** 버튼을 클릭함
  * 팝업 창에서 **지식 유형(Knowledge type)** 으로 **‘Grounding with Bing Search’** 를 선택함
  * 기존 연결 중에서 이름이 **‘bingaiagent’** 인 항목을 선택한 후,**[Connect]** 버튼을 클릭함



## 4. Azure AI Foundry 포털의 Playground에서 에이전트 테스트

* 개요: **Azure AI Foundry 포털의 Playground 기능**을 통해 오케스트레이션 에이전트를 직접 호출하여 테스트해 봄
* 테스트 절차
  * **[Try in playground]** 버튼을 클릭함
  * **최근 인터넷 콘텐츠를 기반으로 그라운딩 데이터가 필요한 질문**을 입력함
    * "오늘 서울 날씨 어때?", "최근에 있었던 주요 뉴스는 무엇인가요?"
  * 질문을 입력하면, UI 상에 **그라운딩된 응답**이 나타남. -> 이 응답은 Bing 검색을 기반으로 최신 데이터를 활용해 생성된 결과

