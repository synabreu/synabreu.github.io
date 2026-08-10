---
title: "OpenAI Agent Plugins란 무엇인가"
date: 2026-08-08
tags: [openai, gpt5, 오픈AI, AgentPlugins, Apple, AppleIntelligence, ios, ChatGPT, Google]
typora-root-url: ../
toc: true
categories: [openai]
---
오늘은 오픈AI 플러그인에 대해 좀더 기술적으로 파헤쳐보자! **`AGENTS.md`는 규칙, Skill은 업무 절차, MCP는 외부 도구 연결, Agent Plugin은 이 둘을 배포 가능한 하나의 패키지로 묶는 규격** 이라는 것을 잘 알고 있을 것이다. Agent Plugins 1.0.0 규격에서는 핵심 구성요소로 **Agent Skills와 MCP servers 두 종류**를 정의하고 있다.

# 1. 전체 구조

<그림>

여기서 Agent Plugin은 새로운 종류의 에이전트가 아니다.  오히려 기존에 따로 관리하던 **Skill과 MCP 서버 같은 에이전트 확장 요소를 하나의 이동 가능한 package로 만들기 위한 컨테이너 규격**에 가깝다. 현재 Agent Plugins 공식 규격은 이를 특정 벤더에 종속되지 않는 호환가능한 포맷()portable format)으로 정의하고 있다.

# 2. AGENTS.md는 무엇인가?

AGENTS.md는 Codex가 특정 프로젝트에서 어떤 방식으로 행동해야 하는지 알려주는 프로젝트 운영 규칙입니다.

예를 들어:

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

Codex 입장에서는 이것이 일종의 회사 또는 프로젝트의 규율같은 역할을 한다. 예를 들어, 다음 질문에 답한다.

```
우리 프로젝트에서는
어떤 방식으로 코딩해야 하는가?

어떤 명령어를 사용해야 하는가?

어떤 파일을 수정하면 안 되는가?

테스트는 어떻게 하는가?

어떤 아키텍처 원칙을 따라야 하는가?
```

여기서 중요한 점은 AGENTS.md 자체가 도구도 아니고 프로그램도 아니라는 것이다. 외부 시스템을 호출하지도 않고 새로운 기능을 설치하지도 않는다.  그저 Codex에게 지속적인 행동 지침과 컨텍스트를 준다.

# 3. Skill은 무엇인가?

Skill은 한 단계 더 나간다. Skill은 Codex에게:

> 이 업무를 할 때는 이렇게 처리해.

라고 알려주는 재사용 가능한 업무 매뉴얼과 같다. OpenAI는 Codex use case에서도 반복되는 workflow를 Skill로 저장해 다시 사용할 수 있는 방식을 소개하고 있다. 예를 들어, 회사에서 매주 AI 뉴스를 분석한다고 하자!

```
skills/
   ai-news-report/
       SKILL.md
       scripts/
       references/
```

`SKILL.md:`

```

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

그러면 Codex는 이 Skill을 사용할 때마다 같은 workflow를 재현할 수 있다.

```
AGENTS.md
   ↓
회사의 기본 운영 규칙

SKILL.md
   ↓
특정 업무의 SOP
```

# 4. MCP는 완전히 다른 역할이다

MCP(Model Context Protocol)는 Skill과 성격이 다르다. Skill이 “어떻게 일할 것인가”라면 MCP는 “어떤 시스템을 사용할 수 있는가?”를 정의한다. 예를 들어, Codex에게 이런 일을 시키고 싶다고 해보겠다.

```
"GitHub에서 issue를 확인하고
관련 코드를 수정하고
테스트를 실행한 다음
PR을 만들어줘."
```

Codex에게 필요한 것은 다음과 같습니다.

```
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

```
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

Skill은 workflow이고 MCP는 capability이다. 

# 5. 그래서 Agent Plugin이 등장한다

기존에는 Skill과 MCP가 별개였다. 예를 들어, 개발자가 AI 코드 리뷰 시스템을 만들었다면:

```Skill

code-review/
   SKILL.md
```

그리고 별도로:

```MCP

github-mcp
sonarqube-mcp
```

를 설정해야 했다. 여기에 환경 설정도 따로 필요하다. Agent Plugins는 이것을 한 package 안에 넣는다. Agent Plugins 1.0.0의 표준적인 구조는 다음과 같다.

```
my-plugin/

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

즉:

```
Agent Plugin
     │
     ├── Skill
     │     └── 업무 프로세스
     │
     └── MCP
           └── 실행 능력
```

이다.

# 6. plugin.json은 무엇인가?

Agent Plugin의 루트에는 plugin.json이 존재하다. 가장 단순한 형태는 다음과 같다.

```JSON
{
  "$schema": "https://agent-plugins.org/schemas/1.0.0/plugin.schema.json",
  "name": "ai-research"
}
```

공식 Agent Plugins 문서 역시 가장 작은 플러그인을 이러한 형태로 설명한다. 조금 더 확장하면 개념적으로:

```JSON
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

처럼 사용할 수 있다. 여기서 중요한 점이 있다. plugin.json 안에 Skill과 MCP 설정을 전부 넣는 방식이 아니다. Agent Plugins 1.0.0은 component discovery 위치를 표준화했다.

```
plugin.json      → plugin metadata
skills/          → Agent Skills
mcp.json         → MCP configuration
```

# 7. MCP 설정까지 함께 묶는다

예를 들어:

```
company-research/
├── plugin.json
├── skills/
│   └── company-analysis/
│       └── SKILL.md
└── mcp.json
```

이라고 하자! `mcp.json`에는 다음과 같은 MCP 서버가 들어갈 수 있다.

```JSON
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

또는 로컬 MCP server라면:

```JSON
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

Agent Plugins 1.0.0은 MCP transport로 stdio, streamable-http, 그리고 선택적인 sse를 정의하고 있다. 또한 ${PLUGIN_ROOT}와 ${PLUGIN_DATA}라는 portable placeholder도 규정한다. 

이게 꽤 중요한 설계이다. 왜냐하면 같은 Plugin package를:

```
Developer A laptop
Developer B laptop
Codex
다른 호환 Agent client
```

에서 사용할 때 절대경로를 하드코딩하지 않아도 되기 때문이다.

# 8. AGENTS.md와 Agent Plugin의 관계

둘은 경쟁 관계가 아니고 같이 써야 한다. 예를 들어 회사 repository가:

```
company-os/

├── AGENTS.md
│
├── plugins/
│   │
│   ├── research/
│   ├── content/
│   ├── sales/
│   └── operations/
│
├── knowledge/
├── projects/
└── reports/
```

라고 해보겠다.

AGENTS.md:
```
우리 회사 전체 운영 원칙
```

research plugin:
```
시장 조사 수행 능력
```

content plugin:
```
블로그·문서 작성 능력
```

sales plugin:
```
리드 분석과 영업 workflow
```

operations plugin:
```
운영 자동화
```

가 된다. 다음과 같은 구조로 구성된다.

```

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

