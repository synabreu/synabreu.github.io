---
title: "OpenAI Agents 패턴(3)-프로그래밍 방식의 핸드오프"
date: 2025-06-26
tags: [오픈AI, OpenAI, Open Agents SDK, Swarm, Visual Studio Code, Multi-Agents, 멀티 에이전트]
typora-root-url: ../
toc: true
categories: [OpenAI]
---

많은 에이전트 애플리케이션은 단일 에이전트만 필요하며, 이러한 것들만으로도 ChatGPT와 같은 LLM 채팅 인터페이스에서 제공되는 단순한 채팅 응답 생성보다 한 단계 발전한 것이다. 에이전트는 루프를 돌며 실행되고 도구를 사용할 수 있어서, 단일 에이전트만으로도 상당히 강력하다. 그러나 멀티 에이전트가 함께 작동하면 훨씬 더 복잡한 동작을 달성할 수 있다.

OpenAI는 다른 프레임워크들처럼 에이전트 오케스트레이션 추상화를 도입하려 하지 않는다. 그러나 단순한 설계에도 불구하고, 단순한 구성과 복잡한 구성을 모두 구축할 수 있도록 지원한다. 한 에이전트가 다른 에이전트에게 제어권을 넘기는 **핸드오프(Handoff)**에 대해 정리하도록 하자. 



## 1. 프로그래밍 방식의 핸드오프

```python
import streamlit as st
import asyncio
from agents import Agent, Runner

writer_agent = Agent(
    name="Writer agent",
    instructions=f"""Re-write the article so that it is suitable for kids
                     aged around 8. Be enthusiastic about the topic -
                     everything is an adventure!""",
    model="o4-mini",
)

researcher_agent = Agent(
    name="Research agent",
    instructions=f"""You research topics and report on the results.""",
    model="o4-mini",
)

async def run_agent(input_string):
    result = await Runner.run(researcher_agent, input_string)
    result2 = await Runner.run(writer_agent, result.final_output)
    return result2

# Streamlit UI

st.title("Writer Agent")
st.write("Write stuff for kids.")

user_input = st.text_input("Enter a query and press 'Send':")

st.write("Response:")
response_container = st.container(height=300, border=True)

if st.button("Send"):
    response = asyncio.run(run_agent(user_input))
    with response_container:
        st.markdown(response.final_output)
    st.write(response)
    st.json(response.raw_responses)
```

* 두 개의 에이전트 정의함 - 하나는 주제를 조사하고, 다른 하나는 어린이에게 적합한 텍스트를 생성

* 단순히 첫 번째 에이전트를 실행하여 결과(result)를 얻고, 그 결과를 두 번째 에이전트의 입력으로 사용하는 방식(result2에 출력 됨)

* 전통적인 프로그래밍에서 하나의 함수 출력값을 다음 함수의 입력값으로 사용하는 것과 동일함

  
