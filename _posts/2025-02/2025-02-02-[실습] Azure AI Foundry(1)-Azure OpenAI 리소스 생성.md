---
title: "[실습] Azure AI Foundry(1)-Azure OpenAI 리소스 생성"
date: 2025-02-02
tags: [마이크로소프트, Microsoft, Build 2025, Azure AI Foundry, Azure, Azure AI Foundry SDK, Azure OpenAI Studio, Azure OpenAI Service, Azure Machine Learning, Azure App Service, Azure Key Vault, Azure Monitor]
typora-root-url: ../
toc: true
categories: [Microsoft, Azure]
---

지난 달에 Azure AI Foundry 챗봇 베이직 아키텍처와 종단간 챗봇 사례 분석들을 했다. 그렇다면, 이제 실제로 애저 포털에서 Azure OpenAI 리소스를 만드는 실습을 노트에 정리하고자 한다. 그렇게 하기 위해서는 먼저 **애저 계정과 구독(subscription)을 생성**해야 함을 잊지 말기를 바란다. 



<img src="/../images/2025-02/AzureOAI-01.png" alt="그림1 - 구독 정보"  />



이번 노트는 Azure OpenAI 를 시작하는 방법을 정리해보겠다. 



## 1. Azure Portal 에서 Azure OpenAI 리소스 생성

* Azure OpenAI 액세스 신청 관련 선수 사항

  * 원래 Azure OpenAI는 다른 애저 서비스들과 다르게 리소스 생성을 위해 사전에 승인이 필요했다. 2025년 1월 부터는 이용 약관과 행동 강령에 위배되지 않는 한 별도의 승인을 할 필요없이 오픈AI에서 발표한 모델을 쓸 수 있다.

* 웹 브라우저를 실행해서 `https://portal.azure.com`를 입력해서 로그인 한 후 애저 포털 화면으로 이동한다.

  ![그림2 - Azure Portal](/../images/2025-02/AzureOAI-02.png)

* 애저 포털 화면 최상단에 있는 검색 창에 `azure openai`를 입력한다. `서비스`항목 중 `[Azure OpenAI]`가 나오면 클릭한다.

  <img src="/../images/2025-02/AzureOAI-03.png" />

* Azure OpenAI의 리소스 관리 화면으로 이동하면 좌측 상단의 `만들기' 또는 'Azure OpenAI 만들기` 버튼을 클릭한다. 

  ![그림4-Azure OpenAI 만들기](/../images/2025-02/AzureOAI-04.png)



## 2. Azure OpenAI 생성(1)-기본 사항

* `구독', `리소스 그룹`, `지역`, `이름`, `가격 책정 계층`을 작성하고 `[다음]`을 클릭한다.

  ![그림5 - 기본사항](/../images/2025-02/AzureOAI-05.png)

  | 세부 정보 | 설정 항목      | 설명                                                         |
  | --------- | -------------- | ------------------------------------------------------------ |
  | 프로젝트  | 구독           | 생성한 구독을 지정한다.                                      |
  |           | 리소스 그룹    | 여러 리소스의 비용 관리 및 일괄 처리를 위한 그룹. Azure OpenAI만을 사용하는 경우라면 괜찮다. 현재 작성한 리소스 그룹이 없다면 `새로 만들기`를 클릭하고 원하는 리소스 그룹 이름을 입력한다. |
  | 인스턴스  | 지역           | 지역 이름을 입력한다. 지역이란 해당 Azure OpenAI 리소스(예: GPT-4 API)가 실제로 호스팅되는 데이터 센터 위치를 말하며, OpenAI 모델이 배포되어 사용 가능한 Azure 데이터 센터로 일반적인 Azure 서비스의 지역 선택과 다르다. 2025년 기준, 한국(Korean Central)은 현재 일부 모델만 제한적으로 지원한다. |
  |           | 이름           | 유일한 인스턴스 이름을 적는다. 만일 다른 이름이 중복되어 있으면 시스템에서 적절한 메시지를 준다. 그것을 보고 지정하면 된다. 필자는 `synabreu-azure-open-test01`로 입력했다. 여러분들은 다른 이름으로 지정하기 바란다. |
  |           | 가격 책정 계층 | Azure OpenAI 인스턴스 세부정보에서 나오는 **"가격 책정 계층(Pricing tier)"**은 일반적인 Azure 리소스의 경우와 달리, **Azure OpenAI 서비스에서는 고정되어 있으며 사용자가 선택할 수 없음**. Azure OpenAI의 가격 책정은 사용량 기반(pay-as-you-go). 가격 책정 계층(Tier)은 "Standard"로 **자동 설정**됨. |

* 2025년 기준 Azure OpenAI 지원 지역: Azure OpenAI는 모든 지역에서 사용 가능하지 않습니다. 사용 가능한 지역만 선택할 수 있음

  | 지역 이름              | 지역 코드        | 국가                                                         |
  | ---------------------- | ---------------- | ------------------------------------------------------------ |
  | East US                | `eastus`         | 미국                                                         |
  | South Central US       | `southcentralus` | 미국                                                         |
  | West Europe            | `westeurope`     | 네덜란드                                                     |
  | France Central         | `francecentral`  | 프랑스                                                       |
  | UK South               | `uksouth`        | 영국                                                         |
  | Sweden Central         | `swedencentral`  | 스웨덴                                                       |
  | Canada East            | `canadaeast`     | 캐나다                                                       |
  | Australia East         | `australiaeast`  | 호주                                                         |
  | Japan East             | `japaneast`      | 일본                                                         |
  | Korea Central (제한적) | `koreacentral`   | 대한민국 (2025년 현재 일부 모델만 제한적으로 지원될 수 있음) |

* 선택 기준
  * **지연 시간(Latency)**: 사용자 또는 애플리케이션 서버와 가까운 지역을 선택
  * **데이터 주권(Data residency)**: 기업/정부 정책에 따라 특정 국가 또는 지역 내에 데이터가 존재해야 하는 경우
  * **고가용성 구성**: 여러 리전을 사용하는 재해 복구(DR) 아키텍처 구성 시 필요



## 3. Azure OpenAI 생성(2)-네트워크

* 네트워크 부분은 Azure OpenAI의 네트워크 보안을 설정한다. 기본적으로 `인터넷을 포함한 모든 네트워크가 이 리소스에 액세스 사용할 수 있습니다.`가 선택되어 있는 데, 그대로 사용하면 된다. 

  ![그림6 - 네트워크 선택](/../images/2025-02/AzureOAI-06.png)

  | 항목                            | 모든 네트워크 허용(Public access from all networks) | 선택한 네트워크만 허용(Selected networks) | Private Endpoint 전용(Private endpoint only) |
  | ------------------------------- | --------------------------------------------------- | ----------------------------------------- | -------------------------------------------- |
  | **접근 가능 범위**              | 모든 인터넷 및 네트워크                             | 허용된 IP, VNet/Subnet만                  | VNet 내부에서만 접근 가능                    |
  | **보안 수준**                   | 낮음                                                | 중간                                      | 매우 높음                                    |
  | **API Key 외 인증 필요**        | 없음                                                | 네트워크 제한 필요                        | Private Endpoint 및 DNS 설정 필요            |
  | **사용 용도**                   | 테스트, 개발 환경                                   | 기업 내부 서비스, 제한적 접근             | 고보안 환경 (금융, 정부, 의료 등)            |
  | **인터넷에서의 접근**           | 가능                                                | 일부 가능 (허용된 IP/VNet만)              | 절대 불가                                    |
  | **Azure VNet 연동**             | 불필요                                              | 선택 가능                                 | 필수 (Private Link 필요)                     |
  | **Azure Private DNS 필요 여부** | 없음                                                | 없음                                      | 필요                                         |
  | **예시 시나리오**               | 단기 PoC, 빠른 API 실험                             | 특정 사무실 IP, 사내망 기반 API 호출      | 내부 시스템에서만 GPT 호출, 외부 차단        |
  | **Zero Trust 보안 전략**        | 비적합                                              | 적합                                      | 강력 추천                                    |
  | **설정 복잡도**                 | 매우 낮음                                           | 보통                                      | 높음 (DNS, Private Link 구성 필요)           |

* 선택 가이드
  * 옵션1 : **인터넷을 포함한 모든 네트워크**가 이 리소스에 액세스할 수 있습니다. -> 간단한 테스트나 개발용. 보안 민감하지 않은 환경에서 사용.
  * 옵션2 : 선택한 네트워크만 허용, Azure AI 서비스 리소스에 대한 네트워크 보안을 구성합니다. -> **기업 환경에서 외부 접근을 제한하고 싶을 때 적합.** IP/VNet 기반 제어.
  * 옵션3 : **비활성화됨,** 이 리소스에 액세스할 수 있는 네트워크가 없습니다. 이 리소스에 액세스하는 독점적인 방법이 될 개인 끝점 연결을 구성할 수 있습니다. -> 고보안 환경 필요 시 필수. **인터넷 완전 차단**, Private VNet 내부 전용.



## 4. Azure OpenAI 생성(3)-Tags

* 리소스에 대해 태그를 지정할 수 있다. 여러 리소스를 관리할 때, 태그를 붙히면 쉽게 구분가능하고 검색할 수 있다. 아래의 그림에서는 아무것도 변경하지 않고, 리소스 이름에 `Azure AI Foundry` 그대로 두고 `다음`버튼을 누른다.

  ![그림7 - 태그](/../images/2025-02/AzureOAI-07.png)



## 5. Azure OpenAI 생성(4)-검토+제출

* 검토 및 제출 화면에서는 지금까지 설정한 항목을 검토할 수 있도록 한 눈에 볼 수 있다. 지금까지 설정한 것이 맞다면, `[만들기]`버튼을 클릭하면 된다. 

  <img src="/../images/2025-02/AzureOAI-08.png" />





## 6. Azure OpenAI 생성(5)-배포 생성 완료

* 모든 Azur OpenAI 생성 설정이 끝났다면, 이제 아래와 같이 `배포 전송 중`이라는 메시지가 나온다. 

  ![그림9 - 배포 전송 중](/../images/2025-02/AzureOAI-09.png)

* 배포가 완료 되었다면, `[리소스로 이동]` 버튼을 누른다. 

  ![그림10 - 리소스로 이동](/../images/2025-02/AzureOAI-10.png)

* `synabreu-azure-openai-test01` 리소스에 대해 한 눈에 살펴 볼 수 있다. 

  ![그림11 - 리소스](/../images/2025-02/AzureOAI-11.png)