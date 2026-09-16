# 🕌 Mizan Academy

> Marketing website **+** RAG-powered AI assistant for **Mizan Academy** (also known as Al-Mizan Online Quran Academy) — one-to-one online Quran classes for kids and adults, taught by **Ubaid ur Rahman** (Hafiz-e-Quran, Fazil Graduate, Saba Qira'at specialist).

![Streamlit](https://img.shields.io/badge/Streamlit-app-FF4B4B?logo=streamlit&logoColor=white)
![Groq](https://img.shields.io/badge/Groq-GPT--OSS%20120B-F55036)
![FAISS](https://img.shields.io/badge/FAISS-vector%20search-0467DF)
![RAG](https://img.shields.io/badge/RAG-retrieval%20augmented-6C4AB6)

**🌐 Live website:** https://ub207.github.io/al-mizan-academy/
**🤖 Live AI assistant:** https://al-mizan-academy.streamlit.app

---

## What's in this repo

One repository ships **two** deliverables that deploy to two different places:

| Component | File(s) | Deploys to |
|-----------|---------|-----------|
| Academy website (single self-contained page) | `index.html` | **GitHub Pages** → `ub207.github.io/al-mizan-academy/` |
| RAG AI chatbot (Streamlit) | `app.py` + `quran_academy_kb.txt` | **Streamlit Cloud** → `al-mizan-academy.streamlit.app` |

The website embeds the chatbot in an `<iframe>`, so visitors can chat with the assistant right on the site.

## ✨ Features

- **24/7 AI assistant** for course, pricing, scheduling, and enrollment questions
- **Grounded answers (RAG)** — replies come from a curated knowledge base, so course details aren't hallucinated
- **Knows the full catalogue** — courses across Quran, Tajweed, Translation, Islamic Studies, and Arabic, plus the *Saba Qira'at* and *Ijazah* specializations
- **Typo / transliteration tolerant** — understands variations like `gardan` → `Gardaan`
- **Warm, culturally-aware tone** with natural Islamic greetings
- **Stays in its lane** — politely redirects religious rulings (Fatwa) to qualified scholars
- **Free-trial guidance** — nudges prospective students toward booking a free class
- **Responsive, accessible, SEO-ready website** with the chatbot embedded inline

## 🧠 How the AI assistant works (RAG)

```
quran_academy_kb.txt
  → split into Q&A chunks
  → embed each chunk (all-MiniLM-L6-v2, sentence-transformers)
  → store in a FAISS IndexFlatL2 vector index (in-memory, no external DB)

user question
  → embed → retrieve the top-6 most relevant chunks
  → inject [retrieved context] + [authoritative course catalogue] + [last 6 turns]
  → Groq chat completion (openai/gpt-oss-120b)
  → grounded answer
```

The full course catalogue also lives directly in the system prompt, so the bot **always** knows every course by name — even if a specific question doesn't retrieve the right chunk.

## 🛠️ Tech stack

- **Python 3.9+**
- **Streamlit** — chat UI
- **Groq API** — `openai/gpt-oss-120b` (GPT-OSS 120B)
- **sentence-transformers** — `all-MiniLM-L6-v2` embeddings
- **FAISS** (`faiss-cpu`) — vector similarity search
- **langchain-text-splitters** — fallback chunking
- **python-dotenv** — local secrets

## 📁 Project structure

```
al-mizan-academy/
├── index.html                  <- Academy website (single self-contained page)
├── assets/                     <- Teacher portraits (WebP + JPEG, 3 sizes) + OG cover
├── app.py                      <- RAG AI chatbot (Streamlit + Groq)
├── quran_academy_kb.txt        <- Chatbot knowledge base (Q&A format)
├── robots.txt                  <- Crawl rules + sitemap reference
├── sitemap.xml                 <- Site sitemap
├── requirements.txt            <- Python dependencies
├── .streamlit/
│   └── secrets.toml.example    <- API-key template for Streamlit Cloud
├── CLAUDE.md                   <- Repo guide for AI coding assistants
└── README.md
```

## 🚀 Run locally

```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Add your Groq API key (free at https://console.groq.com)
echo 'GROQ_API_KEY=your_key_here' > .env

# 3. Launch — opens http://localhost:8501
streamlit run app.py
```

> **Windows note:** if `streamlit` / `pip` aren't on your PATH, use the `py` launcher:
> `py -m pip install -r requirements.txt` and `py -m streamlit run app.py`.

## ☁️ Deployment

### Chatbot → Streamlit Community Cloud (free)

1. Push this repo to GitHub.
2. Go to [share.streamlit.io](https://share.streamlit.io) and sign in with GitHub.
3. **New app** → select this repo → set **Main file path** to `app.py`.
4. **Advanced settings → Secrets**, paste this at the **top level** (not under a `[section]` header):
   ```toml
   GROQ_API_KEY = "your_key_here"
   ```
5. **Deploy.** Every push to `main` auto-redeploys the app.

### Website → GitHub Pages (free)

1. Repo **Settings → Pages**.
2. **Source:** *Deploy from a branch* → `gh-pages` → `/ (root)`. Note this is **not** `main` — the live site is served from a separate `gh-pages` branch that holds its own copy of `index.html`, `app.py`, and `quran_academy_kb.txt`. A push to `main` does **not** update the live site; the deploy is manual.
3. Your site publishes at `https://<username>.github.io/al-mizan-academy/`.

### Connect the two

The website embeds the chatbot via an `<iframe>`. The Streamlit URL lives in one config object at the top of `index.html`'s `<script>` — `SITE.chatbotUrl` — plus the iframe `src` in the `#chatbot` section and two "open in new tab" links. Update them together if the deployment URL changes. `SITE.whatsappNumber`, `SITE.pricing`, and `SITE.analytics` are config-only too.

## 📚 Updating the knowledge base (important)

The assistant answers from **`quran_academy_kb.txt`**, *not* from the website. **If you change the website, mirror the change in the KB** — otherwise the bot won't know about it.

- **Keep the `Q:` / `A:` format.** The loader chunks the KB by splitting on lines that start with `Q:` (one chunk per Q&A pair). Breaking that structure silently degrades retrieval quality.
- **Refresh the cache after editing.** The KB and its embeddings are cached (`@st.cache_resource`), so changes won't appear until you **⋮ → Clear cache** (or restart the app).
- **New/renamed course?** Update it in **both** `quran_academy_kb.txt` **and** the course catalogue in the system prompt inside `app.py`, so the bot always recognizes it.
- **Pricing is quote-based.** Plans are 2, 3, or 5 classes per week; families receive a personalised quote. Do not state fixed prices on the site or in the KB — ask the owner for current rates before adding them.

## 📞 Contact

- 🌐 **Website:** https://ub207.github.io/al-mizan-academy/
- ✉️ **Email:** usmanubaidurrehman@gmail.com
- 💬 **Free trial class** available via WhatsApp

---

© 2026 Mizan Academy (Al-Mizan Online Quran Academy) — Ubaid ur Rahman. Built with ❤️ using Streamlit & Groq AI.