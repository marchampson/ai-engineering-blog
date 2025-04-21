---
title: "The Multi-Agent Conundrum"
date: 2025-04-21
draft: false
tags: ["AI", "LLM", "CrewAI", "Multi-Agent Systems", "Streamlit"]
categories: ["AI Engineering"]
---

# The Multi-Agent Conundrum

Multi-agent systems are growing in popularity as a way to improve AI task performance by dividing responsibilities between specialised roles. In this tutorial, you'll build a web-based assistant using [CrewAI](https://docs.crewai.com/) and OpenAI, wrapped in a Streamlit interface. You'll compare single-agent vs multi-agent outputs, measure token usage and cost, and generate markdown reports for easy export. This post is designed to be practical and modular—ideal for anyone experimenting with LLM-powered workflows.

---

## Full Script

Below is the complete working Streamlit app. You can copy and paste this into a file (e.g. `app.py`) and run it directly using: streamlit run app.py

> **Note:** Don’t forget to create a `.env` file in the same directory as your script and add your OpenAI API key, example setup below.

Example:

OPENAI_API_KEY=sk-1234567890abcdefexample

```bash
streamlit run app.py
```



```python
import os
import streamlit as st
import openai
from openai import OpenAI
from crewai import Agent, Task
from dotenv import load_dotenv
import tiktoken
import hashlib
import json

# Must be first Streamlit command
st.set_page_config(page_title="AI Team Assistant", page_icon="🧠", layout="wide")

# Load environment variables
load_dotenv()
openai.api_key = os.getenv("OPENAI_API_KEY")
client = OpenAI()

# Token estimation helper
def count_tokens(text, model="gpt-4"):
    try:
        encoding = tiktoken.encoding_for_model(model)
    except KeyError:
        encoding = tiktoken.get_encoding("cl100k_base")
    return len(encoding.encode(text))

# ChatCompletion wrapper for single-shot LLM
@st.cache_data(show_spinner=False)
def run_single_agent(prompt, model="gpt-4"):
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}]
    )
    return response.choices[0].message.content.strip(), response.usage.total_tokens

# Analysis function using OpenAI
@st.cache_data(show_spinner=False)
def analyze_outputs(single_output, multi_output, single_tokens, multi_tokens, single_cost, multi_cost):
    summary_prompt = f"""
You are a technical content reviewer. Below are two answers to the same prompt. One is written by a single GPT-4 agent. The other was created by multiple specialized agents.

Compare the two responses and output two tables:

1. A quick side-by-side comparison showing: Topics Hit, Structure, Insight Depth, Tweet Quality, Token Count, and Cost.

2. A table of key stakeholder questions and what this result demonstrates.

Then include a short paragraph explaining which version is stronger and why.

Use Markdown formatting for the tables.

---

### Single Agent Response:
{single_output}

**Tokens:** {single_tokens}
**Cost:** ${single_cost:.4f}

---

### Multi-Agent Response:
{multi_output}

**Tokens:** {multi_tokens}
**Cost:** ${multi_cost:.4f}
"""
    response = client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": summary_prompt}]
    )
    return response.choices[0].message.content.strip()

# Export function
@st.cache_data(show_spinner=False)
def export_results(data: dict):
    filename = f"benchmark_result_{hashlib.md5(json.dumps(data).encode()).hexdigest()[:6]}.md"
    with open(filename, "w") as f:
        f.write(data["summary"])
    return filename

# Sidebar settings
st.sidebar.header("⚙️ Settings")
model = st.sidebar.selectbox("Model", ["gpt-4", "gpt-3.5-turbo"], index=0)
cost_per_1k = 0.03 if model == "gpt-4" else 0.001

use_writer = st.sidebar.checkbox("✍️ Include Writer", value=True)
use_reviewer = st.sidebar.checkbox("🔍 Include Reviewer", value=True)
use_fact_checker = st.sidebar.checkbox("✅ Include Fact Checker", value=False)

preset_prompts = {
    "Marketing Strategy": "Research three cutting-edge strategies for marketing and lead generation in 2024, summarize them for a CMO, and then write a tweet-sized version for social media.",
    "Technical Report": "Analyze the key components of a scalable microservices architecture and summarize the benefits and trade-offs.",
    "Product Research": "Compare three leading electric vehicle models released in 2023 in terms of range, price, and features."
}

st.markdown("<h1 style='text-align: center;'>🧠 AI Team Assistant</h1>", unsafe_allow_html=True)

tab1, tab2 = st.tabs(["🧠 Multi-Agent Demo", "🧪 Benchmark Comparison"])

# --- TAB 1: Multi-Agent Flow ---
with tab1:
    st.markdown("### 🔹 Choose a predefined prompt or write your own:")
    selected_preset = st.radio("Presets", list(preset_prompts.keys()), horizontal=True, key="preset_radio")
    prompt_input = st.text_area("Prompt", value=preset_prompts[selected_preset], height=120, key="main_prompt")

    if st.button("🚀 Run AI Crew") and prompt_input:
        topic = prompt_input.strip()
        with st.spinner("Agents are working..."):

            transcript = {}
            active_roles = ["Researcher"]
            if use_writer: active_roles.append("Writer")
            if use_reviewer: active_roles.append("Reviewer")
            if use_fact_checker: active_roles.append("Fact Checker")

            agent_map_container = st.empty()
            transcript_container = st.container()
            result_container = st.container()

            def update_agent_map(current):
                map_display = []
                for role in active_roles:
                    if role == current:
                        map_display.append(f"[**{role}**]")
                    else:
                        map_display.append(f"[{role}]")
                agent_map_container.markdown(f"**Workflow:** {' ➞ '.join(map_display)}")

            def display_transcript(role, output):
                with transcript_container:
                    st.markdown(f"**📣 {role} said:**")
                    st.markdown(
                        f"<div style='background-color:#eef6ff;padding:12px;border-radius:6px;margin-bottom:20px;border:1px solid #cce;'>{output}</div>",
                        unsafe_allow_html=True
                    )

            researcher = Agent(
                role="Researcher",
                goal=f"Find useful information about {topic}.",
                backstory="You gather concise and relevant facts from trusted sources.",
                verbose=True
            )
            update_agent_map("Researcher")
            task1 = Task(
                description=f"Research the topic: {topic} and provide 3–5 key insights.",
                expected_output="A concise list of insights or takeaways.",
                agent=researcher
            )
            output1 = researcher.execute_task(task1)
            transcript["Researcher"] = output1
            display_transcript("Researcher", output1)

            if use_writer:
                writer = Agent(
                    role="Writer",
                    goal=f"Write a clear explanation about {topic}.",
                    backstory="You turn technical insights into readable summaries.",
                    verbose=True
                )
                update_agent_map("Writer")
                task2 = Task(
                    description=f"Write a 2-paragraph summary of the research findings about: {topic}.",
                    expected_output="A clear and easy-to-read explanation.",
                    agent=writer
                )
                output2 = writer.execute_task(task2)
                transcript["Writer"] = output2
                display_transcript("Writer", output2)

            if use_reviewer:
                reviewer = Agent(
                    role="Reviewer",
                    goal=f"Review the explanation about {topic} for tone and clarity.",
                    backstory="You polish and verify the quality of written content.",
                    verbose=True
                )
                update_agent_map("Reviewer")
                task3 = Task(
                    description="Review the summary for clarity, accuracy, and tone.",
                    expected_output="A final polished summary ready to share.",
                    agent=reviewer
                )
                output3 = reviewer.execute_task(task3)
                transcript["Reviewer"] = output3
                display_transcript("Reviewer", output3)

            if use_fact_checker:
                fact_checker = Agent(
                    role="Fact Checker",
                    goal=f"Verify the factual accuracy of statements made about {topic}.",
                    backstory="You identify and flag any misleading or incorrect information.",
                    verbose=True
                )
                update_agent_map("Fact Checker")
                task4 = Task(
                    description="Fact-check the final summary for accuracy and cite reliable sources.",
                    expected_output="A list of potential inaccuracies and confirmations, with sources.",
                    agent=fact_checker
                )
                output4 = fact_checker.execute_task(task4)
                transcript["Fact Checker"] = output4
                display_transcript("Fact Checker", output4)

            final_result = transcript.get("Fact Checker") or transcript.get("Reviewer") or transcript.get("Writer") or transcript.get("Researcher")

            with result_container:
                st.markdown("---")
                st.subheader("📄 Final Summary")
                st.markdown(
                    f"<div style='background-color:#f9f9f9;padding:15px;border-radius:8px;border:1px solid #ddd'>{final_result}</div>",
                    unsafe_allow_html=True
                )

                combined_output = "\n\n".join(transcript.values())
                tokens = count_tokens(combined_output, model)
                estimated_cost = (tokens / 1000) * cost_per_1k

                st.subheader("💰 Token Usage Estimate")
                st.write(f"**Model used**: `{model}`")
                st.write(f"**Estimated tokens**: `{tokens}`")
                st.write(f"**Estimated cost**: `${estimated_cost:.4f}`")

# --- TAB 2: Benchmark ---
with tab2:
    st.markdown("### 📊 Side-by-Side Benchmark: Single vs Multi-Agent")
    selected_bench = st.radio("Choose a scenario", list(preset_prompts.keys()), horizontal=True, key="bench_radio")
    bench_prompt = preset_prompts[selected_bench]
    st.write("**Prompt:**")
    st.info(bench_prompt)

    if st.button("🧑‍⚖️ Run Benchmark"):
        with st.spinner("Running both versions..."):
            single_output, single_tokens = run_single_agent(bench_prompt, model)

            if selected_bench == "Marketing Strategy":
                research_desc = "Find three cutting-edge strategies for marketing and lead generation in 2024."
                writing_desc = "Summarize the strategies in 2 paragraphs for a CMO, then write a tweet for each one."
            elif selected_bench == "Technical Report":
                research_desc = "Find key components of a scalable microservices architecture and trade-offs."
                writing_desc = "Summarize these findings in a clear executive summary."
            elif selected_bench == "Product Research":
                research_desc = "Find and compare three leading electric vehicle models from 2023 in range, price, and features."
                writing_desc = "Summarize the comparison in 2 paragraphs and conclude with a recommendation."
            else:
                research_desc = bench_prompt
                writing_desc = "Summarize the above into 2 clear paragraphs."

            researcher = Agent(
                role="Researcher",
                goal="Gather information clearly and efficiently",
                backstory="You search and synthesize trusted insights from reliable sources.",
                verbose=True
            )
            task1 = Task(
                description=research_desc,
                expected_output="Concise bullet points of key findings",
                agent=researcher
            )
            out1 = researcher.execute_task(task1)

            writer = Agent(
                role="Writer",
                goal="Summarize research effectively for the target audience",
                backstory="You write clear summaries tailored to business and professional readers.",
                verbose=True
            )
            task2 = Task(
                description=writing_desc,
                expected_output="A structured and engaging explanation",
                agent=writer
            )
            out2 = writer.execute_task(task2)

            multi_output = out2
            total_multi_tokens = count_tokens(out1 + out2, model)
            total_multi_cost = (total_multi_tokens / 1000) * cost_per_1k
            single_cost = single_tokens / 1000 * cost_per_1k

            col1, col2 = st.columns(2)
            with col1:
                st.subheader("🤖 Single-Agent Result")
                st.markdown(single_output)
                st.markdown(f"**Tokens:** {single_tokens}")
                st.markdown(f"**Cost:** ${single_cost:.4f}")
            with col2:
                st.subheader("🧠 Multi-Agent Result")
                st.markdown(multi_output)
                st.markdown(f"**Tokens:** {total_multi_tokens}")
                st.markdown(f"**Cost:** ${total_multi_cost:.4f}")

            with st.spinner("Generating expert analysis..."):
                summary = analyze_outputs(single_output, multi_output, single_tokens, total_multi_tokens, single_cost, total_multi_cost)

            st.markdown("---")
            st.subheader("🧾 Benchmark Analysis")
            st.markdown(summary)

            st.download_button("💾 Export Analysis as Markdown", summary, file_name="benchmark_analysis.md", mime="text/markdown")


```

---

## Breaking It Down

### Imports and Setup

This section loads environment variables, configures Streamlit, and prepares the OpenAI and CrewAI libraries.

```python
import os
import streamlit as st
import openai
from openai import OpenAI
from crewai import Agent, Task
from dotenv import load_dotenv
import tiktoken
import hashlib
import json

st.set_page_config(page_title="AI Team Assistant", page_icon="🧠", layout="wide")
load_dotenv()
openai.api_key = os.getenv("OPENAI_API_KEY")
client = OpenAI()
```

### Utility Functions

Reusable functions for estimating token usage, running a single-agent prompt, analysing outputs, and exporting results.

```python
def count_tokens(text, model="gpt-4"):
    try:
        encoding = tiktoken.encoding_for_model(model)
    except KeyError:
        encoding = tiktoken.get_encoding("cl100k_base")
    return len(encoding.encode(text))

@st.cache_data(show_spinner=False)
def run_single_agent(prompt, model="gpt-4"):
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}]
    )
    return response.choices[0].message.content.strip(), response.usage.total_tokens

@st.cache_data(show_spinner=False)
def analyze_outputs(single_output, multi_output, single_tokens, multi_tokens, single_cost, multi_cost):
    summary_prompt = f"""
You are a technical content reviewer... [PROMPT TRUNCATED]
"""
    response = client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": summary_prompt}]
    )
    return response.choices[0].message.content.strip()

@st.cache_data(show_spinner=False)
def export_results(data: dict):
    filename = f"benchmark_result_{hashlib.md5(json.dumps(data).encode()).hexdigest()[:6]}.md"
    with open(filename, "w") as f:
        f.write(data["summary"])
    return filename
```

### Sidebar UI and Prompt Options

Allows the user to select the model, toggle agent roles, and pick a preset or custom prompt.

```python
st.sidebar.header("Settings")
model = st.sidebar.selectbox("Model", ["gpt-4", "gpt-3.5-turbo"], index=0)
cost_per_1k = 0.03 if model == "gpt-4" else 0.001

use_writer = st.sidebar.checkbox("Include Writer", value=True)
use_reviewer = st.sidebar.checkbox("Include Reviewer", value=True)
use_fact_checker = st.sidebar.checkbox("Include Fact Checker", value=False)

preset_prompts = {
    "Marketing Strategy": "Research...",
    "Technical Report": "Analyse...",
    "Product Research": "Compare..."
}
```

### Streamlit Layout and Tabs

Splits the interface into two tabs: one for the multi-agent demo and one for benchmarking.

```python
tab1, tab2 = st.tabs(["Multi-Agent Demo", "Benchmark Comparison"])
```

### Multi-Agent Execution Flow (Tab 1)

Creates agents dynamically and runs them in sequence. Outputs are displayed and styled.

- Displays a workflow diagram
- Stores and shows agent outputs
- Calculates and displays token cost estimate

Each agent is configured with a role, goal, and task. The results are aggregated.

### Benchmarking (Tab 2)

Runs a side-by-side comparison:

- One single-agent run
- One multi-agent run (Researcher + Writer)
- GPT-4 then compares both outputs with structured tables
- Token and cost metrics are shown
- Markdown export is available

This lets users test and measure the impact of specialised collaboration vs a single prompt.

---

## Summary

Using CrewAI with a multi-agent setup allows roles to specialise and hand off tasks. This structure often improves clarity and depth to the results — and with built-in tools for token usage, cost tracking, and AI-powered analysis, you can measure the impact directly.

## Disclaimer

While this project demonstrates how multi-agent systems can enhance LLM outputs, it’s important to remember that large language models are not perfect. They can still produce incorrect or misleading information (known as "hallucinations"). Use these results as illustrative examples, not authoritative sources. Always verify critical information from reliable references.


