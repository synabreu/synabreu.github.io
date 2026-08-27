---
title: "[실습] OpenAI 관리형 RAG API 워크샵(2)-RAG"
date: 2026-08-26
tags: [오픈AI, OpenAI, GPT-5.6, agenticai, aiagent, openrouter, docker, powershell, fastapi, swagger-ui, visual studio code, Linux, Windows11, Kubernetes, On-Premises ]
typora-root-url: ../
toc: true
categories: [openai]
---

이번 글에서는 RAG의 간단한 개념과 관리형 RAG와 기업 사내 보안, 그리고 OpenAI의 RAG API, Vector Store, File Search 등에 대해 살펴본다. RAG는 외부 문서를 검색해 최신 정보와 사내 지식을 근거로 답변을 생성하는 기술이다.

OpenAI의 Vector Store와 File Search를 활용하면 복잡한 인프라 없이 관리형 RAG를 빠르게 구축할 수 있습니다. 다만 완전한 에어갭이나 데이터 외부 반출 금지 환경에서는 사내 자체 RAG 시스템이 필요하다. 

# 1. 검생 증강 생성(RAG)란?

RAG(검색 증강 생성, Retrieval-Augmented Generation)는 모델이 답변을 생성하기 전에 외부 지식 저장소에서 질문과 관련된 문서를 검색하고, 그 결과를 근거로 답변을 생성하는 구조를 뜻한다. 

```text
사용자 질문
    |
    v
검색(Retrieval)
    |
    v
관련 문서 조각
    |
    v
   LLM
    |
    v
근거 기반 답변
```

이를 통해 모델의 일반 지식만 사용하는 것보다 최신 정보나 사내 문서를 바탕으로 더 근거 있는 답변을 만들 수 있다.

# 2. 워크삽 프로젝트에서 사용한 "OpenAI RAG API"

이 프로젝트는 별도의 Pinecone, Qdrant, FAISS와 같은 벡터 데이베이스를 사용하지 않는다. OpenAI API가 제공하는 다음 기능을 직접 사용한다.

1) **Vector Stores**
   - 문서를 검색 가능한 지식 저장소로 관리한다.

2) **Retrieval API**
   - `client.vector_stores.search(...)`
   - Vector Store에 직접 semantic search를 수행한다.
   - LLM을 호출하지 않고 어떤 문서 조각이 검색되는지 볼 수 있다.

3) **Responses API + File Search**
   - `tools=[{"type": "file_search", ...}]`
   - 모델이 OpenAI-hosted File Search를 사용해 관련 문서를 검색하고 답변한다.

# 3. Retrieval과 File Search의 차이

```text
[A] Retrieval API 직접 사용

Question
   |
   v
vector_stores.search()
   |
   v
Top-K chunks
   |
   +--> 개발자가 결과를 직접 검사/가공
```

```text
[B] Responses API + File Search

Question
   |
   v
Responses API
   |
   v
Model decides to use file_search
   |
   v
OpenAI Vector Store
   |
   v
Retrieved context
   |
   v
Grounded answer
```

초보자는 먼저 [A]를 실행하여 RAG의 Retrieval 단계가 실제로 무엇을 반환하는지 확인한 뒤 [B]로 이동하는 것이 좋다.

# 4. 관리형(Managed) RAG의 의미

전통적인 직접 구축형 RAG는 보통 아래 요소를 개발자가 직접 선택한다.

```text
Parser -> Chunker -> Embedding -> Vector DB -> Retriever -> Reranker -> LLM
```

OpenAI 관리형 RAG에서는 Vector Store와 File Search가 상당 부분을 관리한다.

```text
Files -> OpenAI Vector Store -> Retrieval/File Search -> OpenAI Model
```

따라서 PoC와 첫 번째 엔터프라이즈AI 프로토타입을 빠르게 만들기에 적합하다.

| 단계 | 관리형 RAG 처리 | 사용자 제어 |
|---|---|---|
| Parsing | PDF, DOCX, Markdown 등을 자동으로 읽어 텍스트화한다. | 파서나 OCR 방식 선택은 지원되지 않는다. |
| Chunking | 자동으로 문서를 분할한다. | 청크 크기와 중첩 크기를 설정할 수 있다. |
| Embedding | 청크를 자동 임베딩한다. | 임베딩 모델이나 차원 선택은 제공되지 않는다. |
| Indexing | Vector Store에 자동 인덱싱한다. | 메타데이터, 검색 필터, 랭킹 옵션 등을 조정할 수 있다. |

파일을 벡터 스토어(Vector Store)에 추가하면 자동으로 청킹(chunking), 임베딩(embedding), 인덱싱(indexing)을 수행한다. 

또한 기본 청킹 설정은 청크 크기가 800토큰, 청크 중첩가 400토큰, 청크 크기 허용 범위가 100 ~ 4096토큰이며, 중첩 크기가 청크 크기의 절반 이하 권장한다. 

현재 워크삽 프로젝트의 upload_and_poll() 호출에 chunking_strategy를 지정하면 청킹 설정을 조정할 수 있다.

```python
client.vector_stores.files.upload_and_poll(
    vector_store_id=vector_store.id,
    file=f,
    chunking_strategy={
        "type": "static",
        "max_chunk_size_tokens": 1200,
        "chunk_overlap_tokens": 200,
    },
)
```

다만 현재 공식 문서의 Vector Store 입력 스키마에는 파서 종류나 임베딩 모델을 선택하는 옵션이 없다. 따라서 다음처럼 구분하는 것이 좋다.

- 간편한 워크숍·일반 문서 검색: OpenAI 관리형 파싱 및 임베딩 사용
- 표, OCR, 복잡한 PDF 레이아웃: 직접 전처리한 마크다운  또는 텍스트를 업로드
- 임베딩 모델 및 벡터 차원까지 직접 제어: 임베딩(Embeddings) API와 별도 벡터 DB로 자체 RAG 구성

결론적으로, 관리형 RAG에서는 청킹(Chunking)은 부분적으로 조정할 수 있지만 파싱(Parsing)과 임베딩(Embedding)은 OpenAI가 관리한다.

# 5. 인터넷과 사설 연결이 모두 차단된 기업 환경 

인터넷과 사설 연결이 모두 차단된 완전한 에어갭 환경에서는 OpenAI 관리형 RAG를 사용할 수 없다. 관리형 RAG의 Vector Store, File Search, Responses API가 OpenAI 클라우드에서 실행되기 때문이다. 다만 “공용 인터넷은 차단하지만 승인된 사설망 연결은 허용하는 환경”이라면 사용할 수 있다.

| Private 환경 유형 | 관리형 RAG | 구성 |
|---|---:|---|
| 완전한 에어갭 | 불가능 | 온프레미스 자체 RAG 필요 |
| Azure 사설망 연결 허용 | 가능 | OpenAI Private Link 사용 |
| 제한적 HTTPS 아웃바운드 허용 | 가능 | 방화벽·프록시를 통해 OpenAI API만 허용 |
| 데이터 외부 반출 자체가 금지 | 불가능 | Private Link를 사용해도 데이터는 OpenAI에서 처리됨 |

## 5-1. OpenAI Private Link 사용

OpenAI Private Link를 사용하면 Azure 워크로드가 공용 API 주소를 거치지 않고 Azure Private Endpoint를 통해 OpenAI 지역 API에 접근한다.

```
사내 Private Network
        │
        ▼
Azure VNet
        │
        ▼
Azure Private Endpoint
        │
        ▼
OpenAI Regional Private Link
        │
        ├── Files / Uploads
        ├── Vector Stores
        └── Responses API + File Search
```

현재 공식 호환성 표에는 다음 API가 Private Link 지원 대상으로 명시되어 있다.

- /v1/files, /v1/uploads
- /v1/vector_stores
- /v1/responses
- /v1/embeddings

따라서 이 워크숍의 관리형 RAG 구성도 Private Link를 통해 사용할 수 있다. 다만 현재 Private Link는 셀프서비스가 아니므로 OpenAI 담당자 또는 영업팀을 통해 접근 권한과 지역별 서비스 정보를 받아야 한다.

Python 언어에서는 클라이언트의 base_url을 Private Link 주소로 변경한다

```python
from openai import OpenAI

client = OpenAI(
    api_key=require_api_key(),
    base_url=(
         "https://southcentralus.privatelink.api.openai.com/v1"
    ),
)
```

실제 주소는 OpenAI가 온보딩 과정에서 제공하는 지역 주소를 사용해야 한다. Private Link는 통신 경로를 사설화하지만 OpenAI를 온프레미스에 설치하는 기능은 아니다.

또한 Vector Store와 업로드 파일은 상태를 유지해야 하므로 Zero Data Retention 대상이 아니다. 공식 OpenAI Docs 기준으로 Vector Store와 파일은 사용자가 삭제할 때까지 애플리케이션 상태로 저장된다.

## 5-2. 자체 RAG 시스템을 구축해야 하는 상황

원문, 청크, 임베딩 및 질문 데이터가 사내 경계를 벗어나면 안 된다고 하는 경우에는 다음과 같은 아키텍처 구성으로 자체 RAG를 구축해야 한다.

```
사내 문서
   ↓
로컬 파싱 / OCR
   ↓
로컬 청킹
   ↓
로컬 임베딩 모델
   ↓
사내 벡터 DB
   ↓
온프레미스 LLM
```

결론을 말해서 공용 인터넷만 금지된 환경은 OpenAI Private Link로 관리형 RAG를 사용할 수 있지만, 완전한 에어갭이나 외부 데이터 처리 금지 환경에서는 자체 RAG가 필요하다.