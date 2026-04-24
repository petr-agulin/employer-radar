# AI Employer Radar

AI-powered company intelligence tool for job seekers. Built as a multi-pipeline system with a true AI agent at the synthesis step, using n8n, Groq LLMs, MCP, and public data sources.

> 🍀 **This is a personal learning and portfolio project, not a production service.** It was built to experiment with agentic workflows, LLMs, and AI orchestration tools. I do not claim this as an expert-level implementation. Data may be incomplete or outdated. Use as a starting point or a source of ideas, not as a definitive source.

---

## What it does

Send a company name to a Telegram bot. Employer Radar returns a structured brief covering:

- 🏢 **Company profile** — what they do, size, headquarters, founding, what makes them notable
- 📰 **News signals** — recent developments filtered for employer relevance
- 💬 **Community sentiment** — what professionals say on Reddit and Hacker News
- 🧠 **AI synthesis** — an agent-generated take integrating all three sources, enriched with a targeted web search

Each section carries a signal — 🟢 Positive / 🟡 Neutral / 🔴 Negative / ⚪ Insufficient data — so you can scan quickly before deciding whether to dig deeper.

---

## Why this was built

Job searching in tech requires fast, structured due diligence. When job seekers read a job ad for their role, they often want to:
- Quickly understand a company behind a job posting
- Get a concise overview without browsing multiple sources
- See signals like recent news and community sentiment in one place
- Decide whether this company is right for them

But information about companies is scattered across news sites, review platforms, and community forums. To address this, the tool consolidates the information into one readable, structured brief, delivered in under 2 minutes (on average).

It was also built as a hands-on way to explore agent orchestration, LLM integration, and modern AI tooling — technologies increasingly relevant for product and engineering roles.

---

## Architecture

The system is built in n8n and organized as a main orchestration workflow that calls four specialized sub-workflows (agents) in sequence.

```
User (Telegram)
      ↓
Input Validation (LLM)
      ↓
┌─────────────────────────────────────┐
│  News Agent         → evidence DB   │
│  Community Agent    → evidence DB   │
│  Profile Agent      → evidence DB   │
└─────────────────────────────────────┘
      ↓
Synthesis Agent (true AI agent with tool use via MCP)
      ↓
Brief Formatter
      ↓
Response → Telegram
      ↓
Cleanup (DELETE evidence rows)
```

### What makes Synthesis different from the other agents

News, Community, and Profile agents are **deterministic pipelines**: fixed sequences of HTTP fetches, LLM classification calls, and structured parsing. They run reliably and predictably.

The Synthesis Agent is a **true AI agent** built with n8n's AI Agent node. It receives a goal ("produce an employment brief for this company") and uses tools to accomplish it — deciding which to call, in what order, based on what the evidence actually contains. It connects to an internal MCP (Model Context Protocol) server that exposes two tools:

- `read_evidence_row` — reads a specific agent's output from the evidence database
- `search_web` — runs a targeted web search to enrich the brief with one additional angle

The agent reads all three evidence rows, selects a relevant web search angle based on what the evidence already covers, and produces a synthesized brief. This is where cross-source reasoning happens.

---

## Agent design

### Input Validation
Lightweight binary classifier. Decides whether the input looks like a plausible organization name (vs. a question, command, or gibberish). Intentionally permissive — when in doubt, it proceeds rather than rejects.

### News Agent
Fetches Google News RSS and Bing News RSS for the company name. Runs two LLM passes: a fast pre-filter that discards clearly irrelevant articles, and a deeper filter that selects the most employer-relevant signals and produces a summary. Age filtering ensures recency. Handles rate limits and parse failures with graceful fallback.

### Community Agent
Queries a broad multi-subreddit search and Hacker News. Runs the same two-pass approach: classify first, summarize second. Focused on professional sentiment — compensation, culture, management, hiring — rather than consumer discussion.

### Profile Agent
Single LLM call using Groq's compound-mini model, which has built-in web search. Produces a structured company dossier (WHAT / SIZE / LOCATION / FOUNDED / NOTABLE). Prioritizes operational headquarters over legal/tax incorporation location.

### Synthesis Agent (AI Agent with MCP tools)
Reads all three evidence rows via MCP tool calls. Selects one web search angle contextually — if news is thin, it searches for recent news; if community flagged management issues, it searches leadership; if evidence is rich, it looks for something forward-looking. Produces OVERALL signal + BRIEF (3-5 sentences) + ADVICE (one genuine take for a job seeker).

---

## MCP integration

An internal MCP server (`MCP Server — Synthesis Tools`) is hosted as an n8n workflow and exposes two tools to the Synthesis Agent via n8n's MCP Client node. This centralizes the tool definitions — if the underlying implementation changes (e.g., DB schema or search model), it changes in one place.

This is MCP used for internal tool sharing, not external exposure. The architecture is MCP-ready for external access (e.g., Claude Desktop integration) if desired later.

---

## LLM allocation

Each LLM was chosen for a specific role based on capability, rate-limit bucket, and task requirements. Models can be swapped in n8n without changing pipeline logic.

| Step | Model | Why |
|---|---|---|
| Input Validation | `llama-3.1-8b-instant` | Fast binary classification; tiny token footprint; generous daily limits |
| News pre-filter | `openai/gpt-oss-120b` | High accuracy on relevance judgment; isolated rate-limit bucket |
| News deep filter | `meta-llama/llama-4-scout-17b-16e-instruct` | Strong summarization; 30K TPM headroom; shared with Community |
| Community pre-filter | `meta-llama/llama-4-scout-17b-16e-instruct` | Same bucket as News deep filter; tasks are complementary, not concurrent |
| Community summary | `meta-llama/llama-4-scout-17b-16e-instruct` | Consistent model across Community pipeline |
| Company Profile | `groq/compound-mini` | Built-in web search — required for real-time company data; 3× lower latency than compound |
| Synthesis web search tool | `groq/compound-mini` | Same model, same bucket, separate call |
| Synthesis orchestration | `llama-3.3-70b-versatile` | Best available tool-calling reliability; isolated bucket; no reasoning-token overhead |

The system is LLM-agnostic. Models are configured inside n8n nodes and can be replaced with alternatives from any OpenAI-compatible provider. The allocation above reflects free-tier Groq rate limits — each model operates in a separate rate-limit bucket where possible, reducing contention across concurrent agent calls.

---

## Tech stack

| Layer | Technology |
|---|---|
| Interface | Telegram Bot |
| Orchestration | n8n (self-hosted on Render) |
| AI Agent | n8n AI Agent node with MCP Client |
| MCP Server | n8n MCP Server Trigger (internal) |
| LLM inference | Groq (free tier) |
| Temporary storage | Neon PostgreSQL (Frankfurt) |
| Data sources | Google News RSS, Bing News RSS, Reddit multi-subreddit search, Hacker News Algolia API |

---

## Live demo & screenshots

> Note: My system runs fully on free-tier infrastructure. Responses typically take 30–240 seconds, depending on the LLM model and input query. Occasional unavailability can happen due to LLM rate limits.

- **Telegram Bot:** https://t.me/EmployerRadarBot 
- **Demo video:** [YouTube](https://www.youtube.com/watch?v=nprC_gGGYNU)
- **Screenshots:**
<img width="1748" height="530" alt="Main flow" src="https://github.com/user-attachments/assets/2390014c-94e1-443b-84a7-a6c036d91050" />
...
<img width="1136" height="688" alt="AI Agent Synthesis" src="https://github.com/user-attachments/assets/a7e85e30-3724-4317-abd9-2fa1c789c58e" />
...
<img width="850" height="594" alt="TelegramStartBot" src="https://github.com/user-attachments/assets/84449515-a0af-4d37-a34e-4cdac355165b" />
...
<img width="478" height="1083" alt="TelegramResultMessage" src="https://github.com/user-attachments/assets/22e57337-3135-44d9-bb69-2e2285af1de9" />

## Reproducing this project

This project is reproducible but not one-click deployable. To recreate:

1. Host n8n on Render (or run locally)
2. Create a Neon PostgreSQL database; create the `evidence` table
3. Create a Telegram bot via BotFather
4. Get a Groq API key (free tier is sufficient)
5. Import workflows from `/workflows` in this order: sub-workflows first, then agents, then the main workflow
6. Configure credentials in n8n: Telegram token, Groq API key, Postgres connection
7. Activate all sub-workflows before activating the main workflow
8. Test via Telegram

The evidence table schema:
```sql
CREATE TABLE evidence (
  id SERIAL PRIMARY KEY,
  company TEXT,
  run_id TEXT,
  agent TEXT,
  signal TEXT,
  summary TEXT,
  collected_at TIMESTAMP DEFAULT NOW()
);
```

---

## Limitations

- My implementation is fully dependent on free-tier LLM APIs — rate limits can cause occasional failures and/or degraded output, especially when burst testing
- News and community data is English-language biased; non-English companies may return thinner results
- Companies that rebranded (e.g., Facebook → Meta, Twitter → X) may return incomplete news results if searched by the old name
- Signal quality depends on available public discussion — niche or private companies may return ⚪ across sources
- Not a comprehensive news system — the pipeline is configured to search for the most recent news, and the data reflects what was publicly available at the time of the query

---

## About

Built by [Petr Agulin](https://github.com/petr-agulin) as a portfolio project exploring agent orchestration, LLM integration, and AI workflow design. Developed iteratively with Claude (Anthropic) as a coding and architecture thinking partner (Sonnet 4.5, Sonnet 4.6, but mostly with Opus 4.6 and Opus 4.7) in about 10 days, working late evenings and weekends. 

The original goal: take a real problem (company research for job seekers), build something that actually works, and learn through the process.
