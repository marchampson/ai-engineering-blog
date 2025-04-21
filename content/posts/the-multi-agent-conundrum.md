---
title: "The Multi-Agent Conundrum"
date: 2025-04-21
draft: false
tags: ["AI", "LLM", "CrewAI", "Multi-Agent Systems", "Streamlit"]
categories: ["AI Engineering"]
---

# The Multi-Agent Conundrum

Multi-agent systems are gaining traction as a way to improve AI task performance by dividing responsibilities between specialized roles. In this tutorial, we’ll build a web-based assistant using [CrewAI](https://crewai.com) and [OpenAI](https://openai.com), wrapped in a [Streamlit](https://streamlit.io/) interface. We'll compare single-agent vs multi-agent outputs, measure token usage and cost. Finally, we'll generate markdown reports for export. If you're experimenting with LLM-powered workflows, hopefully you'll gain some insights from this post. 

---

## Setup

### 1. Install Python 3.11

Download from [python.org](https://www.python.org/downloads/).

### 2. Create a virtual environment

```bash
python3.11 -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows
```

### 3. Install dependencies

```bash
pip install streamlit openai crewai tiktoken python-dotenv
```

### 4. Add `.env`

```env
OPENAI_API_KEY=your_openai_api_key_here
```

---

## Multi-Agent Assistant (Tab 1)

### App boilerplate

```python
import os
import streamlit as st
from crewai import Agent, Task
from dotenv import load_dotenv

load_dotenv()
os.environ["OPENAI_API_KEY"] = os.getenv("OPENAI_API_KEY")

st.set_page_config(page_title="AI Team Assistant", page_icon="🧠", layout="wide")
st.title("🧠 AI Team Assistant")
```

### Define agents

```python
researcher = Agent(
    role="Researcher",
    goal="Find concise and relevant facts.",
    backstory="You gather insights from trusted sources.",
    verbose=True
)

writer = Agent(
    role="Writer",
    goal="Summarize clearly.",
    backstory="You simplify technical insights.",
    verbose=True
)

reviewer = Agent(
    role="Reviewer",
    goal="Ensure clarity and tone.",
    backstory="You polish and refine content.",
    verbose=True
)
```

### Workflow

```python
task1 = Task(
    description="Research the topic and provide insights.",
    agent=researcher
)
output1 = researcher.execute_task(task1)
st.markdown(f"### Researcher Output\n{output1}")

task2 = Task(
    description="Summarize the findings.",
    agent=writer
)
output2 = writer.execute_task(task2)
st.markdown(f"### Writer Summary\n{output2}")

task3 = Task(
    description="Review the summary.",
    agent=reviewer
)
final_output = reviewer.execute_task(task3)
st.markdown(f"### Final Reviewed Output\n{final_output}")
```

---

## Benchmark Comparison (Tab 2)

### Single agent output

```python
from openai import OpenAI
client = OpenAI()

response = client.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "Your prompt here."}]
)
single_output = response.choices[0].message.content.strip()
st.markdown(f"### Single-Agent Output\n{single_output}")
```

---

## Comparison and Analysis

### GPT-4-assisted analysis

```python
def analyze_outputs(single_output, multi_output):
    comparison_prompt = f"""
    Compare the two responses:

    Single Agent: {single_output}
    Multi-Agent: {multi_output}

    Provide a comparison in Markdown.
    """
    analysis = client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": comparison_prompt}]
    )
    return analysis.choices[0].message.content.strip()

analysis_result = analyze_outputs(single_output, final_output)
st.markdown(analysis_result)
```

---

## Export Markdown Report

```python
import hashlib
import json

def export_results(content):
    filename = f"benchmark_{hashlib.md5(content.encode()).hexdigest()[:6]}.md"
    with open(filename, "w") as f:
        f.write(content)
    return filename

export_filename = export_results(analysis_result)
st.download_button("Download Analysis", analysis_result, file_name=export_filename)
```

---

## Summary

Using CrewAI with a multi-agent setup allows roles to specialize and hand off tasks. It adds structure, clarity, and depth compared to single-prompt responses — and it's measurable.

---

## ⚠ Disclaimer

While this project demonstrates how multi-agent systems can enhance LLM outputs, it’s important to remember that large language models are not perfect. They can still produce incorrect or misleading information (known as "hallucinations"). Use these results as illustrative examples, not authoritative sources. Always verify critical information from reliable references.


