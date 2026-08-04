---
title: "LLM으로 비정형 문서를 정형 데이터로 변환하는 Document AI 플랫폼"
date: 2026-07-02
tags: [오픈소스, opensource, unstract, rag, agenticai, AI에이전트]
typora-root-url: ../
toc: true
categories: [opensource]
---
기업 데이터의 상당 부분은 여전히 문서 형태로 존재한다. 계약서, 청구서, 금융 명세서, 의료 기록, PDF 보고서처럼 사람이 읽을 수 있지만 컴퓨터가 바로 활용하기 어려운 비정형 데이터가 대표적이다.

과거에는 이러한 데이터를 처리하기 위해 OCR, 정규식, 템플릿 기반 추출 시스템을 구축했다. 하지만 문서 형식이 조금만 변경되어도 다시 개발해야 하는 문제가 있었다.최근 생성형 AI와 LLM 기술의 발전으로 문서 처리 방식도 변화하고 있다. 이제는 사람이 원하는 데이터 구조를 자연어로 정의하고, LLM이 다양한 문서 형태에서 필요한 정보를 추출하는 시대가 열리고 있다.

Unstract는 이러한 LLM 기반 문서 자동화를 위한 오픈소스 플랫폼이다. Unstract는 PDF, 이미지, 스캔 문서 등 다양한 비정형 데이터를 LLM을 활용해 구조화된 JSON 데이터로 변환한다. 개발자는 복잡한 파싱 로직이나 문서별 템플릿을 직접 작성하지 않고, 추출하고 싶은 데이터를 자연어 프롬프트로 정의할 수 있다.


# 1. Unstract의 주요 특징

### 1) Prompt 기반 Document Extraction

기존 문서 추출 방식은 문서마다 별도의 규칙과 코드를 작성해야 했다.

예를 들어 금융 계약서에서 다음 정보를 추출한다고 가정해 보자.

```json
{
  "customer_name": "",
  "contract_date": "",
  "amount": "",
  "contract_period": ""
}
```

기존 방식에서는 문서 레이아웃 분석, OCR 처리, Regex 작성이 필요했다. 하지만 Unstract에서는 원하는 결과를 Prompt 형태로 정의한다.

```
계약서에서 다음 정보를 추출해줘.

- 고객명
- 계약일
- 계약 금액
- 계약 기간

결과는 JSON 형식으로 반환해줘.
```

LLM은 문서의 다양한 표현 방식을 이해하고 구조화된 결과를 생성한다.

## 2) API 기반 Document Processing

Unstract는 단순한 분석 도구가 아니라 실제 서비스 환경에서 사용할 수 있도록 API 형태로 배포할 수 있다. 예를 들어, 기업 내부 시스템에서 PDF 계약서를 업로드하면:

```
PDF 문서
   |
   v
Unstract API
   |
   v
LLM Extraction
   |
   v
Structured JSON
   |
   v
Database / ERP / CRM
```

형태의 자동화 파이프라인을 구축할 수 있다. Unstract는 API 배포뿐 아니라 ETL Pipeline 형태로 데이터를 데이터베이스나 데이터 웨어하우스로 전달하는 기능도 제공한다.

## 3) RAG와 AI Agent 시대의 핵심 컴포넌트

최근 기업 AI 프로젝트에서는 RAG(Retrieval-Augmented Generation)와 AI Agent가 빠르게 확산되고 있다. 하지만 좋은 RAG 시스템을 만들기 위해서는 먼저 문서 데이터를 LLM이 이해할 수 있는 형태로 변환해야 한다. Unstract는 이러한 데이터 준비(Data Preparation) 영역에서 중요한 역할을 한다.

```
기업 문서
(PDF, 계약서, 이미지)

        ↓

Unstract
(Document Extraction)

        ↓

Structured Data

        ↓

Vector DB / Database

        ↓

RAG Application

        ↓

AI Agent
```

즉 Unstract는 AI Agent 시대의 Document Intelligence Layer라고 볼 수 있다.

---

# 2. Unstract 실행하기

## 1) 오픈소스 저장소 복제

```bash
git clone https://github.com/synabreu/unstract.git

cd unstract
```

## 2) 도커 기반 실행

Unstract는 Docker 환경에서 쉽게 실행할 수 있다.

```bash
./run-platform.sh
```

실행 후 브라우저에서:

```
http://frontend.unstract.localhost
```

접속하면 Unstract 플랫폼을 사용할 수 있다. 오픈소스 버전은 Linux 또는 macOS 환경에서 Docker와 Docker Compose를 이용해 실행할 수 있다.

# 3. Python 예제: Unstract API 호출하기

Unstract를 API 서비스로 배포했다고 가정하면 Python 애플리케이션에서 문서를 전달하고 구조화 데이터를 받을 수 있다.

```python
import requests


# Unstract API 엔드포인트
url = "http://localhost:8000/api/v1/extract"


# 분석할 PDF 파일
files = {
    "file": open(
        "contract.pdf",
        "rb"
    )
}


# 추출 요청
response = requests.post(
    url,
    files=files
)


# 결과 출력
result = response.json()

print(result)
```

예상 결과:

```json
{
  "customer_name": "ABC Corporation",
  "contract_date": "2026-08-01",
  "amount": "50000000",
  "contract_period": "12 months"
}
```

이처럼 개발자는 복잡한 OCR Pipeline이나 문서별 Parser를 직접 개발하지 않고 LLM 기반 Document Extraction 기능을 애플리케이션에 통합할 수 있다.

---

# 4. 개발자가 Unstract를 주목해야 하는 이유

생성형 AI 시대의 중요한 변화는 "텍스트 생성"에서 "업무 자동화"로 이동하고 있다는 점이다. 기업 업무의 대부분은 여전히 문서 중심으로 이루어진다. 따라서 앞으로 AI Agent가 실제 업무를 수행하려면 계약서, 보고서, 이메일, 이미지 같은 비정형 데이터를 이해하는 능력이 필수적이다.

Unstract는 LLM을 활용해 이러한 현실 세계의 데이터를 구조화하고, API와 데이터 파이프라인으로 연결하는 오픈소스 플랫폼이다. LLM 애플리케이션 개발자는 이제 모델 호출 방법뿐 아니라 **문서 데이터 처리, Extraction Pipeline, RAG 데이터 준비 과정**까지 이해해야 한다. Unstract는 이러한 AI Engineering 역량을 키우기 위한 좋은 실습 플랫폼이다.

* 깃허브 : [Unstract](https://github.com/synabreu/unstract)

