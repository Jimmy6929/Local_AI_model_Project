---
tags:
  - readme
cssclasses:
  - readme-doc
---

# Project Overview

## What Is This?

Local AI Assistant — a private AI assistant system designed with SaaS architecture from day one.

---

## Vision

Build a personal AI that feels like ChatGPT but runs on self-controlled infrastructure:
- No third-party LLM API costs for core inference
- Full data ownership via Supabase
- Two-tier inference for cost vs capability tradeoff

---

## Current Phase

- Single-user, private deployment
- Local development environment — see [[supabase-local-dev-plan]]
- Cloud GPU rentals for inference — see [[inference-instant]] and [[inference-thinking]]
- Local or self-hosted [[Supabase]] — see [[schema]]

---

## Future Phase

- Multi-user SaaS product — see [[phase-5-saas-launch]]
- Hosted [[Supabase]] Cloud — see [[environments-local-staging-prod]]
- Public web application — see [[ui-pages-and-components]]
- Secure, scalable, production-grade — see [[production-hardening]]

---

## Core Experience

A chat interface with two modes — see [[adr-0001-two-tier-inference]]:

| Mode | Purpose | Infrastructure | Latency |
|------|---------|----------------|---------|
| [[Instant Mode]] | Daily tasks (coding, writing, planning) | [[inference-instant|Always-on GPU pod]] | Fast |
| [[Thinking Mode]] | Complex reasoning, deep analysis | [[inference-thinking|Serverless GPU]] | Cold start OK |

---

## Key Principles

1. **SaaS-ready from day one** — [[rls-policies|Multi-tenant schema]], [[adr-0004-rls-multi-tenant-from-day-1|RLS]], [[environments-local-staging-prod|environment separation]]
2. **Security-first** — [[networking-and-private-access|Private inference]], [[auth-and-jwt|JWT auth]], [[secrets-management|no exposed secrets]]
3. **Cost-controlled** — [[inference-thinking|Scale-to-zero thinking]], [[cost-controls|budget guardrails]]
4. **Local-to-prod parity** — Same architecture in dev as production — see [[deployment-plan]]

---

## Quick Links

- [[glossary]] — Key terms and definitions
- [[02_ARCHITECTURE/system-overview]] — Technical architecture
- [[08_ROADMAP/phase-1-chat-mvp]] — Where to start

