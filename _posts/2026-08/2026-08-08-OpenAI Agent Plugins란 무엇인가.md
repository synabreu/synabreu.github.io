---
title: "OpenAI Agent Plugins란 무엇인가?"
date: 2026-08-08
tags: [openai, gpt5, 오픈AI, AgentPlugins, Apple, AppleIntelligence, ios, ChatGPT, Google]
typora-root-url: ../
toc: true
categories: [openai]
---
오늘은 오픈AI 플러그인에 대해 좀 더 기술적으로 파헤쳐보자! `AGENTS.md`는 규칙, Skill은 업무 절차, MCP는 외부 도구 연결, Agent Plugin은 이 둘을 배포 가능한 하나의 패키지로 묶는 규격이라는 것을 잘 알고 있을 것이다.Agent Plugins 1.0.0 규격에서는 핵심 구성요소로 Agent Skills와 MCP servers 두 종류를 정의하고 있다.

## 1. 전체 구조

![plugins-01]({{ '/images/2026-08/plugins-01.jpg' | relative_url }})

여기서 Agent Plugin은 새로운 종류의 에이전트가 아니다. 오히려 기존에 따로 관리하던 Skill과 MCP 서버 같은 에이전트 확장 요소를 하나의 이동 가능한 package로 만들기 위한 컨테이너 규격에 가깝다. 현재 Agent Plugins 공식 규격은 이를 특정 벤더에 종속되지 않는 호환 가능한 포터블 포맷(portable format)으로 정의하고 있다.

## 2. AGENTS.md는 무엇인가?

`AGENTS.md`는 Codex가 특정 프로젝트에서 어떤 방식으로 행동해야 하는지 알려주는 프로젝트 운영 규칙이다. 예를 들어 다음과 같이 작성할 수 있다.

```markdown
# Project: AI Research Platform

## Build

Install:

npm install

Run:

npm run dev

Test:

npm test

## Coding conventions

- TypeScript 사용
- async/await 사용
- any 사용 금지
- 모든 API 변경에는 테스트 추가

## Architecture

- frontend/: React
- backend/: FastAPI
- agents/: agent workflows
- skills/: reusable skills

## Do not modify

- legacy/
- generated/
- *.generated.ts
```

Codex 입장에서는 이것이 일종의 회사 또는 프로젝트의 규율과 같은 역할을 한다.
예를 들어 다음 질문에 답한다.

```text
우리 프로젝트에서는

어떤 방식으로 코딩해야 하는가?
어떤 명령어를 사용해야 하는가?
어떤 파일을 수정하면 안 되는가?
테스트는 어떻게 하는가?
어떤 아키텍처 원칙을 따라야 하는가?
```

여기서 중요한 점은 `AGENTS.md` 자체가 도구도 아니고 프로그램도 아니라는 것이다. 외부 시스템을 호출하지도 않고 새로운 기능을 설치하지도 않는다. 그저 Codex에게 지속적인 행동 지침과 컨텍스트를 제공한다.

## 3. Skill은 무엇인가?

Skill은 한 단계 더 나간다. Skill은 Codex에게 다음과 같이 알려주는 재사용 가능한 업무 매뉴얼과 같다.

> 이 업무를 할 때는 이렇게 처리해.

OpenAI는 Codex use case에서도 반복되는 workflow를 Skill로 저장해 다시 사용할 수 있는 방식을 소개하고 있다.

예를 들어, 회사에서 매주 AI 뉴스를 분석한다고 하자.

```text
skills/
└── ai-news-report/
    ├── SKILL.md
    ├── scripts/
    └── references/
```

`SKILL.md`:

```markdown
name: ai-news-report
description: AI 산업 뉴스를 조사하고 분석 보고서를 작성한다.

# Workflow

1. 지난 7일간 주요 AI 뉴스를 조사한다.
2. OpenAI, NVIDIA, Google, Anthropic을 우선 확인한다.
3. 제품 발표와 연구 발표를 구분한다.
4. 최소 2개 이상의 출처를 검증한다.
5. 결과를 한국어 평서체로 작성한다.

# Output

- 주요 뉴스
- 기술적 의미
- 비즈니스 영향
- 개발자 관점
- 향후 전망
```

그러면 Codex는 이 Skill을 사용할 때마다 같은 워크플로를 재현할 수 있다.

```text
AGENTS.md
    ↓
회사의 기본 운영 규칙

SKILL.md
    ↓
특정 업무의 SOP
```

## 4. MCP는 완전히 다른 역할이다

MCP(Model Context Protocol)는 Skill과 성격이 다르다. Skill이 “어떻게 일할 것인가?”**라면 MCP는 “어떤 시스템을 사용할 수 있는가?”를 정의한다. 예를 들어 Codex에게 다음과 같은 일을 시키고 싶다고 해보겠다.

```text
"GitHub에서 issue를 확인하고
관련 코드를 수정하고
테스트를 실행한 다음
PR을 만들어줘."
```

Codex에게 필요한 것은 다음과 같다.

```text
GitHub 접근
      ↓
GitHub MCP

Database 접근
      ↓
DB MCP

Slack 접근
      ↓
Slack MCP

내부 API 접근
      ↓
Custom MCP
```

구조적으로 보면 다음과 같다.

```text
Codex
  │
  ├── Skill
  │     └── "GitHub issue 처리 절차"
  │
  └── MCP
        └── GitHub
              │
              ├── list issues
              ├── read issue
              ├── create PR
              └── update issue
```

즉, Skill은 workflow이고 MCP는 capability이다.

## 5. 그래서 Agent Plugin이 등장한다

기존에는 Skill과 MCP가 별개였다. 예를 들어, 개발자가 AI 코드 리뷰 시스템을 만들었다면 Skill은 다음과 같이 구성할 수 있다.

```text
code-review/
└── SKILL.md
```

그리고 별도로 다음과 같은 MCP가 필요할 수 있다.

```text
github-mcp
sonarqube-mcp
```

여기에 환경 설정도 따로 필요하다. Agent Plugins는 이것을 하나의 package 안에 넣는다. Agent Plugins 1.0.0의 표준적인 구조는 다음과 같다.

```text
my-plugin/
│
├── plugin.json
│
├── skills/
│   └── code-review/
│       ├── SKILL.md
│       ├── scripts/
│       └── references/
│
├── mcp.json
│
├── LICENSE
└── CHANGELOG.md
```

즉 다음과 같은 구조다.

```text
Agent Plugin
     │
     ├── Skill
     │     └── 업무 프로세스
     │
     └── MCP
           └── 실행 능력
```

## 6. plugin.json은 무엇인가?

Agent Plugin의 루트에는 `plugin.json`이 존재한다. 가장 단순한 형태는 다음과 같다.

```json
{
  "$schema": "https://agent-plugins.org/schemas/1.0.0/plugin.schema.json",
  "name": "ai-research"
}
```

공식 Agent Plugins 문서 역시 가장 작은 플러그인을 이러한 형태로 설명한다. 조금 더 확장하면 개념적으로 다음과 같이 사용할 수 있다.

```json
{
  "$schema": "https://agent-plugins.org/schemas/1.0.0/plugin.schema.json",
  "name": "company-research",
  "version": "1.0.0",
  "description": "Company research agent plugin",
  "author": {
    "name": "Example Company"
  },
  "license": "MIT"
}
```

여기서 중요한 점이 있다. `plugin.json` 안에 Skill과 MCP 설정을 전부 넣는 방식이 아니다. Agent Plugins 1.0.0은 component discovery 위치를 표준화했다.

```text
plugin.json      → plugin metadata
skills/          → Agent Skills
mcp.json         → MCP configuration
```

## 7. MCP 설정까지 함께 묶는다

예를 들어 다음과 같은 구조가 있다고 하자.

```text
company-research/
├── plugin.json
├── skills/
│   └── company-analysis/
│       └── SKILL.md
└── mcp.json
```

`mcp.json`에는 다음과 같은 MCP 서버가 들어갈 수 있다.

```json
{
  "$schema": "https://agent-plugins.org/schemas/1.0.0/mcp.schema.json",
  "mcpServers": {
    "research-api": {
      "type": "streamable-http",
      "url": "https://research.example.com/mcp"
    }
  }
}
```

또는 로컬 MCP server라면 다음과 같이 구성할 수 있다.

```json
{
  "$schema": "https://agent-plugins.org/schemas/1.0.0/mcp.schema.json",
  "mcpServers": {
    "company-database": {
      "type": "stdio",
      "command": "./bin/company-mcp",
      "args": [
        "--data",
        "${PLUGIN_DATA}/company"
      ],
      "cwd": "${PLUGIN_ROOT}"
    }
  }
}
```

Agent Plugins 1.0.0은 MCP transport로 `stdio`, `streamable-http`, 그리고 선택적인 `sse`를 정의하고 있다. 또한 `${PLUGIN_ROOT}`와 `${PLUGIN_DATA}`라는 portable placeholder도 규정한다.

이것은 꽤 중요한 설계다. 왜냐하면 같은 Plugin package를 다음과 같은 서로 다른 환경에서 사용할 때 절대경로를 하드코딩하지 않아도 되기 때문이다.

```text
Developer A laptop
Developer B laptop
Codex
다른 호환 Agent client
```

## 8. AGENTS.md와 Agent Plugin의 관계

둘은 경쟁 관계가 아니며 함께 사용할 수 있다. 예를 들어, 회사 repository가 다음과 같이 구성되어 있다고 해보겠다.

```text
company-os/
│
├── AGENTS.md
│
├── plugins/
│   ├── research/
│   ├── content/
│   ├── sales/
│   └── operations/
│
├── knowledge/
├── projects/
└── reports/
```

`AGENTS.md`:

```text
우리 회사 전체 운영 원칙
```

Research Plugin:

```text
시장 조사 수행 능력
```

Content Plugin:

```text
블로그·문서 작성 능력
```

Sales Plugin:

```text
리드 분석과 영업 workflow
```

Operations Plugin:

```text
운영 자동화
```

전체적으로 보면 다음과 같은 구조가 된다.

```text
                 Company OS
                     │
                 AGENTS.md
                     │
             Company policies
                     │
         ┌───────────┼───────────┐
         │           │           │
      Research     Sales       Content
       Plugin      Plugin       Plugin
         │           │           │
      Skills       Skills      Skills
         │           │           │
        MCP          MCP         MCP
```

# 9. 플러그인 구성

| 기술          | 핵심 질문                   | 역할                     |
| ------------- | --------------------------- | ------------------------ |
| `AGENTS.md` | 어떻게 행동해야 하는가?     | 프로젝트/회사 규칙       |
| `SKILL.md`  | 이 업무는 어떻게 하는가?    | SOP / Workflow           |
| MCP           | 무엇을 사용할 수 있는가?    | Tools / Data / API       |
| Agent Plugin  | 이것들을 어떻게 배포하는가? | Packaging / Distribution |

지금까지 에이전트 생태계의 문제는 확장 기능의 이식성이었다. 예를 들어, Codex Skill, VS Code Agent, MCP config, Claude용 Skill, Cursor 설정 등이 각각 따로 존재하면 관리가 매우 어려워진다.

Agent Plugins가 노리는 방향은 다음과 같다.

    Agent Plugin

    ↓

    ┌─────────┼──────────┐
       ↓         ↓          ↓

    Codex     Cursor    VS Code
       ↓         ↓          ↓

    동일 Skill + MCP

Agent Plugins 공식 규격 역시 자신을 AI agents를 확장하는 reusable components를 distributable plugin으로 패키징하는 portable 규격으로 정의한다. VS Code도 이미 Agent Plugin 지원을 Preview로 문서화하고 있으며, VS Code 쪽 플러그인 모델에서는 Skills, MCP servers 외에도 slash commands, custom agents, hooks 같은 확장을 다룬다. 다만 이것은 Agent Plugins 1.0.0의 portable core와 VS Code 자체 client extension layer를 구분해서 봐야 한다.

# 11. Agent Plugins v1은 정확히 무엇을 표준화했는가?

현재 Agent Plugins Specification 1.0.0에서 portable core로 정의한 component type은 정확히 두 가지이다.
Agent Skills 과 MCP Servers 이다. 그래서 지금 단계에서:

`Agent Plugin = Skill + MCP + manifest`

라고 이해하는 것이 가장 정확하다. 다만 클라이언트마다 `hooks, custom agents, UI, commands, apps` 등을 자체 extension으로 추가할 수 있다. Agent Plugins 규격도 이런 경우를 위해 reverse-domain namespace 기반의 extensions 영역을 제공한다.

예:

```JSON
{
  "$schema": "...",

  "name": "my-plugin",

  "extensions": {

    "com.example.codex": {
      "something": {}
    }

  }
}
```

따라서 장기적으로는:

```
Portable core

Skills
MCP

      +

Client specific extension

UI
hooks
agents
commands
...
```

형태로 발전할 가능성이 크다.

# 12. OpenAI가 왜 지금 이걸 만드는가?

OpenAI 개발자 사이트의 현재 메시지도 꽤 명확하다. OpenAI는 Plugins를 ChatGPT와 Codex를 Skills, MCP servers, optional UI로 확장하는 수단으로 소개하고 있다.

결국 OpenAI가 만들려는 구조는 단순한 챗봇 ecosystem이 아니다. 제가 보기에는 기술적으로 다음 방향이다.

```
2023

Prompt -> LLM -> Answer
```

에서

```
2024

LLM -> Function Calling -> API
```

를 지나

```
2025

Agent -> MCP -> Tools
```

그리고 이제:

```
2026

Agent
 │
 ├── Instructions
 ├── Skills
 ├── MCP
 ├── Memory / Context
 ├── Computer Use
 └── Plugins
        │
        └── reusable capability packages
```

로 넘어가는 흐름이다. 즉, AI 모델을 사용하는 시대에서 AI에게 “직무”를 설치하는 시대로 이동하고 있는 것이다.

# 13. AI Researcher Plugin 예제

```
ai-researcher-plugin/

├── plugin.json
│
├── skills/
│
│   ├── company-research/
│   │     ├── SKILL.md
│   │     └── references/
│   │
│   ├── competitor-analysis/
│   │     └── SKILL.md
│   │
│   └── tech-news/
│         └── SKILL.md
│
├── mcp.json
│
└── README.md
```

Skill들은:

```
company-research → 기업 조사 방법
competitor-analysis → 경쟁사 분석 방법
tech-news → 기술 뉴스 분석 방법
```

을 정의한다. MCP는:

```
Web
GitHub
Company DB
News API
Google Drive
```

같은 실제 데이터 source에 접근한다. 그러면 사용자는 Codex에게 그냥 `NVIDIA Rubin Ultra 전략 조사해줘`라고 하면 된다.
Codex 내부에서는:

```
User

"NVIDIA Rubin Ultra 조사"

          ↓

Codex

          ↓

company-research Skill

          ↓

Research workflow

          ↓

MCP

 ├─ Web
 ├─ GitHub
 ├─ Docs
 └─ Database

          ↓

source verification

          ↓

analysis

          ↓

report
```

가 된다.
