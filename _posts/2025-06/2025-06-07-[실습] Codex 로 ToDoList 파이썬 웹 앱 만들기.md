---
title: "[실습] Codex 로 ToDoList 파이썬 웹 앱 만들기"
date: 2025-06-07
tags: [오픈AI, OpenAI, CodexCLI, Python, nodejs, ViveCoding, 바이브코딩]
typora-root-url: ../
toc: true
categories: [OpenAI, Python]
---

Codex CLI 관련 설정은 모두 끝났다. 그렇다면, 본격적으로 Codex를 사용해 파이썬 웹 앱들을 직접 만들어 보겠다. 

## 1. Codex 를 사용한 Hello, World 파이썬 앱

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



## 2. ToDoList 파이썬 웹 사이트 개발

* 좀더 복잡한 오늘의 할일을 정리하는 ToDoList 앱을 만들어 보겠다. 계속해서 언어는 Python을 사용하고 프롬프트 명령은 다음 그림과 같이 `python 언어를 이용해서 추가, 조회, 완료 표시, 삭제 기능을 포함하는 ToDoList 웹 사이트를 만들어서 localhost에서 실앵할 수 있도록 해줘.` 명령을 실행하면 된다.

  ![그림 20 - Codex 프롬프트 명령](/../images/2025-06/codexcli-20.png)

* 지난 Hello, World 앱 보다는 좀더 복잡하기 때문에 Codex가 파일 단위별로 사용자에게 확인한다. 명령어를 허락한다면 아래의 그림과 같이 Yes(Y)를 누르면 된다.

  ![그림 21 - 단위 파일별 생성](/../images/2025-06/codexcli-21.png)

* 모두 생성이 끝났다면, ToDoList-Python 폴더에 아래의 그림에서 보듯이 생성된 파일 및 디렉토리 구조를 보여준다. 또한 파일 관련해서 주요 구현 사항을 알려준다. 참고로 Flask 애플리케이션 구조와 SQLite 데이터베이스와 객체–관계 매핑(ORM) 라이브러리인 SQLAlchemy 를 사용했다.

  ![그림 22 - ToDoLIst 생성 설명](/../images/2025-06/codexcli-22.png)

* ToDoList 앱을 실행하기 위해 우선 관련된 의존 패키지 설치를 `pip install -r requirements.txt` 명령어로 실행한다.

  ![그림 23 - ToDoList 실행](/../images/2025-06/codexcli-23.png)

* 의존성 패키지 설치가 완료 되었다면, 이제 파이썬 애플리케이션을 `python app.py` 명령어로 실행한다.

  ![그림 24 - ToDoList 앱 실행](/../images/2025-06/codexcli-24.png)

* 이제 웹 브라우저에서 `http://localhost:5000` 웹 주소를 넣고 실행시키면 다음과 같은 화면이 나온다. 

  ![그림 25 - ToDoList 웹사이트 실행](/../images/2025-06/codexcli-25.png)



## 3. 참고

* 문서: [Codex in ChatGPT](https://help.openai.com/en/articles/11369540-codex-in-chatgpt)
* 문서: [OpenAI Codex CLI - Getting Started](https://help.openai.com/en/articles/11096431-openai-codex-cli-getting-started)
* 문서: [Enterprise admin getting started guide for Codex](https://help.openai.com/en/articles/11390924-enterprise-admin-getting-started-guide-for-codex)
* 문서: [Codex CLI and Sign in with ChatGPT](https://help.openai.com/en/articles/11381614-codex-cli-and-sign-in-with-chatgpt)
* 블로그:[Codex Changelog](https://help.openai.com/en/articles/11428266-codex-changelog) 