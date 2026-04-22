# AI Employer Radar
AI-powered company intelligence tool for job seekers. Multi-agent system using n8n, Groq, and public data sources.

## Overview
This project demonstrates a multi-agent AI workflow that analyzes companies from a job seeker's perspective. A user sends a message with a company name via a Telegram bot; the system orchestrates multiple agents in n8n to gather signals about the compant (news, community sentiment, general company overview) and returns a synthesized, structured summary to the user in Telegram.

💡 This is a concept and learning project, focused on exploring agent orchestration, underlying systems, LLMs and their capabilities and limitations, and workflow design.

## Live demo & assets
Note: This project runs on free-tier infrastructure. Responses may be delayed or occasionally unavailable.
- **Telegram Bot:** link to the bot will be added later, once everything's finished
- **Demo Video:** [early-stage video](https://youtube.com/shorts/NBY-KUTxT3A?feature=share) (in progress, to be replaced)
- **Screenshots (n8n workflows):**
  - Main workflow: <img width="1877" height="419" alt="demoscreen" src="https://github.com/user-attachments/assets/33409b4f-aa35-42e5-ae72-7e89eb9c5c59" />

## Who this is for
Job seekers who want to:
- Quickly understand a company behind a job posting
- Get a concise overview without browsing multiple sources
- See signals like recent news and community sentiment in one place
- Make a decision of whether this company is right for them

## Problem
When evaluating a company as a candidate:
- Information is scattered across multiple sources
- Search results are noisy and time-consuming
- It’s hard to quickly form a structured view

## How the tool helps
The tool helps answer typical questions a candidate might have:
- What does this company actually do?
- What’s happening around it recently?
- What is the general sentiment or signal around it?

## Solution
A Telegram-based interface that:
- Accepts a company name
- Triggers a multi-step agentic workflow
- Gathers different types of signals via specialized agents and from different sources
- Returns a concise, structured summary within seconds

## Architecture
```bash
User (Telegram)
        ↓
Telegram Bot
        ↓
n8n Workflow (Orchestration)
        ↓
[Overview Agent]
[News Agent]
[Community Signal Agent]
        ↓
[Synthesis Agent]
        ↓
Response → Telegram
```
## Components
- **Interface:** Telegram
- **Orchestration:** n8n workflows
- **AI layer:** external LLM APIs
- **Optional / Temporary storage:** Neon
- **Hosting:** Self-hosted n8n on Render

## Key Agents
### Overview Agent
- Generates a concise company profile
- Answers: what the company does, its industry, positioning, basic context; may include foundation year and headquarters location
### News Agent
- Extracts recent news and developments (from different RSS feeds, for example, Google News RSS or Bing RSS)
- Focuses on relevant, recent signals rather than exhaustive lists, filters out noise, caps by amount and recency
### Community Signal Agent
- Surfaces general sentiment and discussion signals from communities (like Reddit)
- Aims to approximate “what people are saying”
- Filters out noise
### Synthesis Agent
- Combines outputs from all agents
- Produces a structured, readable summary for the user

## LLM Configuration
The system is LLM-agnostic:
- LLM providers are configurable inside n8n
- You can replace them with different models, different providers

This allows experimentation with:
- Cost vs quality vs time trade-offs
- Different prompting strategies

## Notes on Reproducibility
This project is reproducible in principle, but not one-click runnable.

To recreate:
- Install n8n (or host it on Render - preferred)
- Import workflows from /workflows
- Create a chatbot in Telegram or other messengers, and connect it in workflows
- Change workflows as needed and test
- Configure credentials:
 - Messenger bot 
 - LLM provider API keys and LLM names
- Activate workflows (publish)

## Disclaimer
This is a learning and exploration project. Built to experiment with agent-based workflows and orchestration. Developed iteratively with the help of AI tools (I used Claude Pro, mainly Opus 4.7 on Pro subscription). Not intended as a production-grade system.

I do not claim this as an expert-level implementation. The goal was to take an idea and make it real, while learning through the process.

## Why This Project
Searching a new job in IT industry is not easy these days. AI caused multiple lay offs in different companies, and I believe knwowledge of AI tools and AI orchestration is essential for many roles in the sector. Learning by doing is the concept that helps candidates become more competitive. 

This project demonstrates:
- Breaking down a problem into cooperating AI agents
- Orchestrating workflows using n8n
- Integrating messaging interfaces with LLM systems
- Making practical trade-offs under real-world constraints

## Limitations
- Dependent on external LLM APIs
- Signal quality depends on prompts and sources
- Free-tier hosting may introduce latency or downtime
