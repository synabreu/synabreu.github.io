---
title: "OpenAI Agents 패턴(2)-Tool Use Agent"
date: 2025-06-26
tags: [오픈AI, OpenAI, Open Agents SDK, Swarm, Visual Studio Code, Multi-Agents, 멀티 에이전트]
typora-root-url: ../
toc: true
categories: [OpenAI]

---



## 1. 단일 도구를 사용한 단일 에이전트 패턴

* 에이전트는 툴을 사용할 수 있으며, **에이전트는 LLM과 협력하여 필요한 경우 어떤 툴을 사용할지 결정**합니다. 아래는 **툴을 사용하는 에이전트의 데이터 흐름 다이어그램**

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

* **Agents 와 Wikipedia 라이브러리 import (**Wikipedia는 Tool로 사용)
* **도구(Tool) 정의** – `@function_tool` decorator를 사용한 함수 정의
* 도구를 실행하는 에이전트 함수(run_agent)  정의
* Streamlit 앱에서 에이전트를 실행하고 결과를 출력