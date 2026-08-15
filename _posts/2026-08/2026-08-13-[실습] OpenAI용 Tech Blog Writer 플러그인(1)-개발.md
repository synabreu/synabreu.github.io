---
title: "[실습] OpenAI용 Tech Blog Writer 플러그인(1)-개발"
date: 2026-08-13
tags: [오픈AI, OpenAI, ChatGPT, MCP, Agent, Plugin,  ]
typora-root-url: ../
toc: true
categories: [openai]
---
OpenAI Skills-only Plugin 프로젝트용으로 Skill 정의와 Plugin 메타데이터를 편집하고 Git으로 관리하며 ChatGPT/Codex에서 설치해 테스트하는 실습에 대해 초보자들도 쉽게 할 수 있도록 가이드를 작성해 보도록 하겠다.

* [Tech Blog Writer Plugin 오픈소스](https://github.com/synabreu/openai-blog-ap)

# 1. 전체 개발 구조 이해하기

```
openai-blog-ap/
│
├── .codex-plugin/
│   └── plugin.json
│
├── skills/
│   └── write-korean-tech-blog/
│       ├── SKILL.md
│       │
│       ├── agents/
│       │   └── openai.yaml
│       │
│       └── references/
│           ├── article-template.md
│           └── editorial-checklist.md
│
└── README.md
```

이 플러그인은 OpenAI Developer Blog를 검색하고, 공식 문서로 내용을 교차 검증한 뒤 한국어 기술 블로그로 재구성하는 스킬-온리 플러그인(Skill-only Plugin)이다. 별도의 MCP 서버, 웹 UI, OPENAI_API_KEY가 필요하지 않도록 설계했다. 따라서 개발 흐름은 다음과 같습니다.

# 2. MCP(Model Context Protocol)란?

MCP는 AI 모델과 외부 서비스를 연결하는 개방형 표준 프로토콜이다. 쉽게 말하면, AI가 다양한 프로그램, 데이터베이스, 클라우드 서비스와 통신할 수 있도록 만든 공통 연결 규격이다.

![plugins-08]({{ '/images/2026-08/plugins-08.jpg' | relative_url }})

MCP Server는 특정 시스템의 기능을 AI에게 제공한다. 예를 들어, AWS MCP는 AI 에이전트가 AWS 클라우드 환경과 연결되어 인프라 운영 및 모니터링 작업을 수행할 수 있도록 지원한다. AI는 AWS 리소스 정보를 조회하고, 애플리케이션 실행 상태를 확인하며, 운영 데이터를 기반으로 문제 분석과 비용 최적화를 수행할 수 있다.

# 3. MCP의 핵심 역할

MCP는 LLM에게 "도구 사용 능력"을 제공한다.

```
AI
 |
 | "AWS 서버 상태 확인 필요"
 |
AWS MCP 호출
 |
AWS API 실행
 |
결과 반환
 |
AI 답변 생성
```

# 4. 에이전트 플러그인이란?

Agent Plugin은 특정 목적을 가진 전문 AI Agent 패키지이다. 단순히 도구를 연결하는 것이 아니라 목표 설정, 계획 수립, 작업 실행, 결과 평가, 반복 개선까지 수행한다. 예를 들어, 소프트웨어 엔지니어 에이전트 플로그인의 목표는 `새로운 웹 서비스를 개발하라` 이다. 내부 동작은 다음과 같다.

```
Software Engineer Agent

1. 요구사항 분석
2. 아키텍처 설계
3. 코드 작성
4. GitHub MCP 연결
5. 테스트 실행
6. 보안 검사
7. Pull Request 생성
8. 리뷰 반영
```

여기서 GitHub MCP는 도구이고 Software Engineer Agent는 일을 수행하는 주체이다.

# 5. MCP와 Agent Plugin 차이

| 구분      | MCP          | Agent Plugin         |
| --------- | ------------ | -------------------- |
| 목적      | 연결         | 업무 수행            |
| 역할      | Tool 제공    | Task 수행            |
| 수준      | 인프라 계층  | 애플리케이션 계층    |
| 핵심 기능 | API 연결     | 판단, 계획, 실행     |
| 예시      | AWS MCP      | Cloud Engineer Agent |
| 예시      | GitHub MCP   | Developer Agent      |
| 예시      | Database MCP | Data Analyst Agent   |

# 6. AI Agent 구조에서 MCP와 Plugin 위치

상위 계층은 Agent가 담당하고 하위 연결 계층은 MCP가 담당한다.

![plugins-07]({{ '/images/2026-08/plugins-07.jpg' | relative_url }})
