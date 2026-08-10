---
title: "OpenAI의 Agent Plugins과 Skills 차이점"
date: 2026-08-09
tags: [openai, gpt5, 오픈AI, AgentPlugins, Apple, AppleIntelligence, ios, ChatGPT, Google]
typora-root-url: ../
toc: true
categories: [openai]
---
OpenAI 생태계에서 **Skill(스킬)** 과 **Plugin(플러그인)** 은 모두 AI의 기능 확장을 위한 개념이지만, 목적과 동작 위치가 어떻게 다른지 궁금해서 정리해보겠다.

## 1. 개념 비교

**Plugin = 외부 서비스를 연결하는 "확장 인터페이스",  Skill = AI Agent가 특정 업무를 수행하는 "능력 모듈"**

| 구분        | OpenAI Skill                            | OpenAI Plugin               |
| ----------- | --------------------------------------- | --------------------------- |
| 목적        | Agent에게 전문 능력 부여                | 외부 서비스 연결            |
| 중심 개념   | **행동(Action)과 업무 수행 능력** | **API Integration**   |
| 실행 주체   | Agent 내부                              | 외부 시스템                 |
| 주요 사용자 | Agent 개발자                            | 서비스/API 개발자           |
| 예          | PDF 분석 Skill, 코드 리뷰 Skill         | GitHub Plugin, Slack Plugin |
| 구조        | Prompt + Tools + Logic                  | API Schema + Authentication |
| 방향        | AI → 작업 수행                         | AI ↔ 외부 서비스           |

## 2. Agent Plugin이란?

Plugin은 ChatGPT가 외부 시스템과 통신하도록 만드는 연결 계층입니다.

![plugins-02]({{ '/images/2026-08/plugins-02.jpg' | relative_url }})

사용자가 "내 GitHub Issue 확인해줘" 라고 하면:

<pre class="overflow-visible! px-0!" data-start="833" data-end="922"><div class="relative w-full mt-4 mb-1"><div class=""><div class="contents"><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-(--code-block-surface) corner-superellipse/1.1 overflow-clip rounded-3xl [--code-block-surface:var(--bg-elevated-secondary)] dark:[--code-block-surface:var(--composer-surface-primary)] lxnfua_clipPathFallback"><div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1"></div><div class="relative"><div class="pe-11 pt-3"><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr"><div class="cm-scroller"><pre class="cm-content q9tKkq_readonly m-0"><code><span>ChatGPT
   ↓
GitHub Plugin 호출
   ↓
GitHub API 요청
   ↓
Issue 반환
   ↓
ChatGPT 답변 생성</span></code></pre></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></div></pre>

## 3. Agent Plugin 특징

### 1) 외부 데이터 접근

예: GitHub, Jira,  AWS,  Google Drive,  CRM, ERP

### 2) API 중심

Plugin은 보통:

<pre class="overflow-visible! px-0!" data-start="1053" data-end="1113"><div class="relative w-full mt-4 mb-1"><div class=""><div class="contents"><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-(--code-block-surface) corner-superellipse/1.1 overflow-clip rounded-3xl [--code-block-surface:var(--bg-elevated-secondary)] dark:[--code-block-surface:var(--composer-surface-primary)] lxnfua_clipPathFallback"><div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1"></div><div class="relative"><div class="pe-11 pt-3"><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr"><div class="cm-scroller"><pre class="cm-content q9tKkq_readonly m-0"><code><span>plugin.json
openapi.yaml
API Endpoint
Authentication</span></code></pre></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></div></pre>

구조이다.

예:

<pre class="overflow-visible! px-0!" data-start="1127" data-end="1181"><div class="relative w-full mt-4 mb-1"><div class=""><div class="contents"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="relative h-full w-full border-radius-3xl bg-(--code-block-surface) corner-superellipse/1.1 overflow-clip rounded-3xl [--code-block-surface:var(--bg-elevated-secondary)] dark:[--code-block-surface:var(--composer-surface-primary)] lxnfua_clipPathFallback"><div class="pointer-events-none absolute inset-x-4 top-12 bottom-4"><div class="pointer-events-none sticky z-40 shrink-0 z-1!"><div class="sticky bg-token-border-light"></div></div></div><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class=""><div class="relative"><div class=""><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr"><div class="cm-scroller"><pre class="cm-content q9tKkq_readonly m-0"><code><span>GET /issues

GET /users/{id}

POST /deploy</span></code></pre></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></div></div></div></pre>

## 4. Skill이란?

Skill은 Agent가 특정 업무를 수행하기 위한 "업무 능력 패키지"이다.

예:

### Coding Agent Skill

<pre class="overflow-visible! px-0!" data-start="1276" data-end="1362"><div class="relative w-full mt-4 mb-1"><div class=""><div class="contents"><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-(--code-block-surface) corner-superellipse/1.1 overflow-clip rounded-3xl [--code-block-surface:var(--bg-elevated-secondary)] dark:[--code-block-surface:var(--composer-surface-primary)] lxnfua_clipPathFallback"><div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1"></div><div class="relative"><div class="pe-11 pt-3"><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr"><div class="cm-scroller"><pre class="cm-content q9tKkq_readonly m-0"><code><span>Coding Skill

├── 코드 분석
├── 테스트 실행
├── Git 작업
├── PR 작성
├── Bug Fix
└── Review</span></code></pre></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></div></pre>

### Research Agent Skill

<pre class="overflow-visible! px-0!" data-start="1394" data-end="1462"><div class="relative w-full mt-4 mb-1"><div class=""><div class="contents"><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-(--code-block-surface) corner-superellipse/1.1 overflow-clip rounded-3xl [--code-block-surface:var(--bg-elevated-secondary)] dark:[--code-block-surface:var(--composer-surface-primary)] lxnfua_clipPathFallback"><div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1"></div><div class="relative"><div class="pe-11 pt-3"><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼd ͼr"><div class="cm-scroller"><pre class="cm-content q9tKkq_readonly m-0"><code><span>Research Skill

├── 검색
├── 논문 분석
├── 요약
├── 비교 분석
└── 보고서 생성</span></code></pre></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></div></pre>

즉 스킬은  "무엇을 할 수 있는가?"에 집중한다.

## 5. 구조 차이

Plugin 구조: 외부 세계와 연결

![plugins-03]({{ '/images/2026-08/plugins-03.jpg' | relative_url }})

Skill 구조: Agent의 지능과 행동 확장

 ![plugins-04]({{ '/images/2026-08/plugins-04.jpg' | relative_url }})

## 6. Skill 과 MCP와의 관계

관계:

 ![plugins-05]({{ '/images/2026-08/plugins-05.jpg' | relative_url }})

예:

MCP

```
AWS MCP Server

Tools:

- list_instances()
- get_cost()
- create_snapshot()
```

Skill

```
AWS Cloud Architect Skill

사용 가능한 MCP:

AWS MCP
Terraform MCP
Kubernetes MCP

업무 수행:

- 비용 최적화
- 아키텍처 설계
- 장애 분석
```

## 7. Agent 시대의 새로운 구조

앞으로의 구조는 다음과 같다. 

![plugins-06]({{ '/images/2026-08/plugins-06.jpg' | relative_url }})


## 8. 개발자 관점에서 선택 기준

| 만들고 싶은 것               | 선택                |
| ---------------------- | ----------------- |
| ChatGPT에서 내 서비스 API 사용 | Plugin/MCP        |
| Agent에게 특정 역할 부여       | Skill             |
| 반복 업무 자동화              | Skill             |
| SaaS 연결                | MCP/Plugin        |
| DevOps Agent           | Skill + MCP       |
| Coding Agent           | Skill + Tools     |
| 기업용 AI Agent           | Skill + MCP + RAG |


## 9. OpenAI Agent 개발 관점

예를 들어, "AI DevOps Engineer Agent" 를 만든다면:

### Skill
```
DevOps Engineer Skill

- Kubernetes 운영
- 장애 분석
- 로그 분석
- 배포 전략
```

### MCP
```
  Kubernetes MCP
  AWS MCP
  GitHub MCP
  Datadog MCP
```

### Agent
```  
AI DevOps Engineer

Prompt:
"You are senior DevOps engineer..."
```

## 10. 핵심 정리

* Plugin = AI가 외부 서비스를 사용하기 위한 연결 통로
* Skill = AI Agent가 특정 전문가처럼 행동하게 만드는 능력
* MCP = Skill이 사용할 수 있는 표준 Tool 연결 방식

현재 Agentic AI 시대의 가장 일반적인 패턴은:

```
Agent
 ├── Skill (전문 역할)
 |
 ├── MCP Tools (외부 실행)
 |
 ├── Memory (Context)
 |
 └── Evaluation (품질 검증)
```

이다.특히, Codex + OpenAI Agent + MCP 환경에서는 과거의 Plugin보다 Skill + MCP 조합이 차세대 Agent 확장 방식으로 보는 것이 맞다.

