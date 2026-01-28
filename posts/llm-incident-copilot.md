# Building an AI-Powered Log Analyzer: LLM Incident Copilot

*How I built a RAG-based tool that helps engineers debug production incidents faster*

---

## The Problem

It's 2 AM. PagerDuty goes off. A critical service is down.

You SSH into the server, tail the logs, and see thousands of lines scrolling past. Somewhere in there is the root cause. But where?

**This is the reality of incident debugging.** Despite advances in observability tooling, much of incident response still relies on:

- Manual log inspection
- Pattern recognition built over years
- Tribal knowledge that lives in senior engineers' heads
- Caffeine and adrenaline

I kept asking myself: *Can AI actually help here  not with a chatbot demo, but with real production logs?*

So I built **LLM Incident Copilot** to find out.

---

## What I Built

LLM Incident Copilot is a local, reproducible system that:

1. **Ingests production logs** (JSON, Logfmt, Syslog, Java/Hadoop formats)
2. **Chunks them intelligently** (preserving error context)
3. **Embeds them into a vector database** (Qdrant)
4. **Answers questions in natural language** with evidence citations

The key innovation: **Every AI conclusion references specific log lines.** No hallucinations. No vague answers.

---

## Architecture

| Component | Technology | Why |
|-----------|------------|-----|
| **Frontend** | React 18 + Vite | Fast dev experience |
| **Backend** | FastAPI (Python 3.11) | Async support, Pydantic validation |
| **Vector DB** | Qdrant | Simple API, self-hosted |
| **Embeddings** | nomic-embed-text (768-dim) | Runs locally via Ollama |
| **LLM** | Groq or Ollama | Cloud speed vs. local privacy |

### The RAG Pipeline

```
Question -> Embed -> Vector Search -> Retrieve Chunks -> LLM Analysis -> Structured Answer
```

1. User asks: "What caused the connection failures?"
2. Question gets embedded into a 768-dim vector
3. Qdrant finds the most similar log chunks
4. Top chunks become context for the LLM
5. LLM returns structured JSON with root cause + evidence

---

## Key Technical Decisions

### Smart Chunking

Not all logs are equal. I prioritize:
- **ERROR/FATAL** logs (highest priority)
- **WARN** logs (medium priority)
- **Stack traces** (grouped together)
- **Temporal context** (nearby lines for context)

### Concurrent Embeddings

Processing 10MB of logs sequentially would take forever. I use ThreadPoolExecutor with 5 workers for parallel processing.

### Dual LLM Support

- **Ollama (local)**: ~15 tokens/sec, private
- **Groq (cloud)**: 500+ tokens/sec, free tier

---

## Performance

**Test file:** Zookeeper.log (9.94 MB, 74,380 lines)

| Metric | Result |
|--------|--------|
| Ingestion | 34 seconds |
| AI Analysis | 2.5 seconds (Groq) |
| Chunks stored | 50 (error-prioritized) |

---

## Try It Yourself

```bash
git clone https://github.com/Avinash-Amudala/llm-incident-copilot.git
cd llm-incident-copilot
cp .env.example .env
# Add your GROQ_API_KEY to .env (optional)
docker compose up --build
```

Open http://localhost:5173 and upload a log file.

---

## What I Learned

1. **RAG works for logs** - semantic search finds relevant context
2. **Evidence is everything** - citations build trust
3. **Speed matters** - Groq's free tier is a game-changer
4. **Chunking is underrated** - smart chunking > more chunks

---

## Future Improvements

- Hybrid retrieval (BM25 + vectors)
- Metrics correlation (CPU/memory/latency)
- Multi-file analysis
- Evaluation harness for retrieval quality

---

**GitHub:** https://github.com/Avinash-Amudala/llm-incident-copilot

*Feedback welcome - especially from engineers working in infrastructure, SRE, or applied AI.*
