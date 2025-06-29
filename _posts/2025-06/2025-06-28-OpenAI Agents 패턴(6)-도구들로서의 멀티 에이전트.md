---
title: "OpenAI Agents 패턴(6)-도구들로서의 멀티 에이전트"
date: 2025-06-27
tags: [오픈AI, OpenAI, Open Agents SDK, Swarm, Visual Studio Code, Multi-Agents, 멀티 에이전트]
typora-root-url: ../
toc: true
categories: [OpenAI]
---

에이전트를 실행하는 것은 함수를 호출하는 것과 같이 도구(tool)를 호출하는 것이다. 그렇다면 왜 에이전트를 지능적인 도구처럼 사용하지 않는 걸까? 전체 제어권을 새로운 에이전트에게 넘기는 대신, 우리는 그것을 정보를 전달하고 결과를 받는 **함수처럼** 사용할 수 있다. 도구들로서의 멀티 에이전트에 대해 한번 노트를 정리해 보도록 하겠다.



## 1. 도구들로서의 멀티 에이전트 구현

```python
import streamlit as st
import asyncio
from agents import Agent, Runner, function_tool
from pydantic import BaseModel

class PRArticle(BaseModel):
    article_text: str
    commentary: str

adult_writer_agent = Agent(
    name="Adult Writer Agent",
    instructions="""Write the article based on the information given that it is suitable for adults interested in culture. 
                    Be mature.""", 
    model="gpt-4o",
)

teen_writer_agent = Agent(
    name="Teen Writer Agent",
    instructions="""Write the article based on the information given that it is suitable for teenagers who want to have a good time. 
                    Be cool!""", 
    model="gpt-4o",
)

kid_writer_agent = Agent(
    name="Kid Writer Agent",
    instructions="""Write the article based on the information given that it is suitable for kids of around 8 years old. 
                    Be enthusiastic!""", 
    model="gpt-4o",
)

format_agent = Agent(
    name="Format Agent",
    instructions=f"""Edit the article to add a title and subtitles and ensure the text is formatted as Markdown. Return only the text of article.""", 
    model="gpt-4o",
)

researcher_agent = Agent(
    name="Research agent",
    instructions="""You are a Travel Agent who will find useful information for your customers of all ages.
                    Find information on the destination(s) given. 
                    When you have a result send it to the appropriate writer agent to produce a short PR text.
                    When you have the result send it to the Format agent for final processing.
                    """,
    model="gpt-4o",
    tools = [kid_writer_agent.as_tool(
                tool_name="kids_article_writer",
                tool_description="Write an essay for kids",), 
            teen_writer_agent.as_tool(
                tool_name="teen_article_writer",
                tool_description="Write an essay for teens",), 
            adult_writer_agent.as_tool(
                tool_name="adult_article_writer",
                tool_description="Write an essay for adults",),
            format_agent.as_tool(
                tool_name="format_article",
                tool_description="Add titles and subtitles and format as Markdown",
        ),],
    output_type = PRArticle
)

async def run_agent(input_string):
    result = await Runner.run(researcher_agent, input_string)
    return result

# Streamlit UI

st.title("Travel Agent")
st.write("The travel agent will write about destinations for different audiences.")

destination = st.text_input("Enter a destination, select the age group and press 'Send':")
age_group = st.radio(
    "What age group is the reader?",
    ["Adult", "Teenager", "Child"],
    horizontal=True,
)

st.write("Response:")
response_container = st.container(height=500, border=True)

if st.button("Send"):
    response = asyncio.run(run_agent(f"The destination is {destination} and reader the age group is {age_group}"))
    with response_container:
        st.markdown(response.final_output.article_text)
    st.write(response)
    st.json(response.raw_responses)
```

* 도구 이름은 **에이전트 이름 뒤에 `.agent_as_tool()`**을 붙인 형태이며, 이 메서드는 해당 에이전트를 다른 도구들과 호환되도록 만들어 준다. 
* 이 도구는 **두 개의 매개변수**, 즉 **이름(name)**과 **설명(description)**을 필요로 한다. 