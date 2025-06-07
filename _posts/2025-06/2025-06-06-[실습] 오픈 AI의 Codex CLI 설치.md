---
title: "[실습] 오픈 AI의 Codex CLI 설치"
date: 2025-06-06
tags: [오픈AI, OpenAI, CodexCLI, Python, nodejs, ViveCoding, 바이브코딩]
typora-root-url: ../
toc: true
categories: [OpenAI, Python]
---

이번 주 오픈AI에서 Codex에 관련 주요 업데이트가 있었다. Codex CLI의 첫 릴리즈는 지난 4월 16일에 이루어졌는 데, 개인적으로 바빠서 정식 릴리즈 기념으로 한번 정리해 보고자 한다. 

참고로 **Codex CLI**는  OpenAI에서 개발한 오픈소스 커맨드라인 도구로, 개발자가 터미널에서 자연어를 사용하여 코드를 생성, 수정, 실행할 수 있도록 지원하는 도구이다. 

요즘 유행하는 바이브코딩(ViveCoding)  도구와는 조금 거리가 멀다. 따라서, **ViveCoding**은 AI 기술을 활용하여 개발자와 비개발자 모두가 효율적으로 소프트웨어를 개발할 수 있도록 지원하는 플랫폼이다. 바이브코딩 플랫폼은 자연어 명령을 통해 코드를 생성하고, 테스트를 자동화하며, 다양한 개발 도구와 통합하여 생산성을 향상시키는 것을 목표로 하는 것이 Codex CLI 에이전트와 조금 다르다. 



## 1. Codex CLI 주요 업데이트

* 4월 16일: 개발자들이 터미널 환경에서 자연어를 사용하여 코드를 작성, 수정 및 실행할 수 있도록 지원하는 경량의 오픈소스 코딩 에이전트인 Codex CLI 정식 발표
* 6월 3일: Codex가 작업 실행 중 인터넷에 접근할 수 있는 기능이 추가되어, 의존성 설치, 패키지 업그레이드, 외부 리소스를 필요로 하는 테스트 실행.  [기본적으로 비활성화되어 있으며, Plus, Pro 및 Team 사용자들이 특정 환경에서 도메인 및 HTTP 메서드에 대한 세부적인 제어를 통해 활성화할 수 있음.](https://help.openai.com/en/articles/11428266-codex-changelog)
* 6월 4일: Codex CLI의 Rust 기반 재작성 소식이 발표. 보안성과 성능 향상을 목표로 하며, Node.js 및 TypeScript 기반에서 벗어나 네이티브 Rust 구현으로 전환되었음. 이를 통해 설치 시 의존성이 줄어들고, 메모리 사용량이 감소하며, 실행 속도가 향상될 것으로 기대됨.



## 2. Codex CLI 준비 사항

* Codex CLI는 윈도우와 맥OS 운영체제 모두 설치해서 사용할 수 있다.

* 비록 이번 주에 Rust로 재작성해서 네이티브 실행 파일 버전이 발표되어 배포되었지만 아직 정식버전이 아니어서 node.js 용 Codex CLI 버전으로 설치하도록 하겠다. 

  ![그림 1 - nodejs 웹사이트](/../images/2025-06/codexcli-01.png)

* 최신 버전인 Node.js 다운로드 (LTS)를 다운로드 하라. 다운로드 후 더블 클릭해서 순서대로 설치해 주면 된다. 특별히 예외적인 옵션은 없기 때문에 [다음] 버튼만 누르면 쉽게 설치할 수 있다.

  ![그림 2 - node.js 설치](/../images/2025-06/codexcli-02.png)

* node.js 설치 후, 명령 프롬프트를 실행시킨 다음에 node.js 가 올바르게 설치 되었는 지 `node --version` 명령을 통해 버전이 나오면 설치가 완료된 것이다.

  ![그림 3 - node 버전 체크](/../images/2025-06/codexcli-03.png)



## 3. Codex CLI 설치

* npm 을 이용해서 Codex CLI 를 아래의 그림과 같이 명령한다. 명령어는 `npm install -g openai/codex` 이다.

  ![그림 4 - codexcli 설치](/../images/2025-06/codexcli-04.png)

* 아래의 그림과 같이 보이면, codex가 설치 완료되었다.

  ![그림 5 - Codex 설치 완료](/../images/2025-06/codexcli-05.png)