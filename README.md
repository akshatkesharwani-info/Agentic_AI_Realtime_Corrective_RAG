\# Real-Time Corrective RAG — Router + Self-Grading + Web Fallback + Semantic Cache

A company assistant that knows when to trust its own documents, when to admit they're not enough and go check the live web instead, and when to just reuse a cached answer because it already solved this exact question.

**Try it yourself → [Open in Colab](https://colab.research.google.com/) | Just run all cells and paste a free [Groq](https://console.groq.com) API key.**

## Problem
A company bot has to answer two kinds of questions: private ones (leave policy, expenses) and live ones (weather, news, sports). A basic RAG bot also has a second, sneakier problem — if the internal search returns weak or irrelevant chunks, it still answers from them instead of admitting it doesn't know. That's how confident wrong answers happen.

## Approach
- **Router:** one LLM call decides if a question is internal-policy or needs live web info
- **Retrieval grader (Corrective RAG):** every retrieved chunk gets a yes/no relevance check before it's allowed into the prompt — irrelevant chunks are dropped
- **Web fallback:** if internal retrieval comes back empty after grading, the pipeline automatically falls back to a live DuckDuckGo search instead of guessing
- **Groundedness check:** after generating an answer, a second LLM call verifies every fact is actually supported by the context; if not, it regenerates once with a stricter prompt
- **Semantic cache:** paraphrased repeat questions (cosine similarity ≥ 0.85 against a past question) are answered instantly from cache instead of re-running the whole pipeline
- Tested on 6 mixed questions: internal, live, a paraphrase (should hit cache), and one policy question NOT in my documents on purpose (should trigger the corrective web fallback)

## Result (from my own run)
| Metric | Result |
|---|---|
| Routing correct | 6 / 6 |
| Semantic cache hits | 2 (served in 0.02–0.03 sec vs 2.9–4.8 sec for a full run) |
| Corrective web fallback triggered | 1 / 1 (exactly when internal docs had nothing relevant) |
| Groundedness checks passed | 4 / 4 non-cached answers |

The "pets in the office" question wasn't in my internal handbook at all — the grader correctly rejected the retrieved chunks as irrelevant, triggered the web fallback, and the assistant answered from live search instead of making something up from the wrong policy chunk.

## Tech Stack
Python · Groq API (`openai/gpt-oss-120b`) · FAISS · Sentence-Transformers · DuckDuckGo Search (ddgs) · pandas · matplotlib

## Why this matters
This is Project 5, the capstone of a 5-project Agentic AI series: RAG → structured/audited outputs → autonomous tool-calling agents → multimodal (vision) RAG → real-time Corrective RAG with routing and caching (this one).

*Note: the semantic cache is in-memory here for demo purposes — a production version would use Redis so it persists and scales across requests.*

---
**Akshat Kesharwani** — Fresher Data Analyst / Data Scientist
Portfolio: https://akshatkesharwani-info.github.io/ | GitHub: https://github.com/akshatkesharwani-info
