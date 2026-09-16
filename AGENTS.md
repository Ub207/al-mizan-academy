# AGENTS.md

Guidance for AI coding agents working in this repo. See also `CLAUDE.md` and `README.md`.

## What this repo is

Two deliverables deploy from one repo (git root = this directory, remote `Ub207/al-mizan-academy`, branch `main`):

- `app.py` + `quran_academy_kb.txt` → RAG chatbot (Streamlit) on Streamlit Cloud → `al-mizan-academy.streamlit.app`. Auto-redeploys on push to `main`.
- `index.html` → single self-contained marketing website on GitHub Pages → `ub207.github.io/al-mizan-academy/`. **Served from a separate `gh-pages` branch, deployed manually — pushes to `main` do NOT update the live site.**

No tests, linter, or build step.

## Commands

```bash
pip install -r requirements.txt
streamlit run app.py    # → http://localhost:8501
```

On Windows, if `pip`/`streamlit` aren't on PATH use the `py` launcher: `py -m pip install ...` / `py -m streamlit run app.py`. Local secrets: `GROQ_API_KEY=...` in `.env` (loaded via `load_dotenv()`); get a key at console.groq.com.

## The KB format is load-bearing

`load_knowledge_base()` in `app.py` chunks `quran_academy_kb.txt` by splitting on lines that start with `Q:` (one chunk per Q&A pair) and keeping only chunks containing `Q:` or `A:`. If you edit the KB, **preserve the `Q:` / `A:` line structure** — without it the code silently falls back to blind 500-char splitting and retrieval quality drops. Section headers like `COURSES OFFERED` and `----` are fine; they fold into the adjacent chunk.

Haunts that will bite:
- KB + embeddings are cached with `@st.cache_resource`. After editing the KB, restart the app or clear the cache (⋮ → Clear cache) or changes won't appear.
- New/renamed course → update it in **both** `quran_academy_kb.txt` **and** the course catalogue in the system prompt inside `app.py` (the catalogue is baked into the prompt so the bot recognizes every course by name even when retrieval misses).
- The chatbot is prompted to answer only from retrieved KB context + catalogue and to redirect Fatwa/ruling questions to qualified scholars. Preserve that guardrail when editing the prompt.
- Model/config in `get_groq_response`: `openai/gpt-oss-120b`, temperature 0.3, max 800 tokens, last 6 turns of history. Comment on line 276 notes Groq retired `llama-3.3-70b-versatile` (404) — read it before touching the model.

## Website ↔ chatbot coupling

`index.html` embeds the chatbot in an `<iframe>` pointing at the Streamlit URL. The URL lives in one config object at the top of the page's `<script>` — `SITE.chatbotUrl` — plus the iframe `src` in the `#chatbot` section and two "open in new tab" links. WhatsApp number (`SITE.whatsappNumber`) and pricing (`SITE.pricing`) are config-only too. The assistant answers from the KB, not the website — **if you change the website, mirror the change in the KB** or the bot won't know about it.

## Secrets

`get_groq_response` resolves the key as `os.environ["GROQ_API_KEY"]` first (so `.env` wins), then `st.secrets["GROQ_API_KEY"]`. That secret must sit at the **top level** of `.streamlit/secrets.toml` — NOT under a `[secrets]` header. The key is gitignored.