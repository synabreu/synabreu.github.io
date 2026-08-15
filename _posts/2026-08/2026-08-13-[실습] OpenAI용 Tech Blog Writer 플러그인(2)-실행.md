---
title: "[실습] OpenAI용 Tech Blog Writer 플러그인(2)-실행"
date: 2026-08-13
tags: [오픈AI, OpenAI, ChatGPT, MCP, Agent, Plugin,  ]
typora-root-url: ../
toc: true
categories: [openai]
---
[OpenAI용 Tech Blog Writer 플러그인(1)-개발]() 블로그에서는 개발 중심으로 설명했다. 그렇다면 이제 Codex 에서 어떻게 실행했는지 알아보도록 하겠다.

# 1. Tech Blog Writer 플러그인 메뉴 선택

![TechWriterBlog-01]({{ '/images/2026-08/TechWriterBlog-01.png' | relative_url }}){: width="30%"}

ChatGPT 서비스에서 PC, 맥, 웹 서비스 운영체제에 관계없이 왼쪽 메뉴의 서브 메뉴에 있는 `플러그인(Plugins)' 메뉴를 선택하라. 참고로 위의 그림에서 파란색 영역으로 표시했다.


# 2. Tech Blog Writer 플러그인 설치하려면?

![TechWriterBlog-02]({{ '/images/2026-08/TechWriterBlog-02.png' | relative_url }}){: width="70%"}

OpenAI Codex 에서 직접 만든 "Tech Blog Writer" 플러그인을 내 서비스에 설치하려면 오른쪽 화면 위에 보듯, `플러그인 설치` 버튼을 클릭하기만 하면 된다.


# 3. Tech Blog Writer 플러그인 사용하기

![TechWriterBlog-03]({{ '/images/2026-08/TechWriterBlog-03.png' | relative_url }}){: width="70%"}

그러면 위에서 보는 [그림]처럼 이제 Codex가 해당 Plugin을 로컬 환경에 등록하고 Agent 실행 대상으로 연결한 상태가 된다.


# 4. ChatGPT 플러그인 선택

이제 플러그인을 직접 사용하려면, ChatGPT에서 사용하려면 아래와 같다.

## 4.1 ChatGPT Chat 모드  

ChatGPT의 Chat 모드와 Work 모드에서의 플러그인 선택은 공통적으로 채팅 창에서 `@`를 누르면 아래의 리스트 콤보 박스에서 선택할 수 있다. 

![TechWriterBlog-04]({{ '/images/2026-08/TechWriterBlog-04.png' | relative_url }}){: width="70%"}


## 4.2 ChatGPT Work 모드  

서비스는 언제나 변경되겠지만 지금 시점에서는 Work 모드에서의 플러그인 사용이 좀더 편리하게 사용할 수 있도록 배치해 놓았다. 

![TechWriterBlog-04]({{ '/images/2026-08/TechWriterBlog-05.png' | relative_url }}){: width="70%"}


# 5. Tech Blog Writer 실행하려면?

Tech Blog Writer 플러그인을 선택한 다음 프롬프트를 입력하는 것처럼 명령을 하면 ChatGPT가 그 명령을 실행한다.

![TechWriterBlog-05]({{ '/images/2026-08/TechWriterBlog-06.png' | relative_url }}){: width="70%"}

그리고 나서 아래 화면에서 그 명령에 대한 실행 결과 화면이다.

![TechWriterBlog-06]({{ '/images/2026-08/TechWriterBlog-07.png' | relative_url }}){: width="70%"}


# 7. 플러그인 검색 화면

현재 플러그인 화면에서 보면 상당히 종류가 많다. 내가 만든 플러그인을 검색해서 설치 또는 실행하는 화면이다.

![TechWriterBlog-07]({{ '/images/2026-08/TechWriterBlog-08.png' | relative_url }}){: width="70%"}


# 9. 오픈 소스

* [Tech Blog Writer Plugin 오픈소스](https://github.com/synabreu/openai-blog-ap)
