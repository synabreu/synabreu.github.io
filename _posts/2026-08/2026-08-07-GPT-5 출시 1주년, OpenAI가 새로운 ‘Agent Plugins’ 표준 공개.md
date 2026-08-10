---
title: "GPT-5 출시 1주년, OpenAI가 새로운 ‘Agent Plugins’ 표준 공개"
date: 2026-08-07
tags: [openai, gpt5, 오픈AI, AgentPlugins, Apple, AppleIntelligence, ios, ChatGPT, Google]
typora-root-url: ../
toc: true
categories: [openai]
---
OpenAI의 GPT-5가 오늘 출시 1주년을 맞았다. OpenAI는 이를 기념하는 이번 주에 개별 모델 자체를 넘어, AI 에이전트를 중심으로 한 새로운 생태계를 바라보기 위해 에이전트 플러그인(Agent Plugins)를 공개했다. Agent Plugins는 재사용 가능한 AI 에이전트 확장 기능을 여러 호환 제품에서 사용할 수 있도록 하기 위한 개방형 표준(Open Standard) 기술이다.

# 1. GPT-5의 첫 1년

GPT-5는 2025년 8월 7일 출시되었으며, ChatGPT의 기본 모델로 GPT-4o와 여러 추론 모델을 대체했다. GPT-5를 특징짓는 가장 중요한 기능은 오토매틱 라우팅(Automatic Routing)이었다. 사용자의 요청에 따라 ChatGPT가 빠르게 답변하거나, 필요한 경우 더 깊은 추론 과정을 자동으로 호출하는 구조였다.

OpenAI는 또한 GPT-5가 GPT-4o보다 사실 오류를 줄였다고 설명했다. Apple 역시 iOS 26, iPadOS 26, macOS Tahoe 26에서 GPT-5를 채택했다. 애플 인텔리전스(Apple Intelligence)를 통해 전달되는 ChatGPT 요청도 GPT-4o에서 GPT-5로 변경됐다. 여기에는 Siri에서 ChatGPT로 요청을 전달하는 기능, Writing Tools, Visual Intelligence 등이 포함됐다.

이후 OpenAI는 GPT-5 계열 모델을 지속적으로 확장했다. 2025년 11월에는 보다 따뜻하고 자연스러운 답변을 목표로 한 GPT-5.1이 등장했다. 12월에는 Gemini와의 경쟁 속에서 전문적인 업무 수행 능력을 강화한 GPT-5.2가 출시됐다. 당시 OpenAI 내부에서는 이 대응을 이른바 ‘Code Red’라고 불렀다.

그 이후 ChatGPT의 다소 어색하거나 과도한 말투를 개선하기 위한 GPT-5.3 Instant가 등장했고, 이틀 뒤에는 100만 토큰 컨텍스트 윈도우와 컴퓨터 사용 기능을 내장한 GPT-5.4 Thinking이 공개됐다. OpenAI는 이후 GPT-5.4 mini와 nano, 보안 목적의 제한된 GPT-5.4-Cyber, 보다 자율적인 작업 수행을 위한 GPT-5.5, GPT-5.5 Instant를 차례로 공개했다.

그리고 2026년 7월에는 GPT-5.6 계열인 Sol, Terra, Luna 모델이 등장했다.

# 2. Codex Mac 앱에서 새로운 ChatGPT까지

모델만 변화한 것은 아니다. 이 모델들을 둘러싼 OpenAI의 소프트웨어 환경 역시 크게 달라졌다. OpenAI는 2026년 2월 멀티 코딩 에이전트를 동시에 실행하고, 코드 변경 사항을 검토하며, 작업을 예약할 수 있는 허브 역할의 Codex 전용 Mac 앱을 출시했다.

7월에는 Codex가 사실상 새로운 ChatGPT 데스크톱 앱으로 발전했다. Chat, Work, Codex 기능이 하나의 앱으로 통합됐으며, 기존 네이티브 ChatGPT 클라이언트는 ChatGPT Classic이라는 이름으로 변경됐다. 지난 1년 동안 ChatGPT 사용 방식에서 가장 큰 변화를 가져온 것은 Codex라고 평가했다. 비판적인 판단이나 깊은 사고가 반드시 필요하지 않은 많은 작업을 Codex에 맡길 수 있게 됐으며, 이전에는 직접 만들기 어려웠던 Mac용 도구들도 구축할 수 있게 됐다는 설명이다.

# 3. OpenAI, Agent Plugins 공개

OpenAI가 다음 단계로 공개한 것은 에이전트 플러그인(Agent Plugins)다.에이전트 플러그인은 Skills와 MCP 서버를 하나의 이식 가능한 패키지 형태로 묶을 수 있도록 설계된 새로운 표준이다.

Agent Plugins 공식 설명에 따르면 다음과 같다.

> Agent Plugins는 재사용 가능한 구성 요소를 이식 가능한 플러그인으로 패키징하기 위한 개방형 벤더 중립 표준이다. 버전 1.0.0 규격에서는 Agent Skills와 MCP 서버를 위한 공통 형식을 정의하며, 호환 클라이언트가 이를 일관된 방식으로 검색하고 불러올 수 있도록 한다.

Agent Plugins 기술은 공개 라이선스를 기반으로 하며, 공개된 환경에서 개발된다. 특정 AI 기업이나 플랫폼에 종속되지 않는 벤더 중립(Vendor-Neutral) 프로젝트라는 점도 중요한 특징이다. 프로젝트의 운영위원회에는 아마존, 커서, 마이크로소프트, 오픈AI, 버셀(Vercel) 기업들이 참여하고 있다. 즉, Agent Plugins는 특정 회사의 전용 플러그인 기술이라기보다, 여러 AI 에이전트 플랫폼에서 공통으로 사용할 수 있는 확장 기능 표준을 목표로 하고 있다.

# 4. 새로운 GPT-6는 언제 등장할까?

OpenAI는 아직 GPT-6를 공식 발표하지 않았다. 현재 가장 분명한 단서는 아직 공개되지 않은 **Astra 모델 패밀리**다. OpenAI는 최근 Astra의 내부 버전이 오랫동안 해결되지 않았던 수학 및 컴퓨터 과학 문제 10개를 진전시켰다고 발표했다.

하지만 OpenAI는 Astra를 GPT-6라고 부르지는 않았다. 향후 모델의 출시 시점이 OpenAI의 결정만으로 정해지지 않을 가능성도 있다. GPT-5.6의 공개 배포 역시 미국 정부의 보안 검토 때문에 잠시 보류된 바 있다. 따라서 Astra가 실제 출시 직전 단계일 수도 있지만, 그렇다고 반드시 곧바로 출시된다는 의미는 아니다.

또 Astra가 GPT-6가 아니라 또 다른 GPT-5 계열 모델 이름으로 등장할 가능성도 있다. 지난 1년 동안 ChatGPT 사용 방식에 가장 큰 변화를 가져온 것은 단순히 새로운 모델의 등장이 아니라 **Codex라는 에이전트형 도구의 등장**이었다.
