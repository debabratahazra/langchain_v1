---
description: "Use when writing or modifying Python code or Jupyter notebooks in this LangChain v1 project. Covers env/secrets loading, the LiteLLM proxy + SSL setup, the model-availability guard, provider integrations, and uv tooling."
name: "LangChain v1 Project Conventions"
applyTo: ["**/*.py", "**/*.ipynb"]
---

# LangChain v1 Project Conventions

This is a LangChain **v1** learning project (Python `>=3.14`, managed with **uv**) that calls LLMs through a company **LiteLLM proxy** plus Groq and Google. Follow these conventions for all code.

## Environment & secrets

- Load config from `.env` using `python-dotenv`. In notebooks, the bootstrap cell already calls `load_dotenv(..., override=True)` after `find_dotenv(usecwd=True)` — do NOT duplicate it; assume `os` and env vars are available in later cells.
- Read provider keys from the environment only: `OPENAI_API_KEY`, `GROQ_API_KEY`, `GOOGLE_API_KEY`. **Never hardcode secrets or print full key values** in committed code.
- Read the model name from `os.environ.get("OPENAI_MODEL", ...)`.

## LiteLLM / OpenAI proxy

When targeting the OpenAI-compatible path, always route through the proxy with the local SSL bundle:

```python
BASE_URL = "https://litellm.icp.infineon.com"   # append "/v1" for the API base
http_client = httpx.Client(verify=SSL_CERT)      # SSL_CERT = local CA bundle path
```

## Model-availability guard (required for the proxy)

The proxy may return only the placeholder `"no-default-models"`. Before any chat call:

- List models, filter out `"no-default-models"`.
- If the requested model is missing, fall back to the first valid model.
- If none are valid, print guidance to set `OPENAI_MODEL` and skip the call — do not raise.

## Provider integrations

- Prefer the LangChain integrations already in `pyproject.toml`: `langchain-openai` (proxy), `langchain-groq` (Groq), `langchain-google-genai` (Google).
- Use LangChain v1 APIs (`ChatOpenAI`, `ChatGroq`, `ChatGoogleGenerativeAI`, prompt templates, `.invoke`, runnables) over raw provider SDKs unless raw access is specifically required.

## Tooling

- Manage dependencies with **uv** (e.g. `uv add <pkg>`, `uv run`); do not edit `requirements.txt` by hand as the source of truth.
- Keep notebook cells self-contained and runnable given the bootstrap cells; keep test prompts short.
