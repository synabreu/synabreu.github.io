---
title: "OpenAI Agents 패턴(2)-Tool Use Agent"
date: 2025-06-26
tags: [오픈AI, OpenAI, Open Agents SDK, Swarm, Visual Studio Code, Multi-Agents, 멀티 에이전트]
typora-root-url: ../
toc: true
categories: [OpenAI]
---

에이전트는 도구를 사용할 수 있으며, 에이전트는 LLM과 협력해 어떤 도구를 사용할지(또는 사용할 필요가 없는지)를 결정한다. 이번 노트에서는 단일 도구를 사용한 단일 에이전트 패턴에 대해 알아보자!



## 1. 단일 도구를 사용한 단일 에이전트 패턴

* 단순한 에이전트와 유사하지만, 에이전트가 활용하는 추가적인 과정인 도구(tool)를 확인할 수 있음

* 에이전트가 LLM에 요청을 보내면, 응답에는 도구가 필요한지 여부가 포함되어 있음

* 만일 도구가 필요하다면, 에이전트는 해당 도구를 호출하고 그 결과를 다시 LLM에 제출함

* 다시 LLM의 응답을 통해 또 다른 도구 호출이 필요한지 여부가 결정됨

* 에이전트는 LLM이 더 이상 도구의 입력을 필요로 하지 않을 때까지 이 과정을 반복함

* 그런 다음 에이전트는 사용자에게 응답할 수 있음

  ​       

## 2. 프로그램 소스 분석

```python
# 1. Agents와 Wikipedia 라이브러리 import
import streamlit as st
import asyncio
from agents import Agent, Runner, function_tool
import wikipedia

# 2. 도구(Tool) 정의 – `@function_tool` decorator를 사용한 함수 정의
@function_tool
def wikipedia_lookup(q: str) -> str:
    """Look up a query in Wikipedia and return the result"""
    return wikipedia.page(q).summary

research_agent = Agent(
    name="Research agent",
    instructions="""You research topics using Wikipedia and report on 
                    the results. """,
    model="o4-mini",
    tools=[wikipedia_lookup],
)

# 3. 도구를 실행하는 에이전트 함수 정의
async def run_agent(input_string):
    result = await Runner.run(research_agent, input_string)
    return result.final_output

# 4. Streamlit 앱에서 에이전트를 실행하고 결과를 출력
st.title("Simple Tool-using Agent")
st.write("This agent uses Wikipedia to look up information.")

user_input = st.text_input("Enter a query and press 'Send':")

st.write("Response:")
response_container = st.container(height=300, border=True)

if st.button("Send"):
    response = asyncio.run(run_agent(user_input))
    with response_container:
        st.markdown(response)
```

* 프로그램 플로우
  * **Agents 와 Wikipedia 라이브러리 import (**Wikipedia는 Tool로 사용)
  * **도구(Tool) 정의** – `@function_tool` decorator를 사용한 함수 정의
  * 도구를 실행하는 에이전트 함수(run_agent)  정의
  * Streamlit 앱에서 에이전트를 실행하고 결과를 출력