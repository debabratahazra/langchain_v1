---
description: "Scaffold a new LangChain v1 experiment cell that follows this project's env, proxy, and model-guard conventions"
name: "LangChain Experiment"
argument-hint: "What should the experiment do? (e.g. 'few-shot prompt with Groq', 'streaming chat with the LiteLLM proxy')"
agent: "agent"
---

Create a new LangChain v1 experiment for the task described in the prompt input. Add it to [src/01_langchain.ipynb](../../src/01_langchain.ipynb) as a new code cell (or a small notebook file under `src/` if the user asks for a standalone script).

Follow this project's established conventions:

## Environment & configuration

- Assume `os` and the environment are already loaded by the notebook's first cells via `load_dotenv` with `override=True`. Do NOT re-add the dotenv bootstrap unless creating a standalone file.
- Read provider keys from the environment: `OPENAI_API_KEY`, `GROQ_API_KEY`, `GOOGLE_API_KEY`. Never hardcode secrets.
- For the OpenAI/LiteLLM path, use the company proxy and SSL bundle:
  - `BASE_URL = "https://litellm.icp.infineon.com"` with the `/v1` suffix
  - `httpx.Client(verify=SSL_CERT)` where `SSL_CERT` points to the local CA bundle
  - Read the model from `os.environ.get("OPENAI_MODEL", ...)`

## Model guard (required for OpenAI/LiteLLM)

- List available models and filter out the `"no-default-models"` placeholder.
- If the requested model is unavailable, fall back to the first valid model.
- If no valid model exists, print clear guidance to set `OPENAI_MODEL` and skip the API call instead of erroring.

## Provider selection

- Prefer the appropriate LangChain integration already in `pyproject.toml`:
  - `langchain-openai` for the LiteLLM proxy
  - `langchain-groq` for Groq
  - `langchain-google-genai` for Google
- Use LangChain v1 message/runnable APIs (e.g. `ChatOpenAI`, `ChatGroq`, `ChatGoogleGenerativeAI`, `.invoke`, prompt templates) rather than raw provider SDK calls, unless the task specifically needs the raw client.

## Output

- Keep the cell self-contained and runnable in isolation given the bootstrap cells.
- Print results clearly and keep test prompts short.
- Add a one-line comment header describing what the cell demonstrates.

Ask which provider to target only if the task is ambiguous; otherwise infer the most fitting one from the prompt input.
