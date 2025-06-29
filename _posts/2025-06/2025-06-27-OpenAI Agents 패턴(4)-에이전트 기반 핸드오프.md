---
title: "OpenAI Agents 패턴(4)-에이전트 기반 핸드오프"
date: 2025-06-26
tags: [오픈AI, OpenAI, Open Agents SDK, Swarm, Visual Studio Code, Multi-Agents, 멀티 에이전트]
typora-root-url: ../
toc: true
categories: [OpenAI]
---

에이전트는 이전에 어떤 일이 있었는지를 알아야 할 필요가 있다. 이럴 때 OpenAI의 에이전트 기반 핸드오프(Agentic Handoff)를 사용하는 데,  에이전트 기반 핸드오프(Agentic Handoff)는 프로그래밍 방식의 핸드오프(Programmatic Handoff)와 매우 유사하지만, 두 번째 에이전트로 전달되는 데이터가 다르고, 핸드오프가 필요하지 않은 경우 첫 번째 에이전트에서 출력이 발생할 수 있다는 점이 차이다. 



## 1. 에이전트 기반 핸드오프 소스

```python
import streamlit as st
import asyncio
from agents import Agent, Runner

kids_writer_agent = Agent(
    name="Kids Writer Agent",
    instructions=f"""Re-write the article so that it is suitable for kids aged around 8. 
                     Be enthusiastic about the topic - everything is an adventure!""",
    model="o4-mini",
)

researcher_agent = Agent(
    name="Research agent",
    instructions=f"""Answer the query and report the results.""",
    model="o4-mini",
    handoffs = [kids_writer_agent]
)

async def run_agent(input_string):
    result = await Runner.run(researcher_agent, input_string)
    return result

# Streamlit UI

st.title("Writer Agent2")
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

* 이전 프로그래밍 방식의 핸드오프 방식 달리, `researcher_agent`에 있는 `handoffs` 리스트이다.
* Research Agent는 작업을 완료한 후 Kid’s Writer Agent로 핸드오프할 수 있도록 허용되어 있음
* 이 방식의 효과는 Kid’s Writer Agent가 처리의 제어권을 넘겨받을 뿐만 아니라, Research Agent가 수행한 작업 내용과 원래의 프롬프트에 대한 정보도 함께 갖게 된다.
* 핸드오프가 이루어질지는 에이전트가 스스로 판단함
* 에이전트에게 아이들을 위한 글을 쓰라고 지시했기 때문에 Kid’s Writer Agent로 핸드오프가 발생합니다. 만약 그런 지시가 없었다면, 단순히 원래의 텍스트만 반환했을 것임

