---
name: fix-proxy-connection
description: "Diagnose and fix APIConnectionError / WinError 10061 (connection refused) when LangChain or OpenAI calls fail in this project. Use when a chat model, init_chat_model, ChatOpenAI, or create_agent call cannot reach the LiteLLM proxy, or when a model defaults to the public OpenAI endpoint instead of https://litellm.icp.infineon.com. Rewires the model to use the proxy base_url, SSL bundle, and API key."
argument-hint: 'Which cell/model is failing? (e.g. "the ChatOpenAI cell", "create_agent invoke")'
---

# Fix LiteLLM Proxy Connection Errors

In this project all OpenAI-compatible traffic must go through the company **LiteLLM proxy** (`https://litellm.icp.infineon.com/v1`) using a custom SSL bundle. The most common failure is a LangChain model built from a bare model **name/string**, which silently defaults to the public OpenAI endpoint and fails with a connection error.

## When to Use

- `APIConnectionError: Connection error.`
- `ConnectError: [WinError 10061] No connection could be made because the target machine actively refused it`
- A `ChatOpenAI`, `init_chat_model`, or `create_agent` call fails while the raw `OpenAI` client cell in the same notebook works.

## Root Cause Check

A model is misconfigured if it was created **without** the proxy wiring, e.g.:

- `init_chat_model(MODEL)` — no `base_url`/`http_client`
- `ChatOpenAI(MODEL)` or `ChatOpenAI(model=MODEL)` — no `base_url`/`http_client`
- `create_agent(model=MODEL, ...)` — passing the **string** instead of a configured model object

To confirm, inspect the model repr: a correct one shows `openai_api_base='https://litellm.icp.infineon.com/v1'`. If it shows the default OpenAI base, it's wrong.

## Procedure

1. Read the failing notebook with the notebook summary and the cell's error output to confirm it is `APIConnectionError` / `WinError 10061`.
2. Verify the proxy setup cell ran successfully and defined `BASE_URL`, `SSL_CERT`, `http_client`, and a non-empty `MODEL` (the `no-default-models` guard must have selected a real model).
3. Rewire the model using the matching pattern below.
4. Re-run the model creation cell, then the invoke/agent cell. Success = a returned `AIMessage` / agent result, not an error.
5. Confirm the model repr shows the proxy `openai_api_base`.

## Fix Patterns

`init_chat_model`:

```python
model = init_chat_model(
    MODEL,
    model_provider="openai",
    base_url=f"{BASE_URL}/v1",
    api_key=os.environ.get("OPENAI_API_KEY"),
    http_client=http_client,
)
```

`ChatOpenAI`:

```python
model = ChatOpenAI(
    model=MODEL,
    base_url=f"{BASE_URL}/v1",
    api_key=os.environ.get("OPENAI_API_KEY"),
    http_client=http_client,
)
```

`create_agent` — build a proxy-bound model first, then pass the **object**:

```python
llm = ChatOpenAI(
    model=MODEL,
    base_url=f"{BASE_URL}/v1",
    api_key=os.environ.get("OPENAI_API_KEY"),
    http_client=http_client,
)
agent = create_agent(model=llm, tools=[...], system_prompt="...")
```

## Notes

- Google (`google_genai:...`) and Groq models call their own provider APIs directly and do **not** use the LiteLLM proxy — do not add the OpenAI `base_url` to them.
- Never hardcode or print full API keys; always read from the environment.
- See the project conventions in [langchain.instructions.md](../../instructions/langchain.instructions.md).
