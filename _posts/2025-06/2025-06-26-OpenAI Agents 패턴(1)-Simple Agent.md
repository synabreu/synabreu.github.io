---
title: "OpenAI Agents 패턴(1)-Simple Agent"
date: 2025-06-26
tags: [오픈AI, OpenAI, Open Agents SDK, Swarm, Visual Studio Code, Multi-Agents, 멀티 에이전트]
typora-root-url: ../
toc: true
categories: [OpenAI]
---





## 1. 간단한 에이전트 호출 패턴

* 실행 중인 에이전트를 하나의 프로세스로 표현하고, 그 안팎으로 데이터가 흐르는 방식

  ```python
  import streamlit as st
  import asyncio from agents 
  import Agent, Runner 
  ```

*  에이전트의 실행이 완료될 때까지 기다리는 비동기처리 하기 위해  **Streamlit 과 asyncio 패키지**는 필요

  ```python
  agent = Agent(name="Assistant", instructions="You are a helpful assistant")
  
  async def run_agent(input_string):
      result = await Runner.run(agent, input_string)
      return result.final_output 
  ```

* Streamlit 함수들을 이용해 사용자 인터페이스(UI) 정의

  ```python
  st.title("Simple Agent SDK Query")
  
  user_input = st.text_input("Enter a query and press 'Send':")
  
  st.write("Response:")
  response_container = st.container(height=300, border=True)
  
  if st.button("Send"):
      response = asyncio.run(run_agent(user_input))
      with response_container:
          st.markdown(response)
  ```

* 사용자는 질문을 입력하고 ‘Send’ 버튼을 누를 때, 

  <응답 메시지 결과 화면>

* 루프를 한 번만 돌고 끝남

  ```python
  from agents import set_default_openai_api
  set_default_openai_api("chat_completions")
  ```

* 단순화를 위해 기본 Response API를 사용

