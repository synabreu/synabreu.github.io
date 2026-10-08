---
title: "[실습] Decisions API 사용기-보험 심사(1)"
date: 2026-10-08
tags: [openai, devday, samaltman, chatgpt, codex, linux, codexmobile, meta, muse, spacexai, grotbot, dots, ChatGPT Spaces]
typora-root-url: ../
toc: true
categories: [openai]
---
지난 [오픈AI DevDay 키노트를 보고] 후기 글을 본 사람은 알것이다. Decisions API가 소개만 되었고 관련된 API를 직접 사용해 볼 수 없었다. 그러나 어제 Decisions API를 베타 버전으로 오픈AI가 전격적으로 공개했다. 그래서 첫편에서는 Decisions API를 먼저 활용하기 전에 OpenAI가 제공하는 베타 API에 대해 알아보았다.

---

# 1. Decisions API란 무엇인가?

Decisions API는 일반적인 “자유 형식 답변 생성”보다 정해진 후보 중 선택하거나, 참/거짓 가능성을 평가하거나, 등급을 매기는 판단 작업에 특히 잘 맞는다. [현재 공식 문서 기준](https://developers.openai.com/api/reference/resources/decisions/methods/create)으로 predicate, choice, score 세 종류의 질문을 한 요청에 함께 넣을 수 있고 결과로 확률/신뢰도까지 받을 수 있다.

| 유형                | 사용하는 경우                                | 질문 예시                                                            | 주요 반환값                                          |
| ------------------- | -------------------------------------------- | -------------------------------------------------------------------- | ---------------------------------------------------- |
| **predicate** | 하나의 조건이 참일 가능성을 판단할 때        | “이 사고에 부상 관련 검토가 필요한가?”                             | 참일 확률인`probability`                           |
| **choice**    | 정해진 후보 중 하나를 선택할 때              | “일반 심사·의료 심사·긴급 대응 중 어디로 보낼 것인가?”           | 선택값`choice`, 신뢰도 `confidence`, 후보별 확률 |
| **score**     | 순서가 있는 등급을 기준으로 수준을 평가할 때 | “이 사고의 긴급도는 낮음·보통·높음·매우 높음 중 어느 수준인가?” | 점수`score`, 신뢰도 `confidence`, 등급별 확률    |

---

# 2. decisions 생성

그러면 decisions API가 어떻게 구성되었는지 한번 알아보자!

```
curl https://api.openai.com/v1/decisions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "gpt-6-luna",
    "input": "The package arrived with a broken screen.",
    "questions": [
      {
        "type": "predicate",
        "name": "damaged",
        "instructions": "Does the customer report a damaged item?"
      }
    ]
  }'
```

이 엔드포인트는 동일한 입력 데이터에 대해 분류(Classification) 또는 점수 산정(Scoring) 질문을 요청할 때 사용한다. 응답 결과는 질문을 요청한 순서대로 반환된다.

텍스트 입력은 문자열(String) 형태로 전달할 수 있다. 또한 `input_text`와 `input_image` 요소가 포함된 사용자 메시지를 전송할 수 있으며, 요청당 최대 128개의 이미지를 포함할 수 있다. 이미지는 반드시 데이터 URL(Data URL) 형식이어야 하며, 외부 URL이나 파일 ID는 사용할 수 없다. 그 밖의 메시지 역할(Role), 함수 호출(Function Call), 파일, 오디오, 항목 참조(Item Reference)는 지원하지 않는다.

일부 질문에서는 정상적인 답변 대신 요청 거부(Refusal) 결과가 반환될 수 있다. 이 경우 결과의 `type`은 `refusal`로 표시되며, 질문에 지정된 이름이 함께 포함된다. 질문에 이름을 지정하지 않았다면 해당 값은 `null`로 반환된다.

# 3. Body Parameters - Decisions API 요청 파라미터

```json
{
  "model": "gpt-6-luna",
  "answers": [
    {"type": "predicate", "name": "damaged", "probability": 0.95}
  ],
  "usage": {
    "input_tokens": 42,
    "input_tokens_details": {"cached_tokens": 0, "cache_write_tokens": 0},
    "output_tokens": 0,
    "output_tokens_details": {"reasoning_tokens": 0},
    "total_tokens": 42
  }
}
```

## 3-1. input: string 또는 DecisionInputMessage 배열

모든 질문에서 평가할 텍스트 또는 이미지 데이터를 지정한다.

텍스트 문자열 또는 텍스트와 인라인 이미지가 포함된 사용자 메시지를 전달할 수 있다. 이미지는 반드시 인라인 데이터 URL 형식이어야 하며, 하나의 요청에 포함된 전체 메시지에서 최대 128개의 이미지를 사용할 수 있다.

외부 URL, 파일, 오디오, 도구(Tools), 항목 참조(Item References)는 지원하지 않는다.

다음 형식 중 하나를 사용한다.

- `string`: 문자열
- `array of DecisionInputMessage`: DecisionInputMessage 객체의 배열

## 3-2. DecisionInputMessage

content: string 또는 DecisionInputPart 배열

평가에 사용할 텍스트 데이터 또는 텍스트와 인라인 이미지 요소로 구성된 순서 있는 목록을 지정한다.

다음 형식 중 하나를 사용한다.

- `string`: 문자열
- `Parts`: DecisionInputPart 객체의 배열

DecisionInputPart

다음 두 가지 형식 중 하나를 사용한다.

### 1) DecisionInputText

| 속성          | 자료형           | 설명                                                |
| ------------- | ---------------- | --------------------------------------------------- |
| `text`      | string           | 입력할 텍스트 데이터를 지정한다.                    |
| `minLength` | 0                | 최소 문자열 길이는 0이다.                           |
| `maxLength` | 10485760         | 최대 문자열 길이는 10,485,760자이다.                |
| `type`      | `"input_text"` | 객체 유형을 나타내며 항상`input_text`를 사용한다. |

### 2) DecisionInputImage

인라인 이미지를 입력한다. 외부 URL과 파일 ID는 지원하지 않는다.

| 속성          | 자료형            | 설명                                                   |
| ------------- | ----------------- | ------------------------------------------------------ |
| `image_url` | string            | Base64로 인코딩된 데이터 URL 형식의 이미지를 지정한다. |
| `minLength` | 0                 | 최소 문자열 길이는 0이다.                              |
| `maxLength` | 1073741824        | 최대 문자열 길이는 1,073,741,824자이다.                |
| `type`      | `"input_image"` | 객체 유형을 나타내며 항상`input_image`를 사용한다.   |
| `detail`    | 선택 사항         | 이미지의 세부 처리 수준을 지정한다.                    |

`detail`은 선택한 모델의 이미지 처리 프로필에 따라 적용되며, 기본값은 `auto`다.

사용할 수 있는 값은 다음과 같다.

- `low`: 낮은 세부 수준
- `high`: 높은 세부 수준
- `auto`: 모델이 자동으로 결정
- `original`: 원본 이미지 수준
- `null`: 값을 지정하지 않음

role: "user"

메시지의 역할을 지정한다. 항상 `user`를 사용한다.

type: "message" (선택 사항)

메시지 객체의 유형을 지정한다.

## 3-3. model: string

사용할 AI 모델의 이름을 지정한다.

| 속성          | 값      |
| ------------- | ------- |
| 자료형        | string  |
| `minLength` | 0       |
| `maxLength` | 1048576 |

## 3-4. questions: 객체 배열

입력 데이터에 대해 수행할 질문을 지정한다.

질문은 다음 세 가지 유형으로 구성된다.

### 1) Predicate — 참일 확률 추정

입력 데이터에 대한 특정 진술이 참일 가능성을 추정한다.

| 속성             | 자료형            | 설명                                               |
| ---------------- | ----------------- | -------------------------------------------------- |
| `instructions` | string            | 평가할 진술이나 판단 기준을 지정한다.              |
| `minLength`    | 0                 | 최소 문자열 길이는 0이다.                          |
| `maxLength`    | 1048576           | 최대 문자열 길이는 1,048,576자이다.                |
| `type`         | `"predicate"`   | 객체 유형을 나타내며 항상`predicate`를 사용한다. |
| `name`         | string, 선택 사항 | 질문을 식별하기 위한 이름을 지정한다.              |

예를 들어 고객의 문의가 환불 요청인지, 문서에 개인정보가 포함되어 있는지, 특정 진술이 사실인지를 확률적으로 판단할 때 사용한다.

### 2) 4.2 Choice — 선택지 중 하나 선택

입력 데이터를 분석한 뒤 제공된 선택지 중 가장 적합한 항목을 선택한다.

| 속성             | 자료형              | 설명                                            |
| ---------------- | ------------------- | ----------------------------------------------- |
| `choices`      | 객체 배열           | 선택 가능한 항목을 지정한다.                    |
| `value`        | string 또는 boolean | 선택지의 값을 지정한다.                         |
| `description`  | string, 선택 사항   | 선택지에 대한 설명을 지정한다.                  |
| `instructions` | string              | 선택 기준이나 판단 지침을 지정한다.             |
| `type`         | `"choice"`        | 객체 유형을 나타내며 항상`choice`를 사용한다. |
| `name`         | string, 선택 사항   | 질문을 식별하기 위한 이름을 지정한다.           |

`choices`에는 최소 2개에서 최대 255개의 선택지를 지정할 수 있다. 각 선택지는 고유해야 한다.

선택지의 값은 자료형을 구분한다. 따라서 문자열 `"true"`와 불리언 값 `true`는 서로 다른 값으로 처리된다.

예를 들어 고객 문의를 영업, 기술지원, 결제 부서 중 하나로 분류하거나 AI Agent Router에서 적합한 에이전트를 선택할 때 사용한다.

### 3) Score — 단계별 점수 평가

입력 데이터를 제공된 순서 있는 평가 수준에 따라 평가한다.

| 속성             | 자료형            | 설명                                           |
| ---------------- | ----------------- | ---------------------------------------------- |
| `instructions` | string            | 평가 기준과 지침을 지정한다.                   |
| `levels`       | 객체 배열         | 점수 평가에 사용할 단계별 수준을 지정한다.     |
| `label`        | string            | 평가 단계의 이름을 지정한다.                   |
| `description`  | string, 선택 사항 | 해당 평가 단계에 대한 설명을 지정한다.         |
| `type`         | `"score"`       | 객체 유형을 나타내며 항상`score`를 사용한다. |
| `name`         | string, 선택 사항 | 질문을 식별하기 위한 이름을 지정한다.          |

예를 들어 고객 만족도를 매우 낮음, 낮음, 보통, 높음, 매우 높음으로 평가하거나 문서의 품질을 1\~5단계로 평가할 때 사용한다.

## 3-5. safety_identifier: string 또는 null (선택 사항)

API 호출자가 제공하는 최종 사용자 식별자를 지정한다.

이 값은 검증된 조직(Verified Organization)의 범위 내에서 사용되는 불투명 식별자(Opaque Identifier)다.

Responses API와 동일한 길이 제한이 적용되며, 인증된 사용자의 실제 신원을 나타내는 값은 아니다.

| 속성          | 값               |
| ------------- | ---------------- |
| 자료형        | string 또는 null |
| 필수 여부     | 선택 사항        |
| `minLength` | 0                |
| `maxLength` | 128              |

# 4. Predicate, Choice, Score 비교

| 구분      | Predicate             | Choice                        | Score                   |
| --------- | --------------------- | ----------------------------- | ----------------------- |
| 주요 목적 | 진술의 참일 확률 추정 | 최적의 선택지 선정            | 단계별 수준 평가        |
| 평가 방식 | 확률 기반 판단        | 후보 선택                     | 순서형 척도 평가        |
| 사용 사례 | 사기 의심 여부        | AI Agent 라우팅               | 위험도 평가             |
| 질문 예시 | 사기 거래인가?        | 어느 Agent가 처리해야 하는가? | 위험도가 어느 수준인가? |

핵심적으로 `Predicate`는 가능성을 판단하고, `Choice`는 주어진 후보 중 하나를 선택하며, `Score`는 정의된 단계에 따라 입력 데이터를 평가하는 역할을 수행한다.

# 4. Decisions API 동영상 - 어떤 것을 만들 수 있는가?

[지난 후기에서 Decisions API는](https://synabreu.github.io/openai/%ED%9B%84%EA%B8%B0-%EC%98%A4%ED%94%88AI-DevDay-%ED%82%A4%EB%85%B8%ED%8A%B8%EB%A5%BC-%EB%B3%B4%EA%B3%A0/#43-decisions-api) 최근에 각광 받은 [Jev와](https://typesafe.ai/) 유사한 부분과 차이점을 각각 말한 적이 있다.

# 5. 참고 자료

---
