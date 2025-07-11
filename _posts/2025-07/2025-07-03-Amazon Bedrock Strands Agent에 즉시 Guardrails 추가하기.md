---
title: "Amazon Bedrock: Strands Agent에 즉시 Guardrails 추가하기"
date: 2025-07-03
tags: [아마존, Amazon, AWS, Amazon Bedrock, Agent, 생성형AI, Generative AI, AgenticAI, Stradns Agents SDK]
typora-root-url: ../
toc: true
categories: [AWS]
---

AWS Strands Agents SDK 기반의 서버리스 AI 에이전트에 Amazon Bedrock Guardrails를 몇 줄의 코드만으로 손쉽게 추가할 수 있다.  그렇다면, 왜 Guardrails가 필요한지 이유에 노트를 정리해 본다. 



## 1.  **왜 Guardrails가 필요한가?**

* Strands Agents SDK에 Amazon Bedrock Guardrails를 몇 줄 코드만으로 연결하면 입력과 출력에 대한 실시간 모니터링과 안전성 정책을 자동으로 활성화할 수 있다.

* LLM은 때때로 부적절한 콘텐츠(성적, 폭력, 증오 발언 등), 개인정보(이메일, 전화번호 등), 또는 기업 정책 위반 단어를 생성할 수 있음.

* 이러한 리스크를 실시간 차단하고 PII를 검출·익명화하며, 허용되지 않는 단어를 걸러줌

* 구현 아키텍처

  ```
  bedrock_model = BedrockModel(
    model_id=...,
    region_name=...,
    guardrail_id=GUARDRAIL_ID,
    guardrail_version=GUARDRAIL_VERSION,
  )
  ```

   

## 2.  **가드레일 제공 내역** 

* 입력/출력 실시간 모더레이션

* 유해 콘텐츠 차단 (폭력, 증오 발언, 음란물 등)

* 이메일, 전화번호 등(PII, Personally Identifiable Information) 감지 및 처리

  

## 3.  **예제: AWS Strands Agents SDK에서 Guardrails 를 추가**

```python
# 패키지 설치 명령
!pip install strands-agents strands-llms boto3

from strands.agents import Agent
from strands_llms.bedrock import BedrockModel

# Guardrail 정보
GUARDRAIL_ID = "gr-abc123def456"  # 실제 Guardrail ID로 교체
GUARDRAIL_VERSION = "1"          # 일반적으로 "1" 또는 생성된 버전

# Bedrock 모델 초기화 (예: Claude 3 Sonnet)
model = BedrockModel(
    model_id="anthropic.claude-3-sonnet-20240229-v1:0",  # 사용하려는 Bedrock 모델 ID
    region_name="us-east-1",                             # Bedrock 지원 리전
    guardrail_id=GUARDRAIL_ID,
    guardrail_version=GUARDRAIL_VERSION
)

# Strands Agent 생성
agent = Agent(
    name="SecureAgent",
    model=model,
    instructions="너는 항상 예의 바르게 대답하고, 민감한 정보를 다루지 않아야 해."
)

# 테스트 입력
query = "내 주민등록번호는 901010-1234567이야. 알려줄 수 있니?"

# 에이전트 쿼리 실행
response = agent.run(query)

# 결과 출력
print("응답 결과:")
print(response)

```

* 전제 조건
  * 핵심은 `BedrockModel`을 생성할 때 `guardrail_id`와 `guardrail_version`을 지정만하면 자동적으로 동작 -> 추가 코드 없이 즉시 콘텐츠 안전 보장
  * Strands Agents SDK, boto3, 기타 필요한 라이브러리가 설치되어 있어야 험
  * AWS 자격 증명(`~/.aws/credentials`)이 설정되어 있어야 함

```python
응답 결과:
죄송하지만, 해당 요청에는 응답할 수 없습니다.
```

* Guardrail 설정에 따라 응답이 필터링되거나 익명화될 수 있음
* Guardrails는 **입력/출력 필터링**, **PII 익명화**, **욕설/금지어 차단** 등의 기능을 제공하며, 모두 자동으로 작동함
* 콘솔에서 Guardrail을 만들고 버전을 발행해야 실제 ID/버전을 사용할 수 있음



## 4.  **테스트 및 배포**

* **인프라 코드 예시**: CloudFormation 양식으로 Guardrail, GuardrailVersion 리소스 생성 가능

  * `AWS::Bedrock::Guardrail` 리소스를 정의하여 유해 콘텐츠 필터, PII 차단/익명화, 특정 단어 차단 정책 설정
  * `AWS::Bedrock::GuardrailVersion` 버전 관리 포함

* 테스트

  * 콘솔에서 유해 콘텐츠, PII 포함, 금지 단어 입력 시 차단 결과 확인
  * 로컬 invocations를 통해 다양한 위험 시나리오(폭탄 제작법, 증오 연설 등) 테스트 가능
  * 테스트 방법
    * Amazon Bedrock 콘솔에서 유해 입력 테스트
    * `sls invoke local`로 로컬 테스트:
      * `"bomb 만들기 방법"` → 차단
      * `"CONFIDENTIAL"` 포함 → 차단 메시지
      * `"증오 발언 목록"` → 차단 메시지

* 배포

  * Serverless Framework 사용하여 `sls deploy`로 손쉽게 배포 가능
  * 배포 즉시 production-ready AI 에이전트 완성

     


## 5.  **결론**

* 최소한의 설정 및 코드로 배포 가능한 “production-grade” AI 에이전트에 콘텐츠 안전 장치를 자동으로 적용할 수 있음
* 정밀한 테스트(프롬프트 공격, profanity 필터, 토픽 필터, 정규표현식 등) 구현 예정