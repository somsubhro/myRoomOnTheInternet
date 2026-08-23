---
title: "Working with Langchain"
date: 2026-08-23
---

I started working with Langchain some time ago, and did not have the chance to share my diary. So here we go:

### Langchain install

I followed the install instructions [here](https://docs.langchain.com/oss/python/langchain/install). 

* I think the first `uv add langchain` command can be improved by adding `uv init` first. Otherwise you may run into:
  ```
  error: No `pyproject.toml` found in current directory or any parent directory` error
  ```

### What magic is this? 🪄

So I was following the instructions of working with a quick start agent [here](https://docs.langchain.com/oss/python/langchain/quickstart). The interesting code is: 
```
agent = create_agent(
    model="claude-sonnet-4-6",
    tools=[get_weather],
    system_prompt="You are a helpful assistant",
)
```

And when I run the file by `uv run python3 <filename>.py` I get a nice response like so: 
```
[{'type': 'text', 'text': "It looks like the weather in **San Francisco** is great — it's **sunny**! ☀️ Perfect weather to get outside and enjoy the city! 🌉"}]
```

But I did not read the API key from Anthropic anywhere in code. So what is happening? How is this response possible? Turns out, LangChain can read well known API key names (like `ANTHROPIC_API_KEY`) through its model integrations (which is essentially the command I ran separately `export ANTHROPIC_API_KEY="your-api-key"`). When I emptied out ANTHROPIC_API_KEY, it invariably failed the call as:
```
TypeError: Anthropic authentication failed: no API key or authorization credentials were provided. Set the ANTHROPIC_API_KEY environment variable, pass api_key=... to ChatAnthropic, or provide credentials via default_headers={"Authorization": ...}. If you are routing through the LangSmith gateway, set LANGSMITH_GATEWAY and LANGSMITH_GATEWAY_API_KEY.
```

Okay, faith in humanity is restored. Computing isn't magic after all.
