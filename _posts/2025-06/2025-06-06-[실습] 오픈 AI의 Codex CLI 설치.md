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



## 3. Codex CLI 설치 및 준비 사항

* npm 을 이용해서 Codex CLI 를 아래의 그림과 같이 명령한다. 명령어는 `npm install -g openai/codex` 이다.

  ![그림 4 - codexcli 설치](/../images/2025-06/codexcli-04.png)

* 아래의 그림과 같이 보이면, codex가 설치 완료되었다.

  ![그림 5 - Codex 설치 완료](/../images/2025-06/codexcli-05.png)

* Codex CLI 자체는 무료로 설치하고 실행할 수 있지만, 실제로 코드를 생성하거나 실행하려면 API 크레딧이 있어야 한다. 따라서, 아래의 그림과 같이 Codex CLI를 사용하는 데 OpenAI API에 사용 가능한 **크레딧**이 있어야 한다. 또한, ChatGPT **Plus** 사용자는 Codex CLI 로그인 시 **$5**, **Pro** 사용자는 **$50**의 프로모션 크레딧을 제공합니다. 단, 이는 **30일만 유효**하며, 신용카드 등록과 가입 7일 경과 등의 조건을 만족해야 지급됨을 알아둬야 한다. (이 정책은 회사 사정에 따라 변경될 수 있음)

  ![그림 6 - OpenAI Credit](/../images/2025-06/codexcli-06.png)



## 4. Codex CLI 실행 시 필수 사항

* Codex CLI을 실행하려면, OpenAI의 Secret 키를 생성하거나 기존의 Secret 키를 이용하면 된다. 

  ![그림 7 - Secret Key](/../images/2025-06/codexcli-07.png)

* 만일 새로운 Secret 키를 생성하려면,  [Create new secret key] 를 눌러 생성하면 된다. 아래와 같이 Name 에 CodexCLI, Project는 Default project로 설정한 다음 [Create  secret key] 버튼을 누른다. 그리고 생성한 Secret Key는 잃어 버리지 않게 반드시 메모장에게 복사해둔다. 

  ![그림 8 - Create new secret key](/../images/2025-06/codexcli-08.png)

* 명령 창에서  `set OPENAI_API_KEY=<YOUR SECRET KEY>` 설정한다. 이때 <YOUR_SECRET_KEY> 부분에 위에서 복사한 Secret Key 를 그대로 복사하고 실행한다.

  ![그림 9 - OPENAI API 설정](/../images/2025-06/codexcli-09.png)



## 5. Codex CLI 실행

* 이제 본격적으로 Codex CLI를 실행하기 위해 명령 창에서`codex` 명령을 실행한다. 또한 아래의 그림과 같이 "Git 저장소 밖에서 코딩 에이전트를 실행하는 것은 위험할 수 있습니다.
   변경 사항을 되돌리고 싶을 때 어려움이 생길 수 있기 때문입니다.
   계속하시겠습니까?" 경고 창이 나오는 데, `Y` 를 눌러 주면 된다.

  ![그림 10 - Codex 명령](/../images/2025-06/codexcli-10.png)

* 먼저 codex의 다양한 정보를 살펴 볼 수 있다. OpenAI codex 의 버전과 localhost의 세션 정보, 그리고 현재 여러분이 codex를 실행한 workdir 그리고 model 명 (현재 그림에서는 디폴트로 codex-mini-latest로 설정), provider는 openai 이다. approval은 suggest 가 설정되어 있다. 

  ![그림 11 - Codex Help](/../images/2025-06/codexcli-11.png)

* help를 실행하면, 아래와 같이 codex에서 사용할 수 있는 명령어와 설명이 간단하게 나온다.

  ![그림 12 - help 결과](/../images/2025-06/codexcli-12.png)

* 명령 창에서`/model` 명령을 입력하고 실행하면, 현재 사용할 수 있는 모델들이 나온다. codex-min-latest 모델은 기본이고, 그외 여러분들이 바꾸려면 커서를 통해 아래를 움직여 원하는 모델을 선택하면 된다. 

  ![그림 13 - Codex 모델](/../images/2025-06/codexcli-13.png)



## 6. Codex 를 사용한 Hello, World 파이썬 앱

* 그러면 간단히 앱이 어떻게 생성되는 지, Python 언어를 사용해서 Hello, World 문자열을 출력해보자.

   ![그림 14 - codex 명령](/../images/2025-06/codexcli-14.png)

* 이제 codex 가 자동적으로 HelloWorld 폴더에 파이썬 코드를 생성한다. 이때, 여러 개의 코드가 존재한다면 파일 하나 씩 사람이 확인할 수 있는 옵션(y)를 선택할 수 있고, 그렇지 않고 자동적으로 생성하고 싶다면, 옵션(a)를 선택하면 된다. 아래의 그림은 최종 생성 화면이다.

  ![그림 15 - Codex 생성 결과](/../images/2025-06/codexcli-15.png)

* 생성한 Hello, Python 소스를 실행하는 것은 두 가지 방법이 있다. 첫번쩨, 코드가 생성된 디렉토리에서 직접 `python3 hello.py` 명령을 실행하는 방법이 있다.

  ![그림 16 - 명령창에서 Hello World 파이썬앱 실행](/../images/2025-06/codexcli-16.png)

* 두 번째 방법은 codex 창에서 `python3 hello.py` 명령을 하는 경우이다. 

  ![](/../images/2025-06/codexcli-17.png)

* 그러면 codex가 자동적으로 실행하는 데, 간단한 코드일지라도 내부의 처리가 많아서 아래의 그림에서 보는 것 같이 명령 창에서 파이썬 코드를 실행하는 것 보다는 내부적으로 처리하는 것이 많고 시간이 조금 더 오래 걸린다.

   ![그림 18 - 내부 동작](/../images/2025-06/codexcli-18.png)

* 자동적으로 내부 처리가 끝나면, 아래의 그림과 같이 codex의 실행 결과를 확인할 수 있다.

  ![그림 19 - Codex 결과 화면](/../images/2025-06/codexcli-19.png)



## 7. 참고

* 문서: [Codex in ChatGPT](https://help.openai.com/en/articles/11369540-codex-in-chatgpt)

* 문서: [OpenAI Codex CLI - Getting Started](https://help.openai.com/en/articles/11096431-openai-codex-cli-getting-started)

* 문서: [Enterprise admin getting started guide for Codex](https://help.openai.com/en/articles/11390924-enterprise-admin-getting-started-guide-for-codex)

* 문서: [Codex CLI and Sign in with ChatGPT](https://help.openai.com/en/articles/11381614-codex-cli-and-sign-in-with-chatgpt)

* 블로그:[Codex Changelog](https://help.openai.com/en/articles/11428266-codex-changelog) 

  