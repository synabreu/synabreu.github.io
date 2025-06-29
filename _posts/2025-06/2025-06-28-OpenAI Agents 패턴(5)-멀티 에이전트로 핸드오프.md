---
title: "OpenAI Agents 패턴(5)-멀티 에이전트로 핸드오프"
date: 2025-06-27
tags: [오픈AI, OpenAI, Open Agents SDK, Swarm, Visual Studio Code, Multi-Agents, 멀티 에이전트]
typora-root-url: ../
toc: true
categories: [OpenAI]
---

멀티 에이진트로 핸드오프 방식은 세 가지 대상층, 즉 성인, 청소년, 어린이를 위한 텍스트를 생성하는 것이다. Research Agent는 정보를 수집한 다음 세 개의 에이전트 중 하나에게 이를 넘긴다. 각 에이전트는 LLM과 통신하지만, 이는 에이전트의 내부 기능으로 간주할 수 있다. 



## 1. 멀티 에이전트로 핸드오프 구현

```python
import streamlit as st
import asyncio

from agents import Agent, Runner, handoff

adult_writer_agent = Agent(
    name="Adult Writer Agent",
    instructions=f"""Write the article based on the information given that it is suitable for adults interested in culture.
                    """, 
    model="o4-mini",
)

teen_writer_agent = Agent(
    name="Teen Writer Agent",
    instructions=f"""Write the article based on the information given that it is suitable for teenagers who want to have a cool time.
                    """, 
    model="o4-mini",
)

kid_writer_agent = Agent(
    name="Kid Writer Agent",
    instructions=f"""Write the article based on the information given that it is suitable for kids of around 8 years old. 
                    Be enthusiastic!
                    """, 
    model="o4-mini",
)

researcher_agent = Agent(
    name="Research agent",
    instructions=f"""Find information on the topic(s) given.""",

    model="o4-mini",
    handoffs = [kid_writer_agent, teen_writer_agent, adult_writer_agent]
)

async def run_agent(input_string):
    result = await Runner.run(researcher_agent, input_string)
    return result

# Streamlit UI

st.title("Writer Agent3")
st.write("Write stuff for adults, teenagers or kids.")

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

* 작업을 넘길 수 있는 에이전트 집합과 이를 포함한 목록이 Research Agent에 있음
* 각 에이전트에 있는 지시사항은 자명하며, 프로그램은 “어린이를 위한 프랑스 파리에 대한 에세이를 써줘” 또는 “청소년을 위한…” 혹은 “성인을 위한…”과 같은 프롬프트에 올바르게 응답함
* Research Agent는 작업에 적합한 Writer Agent를 정확히 선택하게 됨
