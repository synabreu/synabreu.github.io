---
title: "[실습] Decisions API 사용기(5)-보험접수 분류"
date: 2026-10-12
tags: [openai, devday, samaltman, chatgpt, codex, linux, codexmobile, meta, muse, spacexai, grotbot, dots, ChatGPT Spaces]
typora-root-url: ../
toc: true
categories: [openai]
---
이 프로젝트는 지금까지 배운 OpenAI의 Decisions API 를 응용해 실제로 B2B에서 사용할만 한 응용 AI로 보험 접수 분류 프로젝트를 작성해 보았다. 보험 기록자가 사고 접수 문장을 받아 접수 대상 여부(predicate), 처리 경로(choice), 긴급도(score), 부상 관련 검토 필요성(predicate)을 평가한다.

![그림 1. decision insurance main]({{ '/images/2026-10/decisions-insurance-main.png' | relative_url }}){: style="width: 90%; height: auto;"}


# 1. 실제 OpenAI API 연결

실제 Decisions API를 사용하려면 `OPENAI_API_KEY`가 필요하다. `decisions-insurance` 프로젝트 루트의 `.env` 파일에 기존 API 키와 모델을 설정한다. `.env` 파일이 없다면 `.env.example`을 복사해 만든다.

다음과 같이 여러분의 API 키를 <Your_OpenAI_API_Key>에 대체해 입력한다.

``dotenv OPENAI_API_KEY=<Your_OpenAI_API_Key> OPENAI_DECISIONS_MODEL=gpt-6-luna ``

설정은 현재 환경변수 → `.env.local` → `.env` 순서로 적용한다. 앞에서 읽은 값은 뒤의 파일로 덮어쓰지 않는다. 키는 서버에서만 사용하며 브라우저에 전달하지 않는다.

실제 분석은 다음 명령으로 실행한다.

``powershell python decision_insurance.py --input examples/car_accident.txt --output output/car_accident_result.json ``

사고 접수 텍스트를 Decisions API에 전달해 접수 대상 여부, 처리 경로, 긴급도, 부상 관련 검토 필요성을 판단한다. 반환된 결과를 검증한 뒤 담당자의 검토 경로를 제안하고 JSON 파일로 저장한다. 실제 부서 배정이나 보험금 지급은 실행하지 않는다.

웹 화면은 `.\start-web.ps1`로 실행하고 http://127.0.0.1:8093 에 접속한다. 실제 분석 모드를 선택해야 API를 호출한다. 환경 설정을 변경했다면 서버를 재시작한다.

`--mock`은 고정 예제 응답만 사용하며 API를 호출하지 않는다. 실제 호출에는 API 사용료가 발생한다. 실제 키가 들어 있는 `.env`와 `.env.local`은 Git에 올리지 않는다.

# 2. Decisions-insurance 예제 실행

## 2-1. Windows 11 운영체제의 PowerShell 환경에서 설치 및 실행

1) 가상환경 생성

```powershell
python -m venv .venv
```

2) 가상환경 활성화

```powershell
.\.venv\Scripts\Activate.ps1
```

3) 필요한 패키지 설치

```powershell
python -m pip install -U -r requirements.txt
```

4) 설치가 끝나면 모의 실행한다.

```powershell
python decision_insurance.py --mock
```

실행 후 결과 화면은 [그림1]와 같다.

![그림 2. decision insurance mook]({{ '/images/2026-10/decisions-insurance-01.png' | relative_url }}){: style="width: 70%; height: auto;"}

5) 콘솔에서 실제 실행은 다음과 같다.

```powershell
python decision_insurance.py --input examples/claim.txt --output output/car_accident_result.json
```

실행 후 결과 화면은 [그림2]와 같다.

![그림 3. decision insurance json 결과]({{ '/images/2026-10/decisions-insurance-02.png' | relative_url }}){: style="width: 70%; height: auto;"}

또한 car_accident_result.json 파일은 다음과 같다.

```json
{
  "mode": "live",
  "decision": {
    "answers": [
      {
        "name": "valid_insurance_claim",
        "probability": 1.0,
        "type": "predicate"
      },
      {
        "choice": "MEDICAL_REVIEW",
        "confidence": 0.95,
        "name": "claim_route",
        "probabilities": [
          {
            "probability": 0.0,
            "value": "AUTO_APPROVE"
          },
          {
            "probability": 0.02,
            "value": "NORMAL_ADJUSTER"
          },
          {
            "probability": 0.96,
            "value": "MEDICAL_REVIEW"
          },
          {
            "probability": 0.01,
            "value": "FRAUD_REVIEW"
          },
          {
            "probability": 0.01,
            "value": "EMERGENCY"
          }
        ],
        "type": "choice"
      },
      {
        "confidence": 0.92,
        "name": "urgency",
        "probabilities": [
          {
            "label": "LOW",
            "probability": 0.0,
            "value": 0
          },
          {
            "label": "MEDIUM",
            "probability": 0.05,
            "value": 1
          },
          {
            "label": "HIGH",
            "probability": 0.92,
            "value": 2
          },
          {
            "label": "CRITICAL",
            "probability": 0.03,
            "value": 3
          }
        ],
        "score": 1.98,
        "type": "score"
      },
      {
        "name": "medical_attention_required",
        "probability": 1.0,
        "type": "predicate"
      }
    ],
    "model": "gpt-6-luna",
    "usage": {
      "input_tokens": 1063,
      "input_tokens_details": {
        "cache_write_tokens": 0,
        "cached_tokens": 0
      },
      "output_tokens": 0,
      "output_tokens_details": {
        "reasoning_tokens": 0
      },
      "total_tokens": 1063
    }
  },
  "workflow": {
    "route": "MEDICAL_REVIEW",
    "action": "부상 또는 의료비 관련 담당자 검토",
    "reason": "검토 경로 제안",
    "executed": false
  }
}
```

## 2-2. macOS / Linux 운영체제에서 설치

```bash
cd decisions-insurance
python3 -m venv .venv
.venv/bin/python -m pip install -U -r requirements.txt
.venv/bin/python decision_insurance.py --input examples/claim.txt --output output/car_accident_result.json
```

VS Code에서는 이 폴더를 열고 Python 인터프리터를 `.venv`로 선택한다. 명령은 VS Code 터미널에서도 그대로 사용할 수 있다.

## 2-3. API 키 없이 실행하려면?

모의 모드는 별도 패키지 설치가 필요 없다.

```bash
python decision_insurance.py --mock
```

고정된 가상 응답을 읽고 `MEDICAL_REVIEW`를 표시한다. 확률은 설명용으로 작성한 값이며 실제 모델 결과가 아니다. 임의 텍스트를 분석하지 않으므로 `--mock`과 `--input`을 함께 사용할 수 없다.

# 3. 명령행 인자에 따른 실행 방식

프로젝트 폴더에서 다음 형식으로 실행한다.

```powershell
python decision_insurance.py [argument]
```

## 3-1. CLI 인자와 기본 동작

| 인자                  | 기본값                           | 실행에 미치는 영향                                                                                |
| --------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------- |
| `--mock`            | 꺼짐                             | API 대신`examples/mock_response.json`의 고정 응답을 사용한다. API 키와 외부 패키지가 필요 없다. |
| `--input 파일경로`  | 프로젝트의`examples/claim.txt` | 실제 API에 전달할 UTF-8 사고 접수 텍스트 파일을 지정한다.                                         |
| `--model 모델명`    | 환경변수 또는`gpt-6-luna`      | 실제 API 호출에 사용할 모델을 지정한다.                                                           |
| `--threshold 숫자`  | `0.80`                         | API 응답을 받은 뒤 검토 경로를 결정할 임계값을 지정한다. 범위는`0~1`이다.                       |
| `--output 파일경로` | 저장 안 함                       | 콘솔 출력과 함께 결과를 JSON 파일로 저장한다. 상위 폴더는 자동 생성한다.                          |
| `--help`            | 해당 없음                        | 사용 가능한 인자를 출력하고 종료한다.                                                             |

인자가 없으면 기본 사고 파일로 실제 API를 호출한다. 실제 모드의 모델 선택 우선순위는 `--model` → `OPENAI_DECISIONS_MODEL` → `gpt-6-luna`다. 환경 설정은 기존 환경변수 → `.env.local` → `.env` 순서로 읽으며 먼저 설정된 값을 유지한다.

`--mock`과 `--input`을 함께 지정하면 오류로 종료한다. 모의 모드에서는 환경 파일을 읽지 않으며 `--model`은 실제 호출에 사용되지 않는다. `--threshold`는 고정 응답의 경로 계산에도 적용된다.

## 3-2. 실행 예시

다음 명령은 모두 프로젝트 폴더에서 실행한다.

```powershell
# 인자가 없으면 기본 사고 파일로 실제 API 호출
python decision_insurance.py

# API 호출 없이 고정 예제로 실행
python decision_insurance.py --mock

# 지정한 사고 파일로 실제 분석
python decision_insurance.py --input examples/car_accident.txt

# 모델과 임계값을 지정하고 결과 저장
python decision_insurance.py --input examples/claim.txt --model gpt-6-luna --threshold 0.90 --output output/result.json

# 모의 결과를 파일로 저장
python decision_insurance.py --mock --output output/mock_result.json

# 인자 도움말 확인
python decision_insurance.py --help
```

## 3-3. 임계값에 따른 검토 경로 분기

`--threshold`는 모델의 판단을 바꾸는 인자가 아니라 응답 이후의 분기 기준이다. 응답 형식과 확률 검증을 통과하면 다음 순서로 처리한다.

1. 선택 경로가 `EMERGENCY`이면 긴급 검토를 우선한다.
2. 경로 선택 신뢰도가 임계값보다 낮으면 `HUMAN_REVIEW`로 보낸다.
3. `AUTO_APPROVE`도 이 데모에서는 `HUMAN_REVIEW`로 보낸다.
4. `NORMAL_ADJUSTER`이면서 의료 검토 확률이 임계값 이상이면 `MEDICAL_REVIEW`로 바꾼다.
5. 그 외에는 선택된 경로를 유지한다.

임계값을 높이면 낮은 신뢰도로 담당자 검토에 넘어가기 쉬워지지만, 의료 검토 전환 조건은 더 엄격해진다. 응답 거부, 누락, 중복 또는 잘못된 형식이 발견되어도 담당자 검토로 전환한다.

결과 JSON은 `mode`, `decision`, `workflow`를 포함한다. 실제 부서 배정이나 보험금 지급은 실행하지 않으므로 `workflow.executed`는 항상 `false`다. `--output`이 없으면 콘솔에만 출력한다.

# 4. 웹 UI 실행

`decisions-insurance` 프로젝트 폴더에서 다음 명령 중 하나를 실행한다. 일반 Python 환경에서는 기존 설치 절차를 먼저 완료한다.

```powershell
# PowerShell 실행 스크립트 사용
.\start-web.ps1

# 일반 Python 환경
python server.py
```

기본 포트는 `8093`이며, 실행 후 [http://127.0.0.1:8093](http://127.0.0.1:8093)을 연다. 포트를 변경하려면 다음과 같이 실행한다.

```powershell
.\start-web.ps1 -Port 8094
# 또는
python server.py --port 8094
```

포트를 `8094`로 지정한 경우 [http://127.0.0.1:8094](http://127.0.0.1:8094)에서 접속한다. 종료하려면 실행 터미널에서 Ctrl+C를 누른다.

웹 화면에서는 사고 내용 입력, 가상 예시 불러오기, 실제·모의 분석 모드 선택, 임계값 설정, 질문별 확률과 검토 경로 확인, 전체 응답 보기 및 JSON 다운로드를 지원한다.

실제 모드는 기존 `.env.local` 또는 `.env`의 API 키를 서버에서만 읽는다. 모의 모드는 입력 내용을 분석하지 않고 고정된 예시 응답을 표시한다. 사고 내용과 결과는 서버 파일로 자동 저장하지 않으며, 필요한 결과는 브라우저에서 JSON으로 다운로드할 수 있다.

![그림 4. decision insurance Web UI]({{ '/images/2026-10/decisions-insurance-03.png' | relative_url }}){: style="width: 70%; height: auto;"}

# 5. 참고 자료

* [OpenAI Developers - Decisions API](https://developers.openai.com/api/docs/guides/decisions)
* [decisions-insurance 예제 소스](https://github.com/synabreu/openai-decisions-workshop/tree/main/decisions-insurance)
