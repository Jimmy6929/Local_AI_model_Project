PROJECT PROMPT (Obsidian Notes Only — No Code Output)

IMPORTANT RULES FOR THE AI (READ FIRST)
- This is an Obsidian documentation vault, NOT a real code repository.
- You must NOT write executable code, scripts, config files, or runnable commands.
- Only write structured notes in plain text that are markdown-friendly.
- Treat this like a real software project, but every “file” is a note.
- Whenever you produce output:
  1) Show folder structure first
  2) Then write each file’s content under “File: /path/to/file.md”
- Keep everything clean, readable, and production-ready in architecture.
- Assume the reader is building this step-by-step using VS Code, Copilot, and Obsidian.

============================================================
1) SUMMARY: WHAT THIS PROJECT IS BUILDING AND WHY
============================================================

Goal:
Build a Private Personal AI Assistant now, structured for future multi-user deployment.

Core experience:
- A chat UI (web app) that feels like ChatGPT
- Two modes:
  (1) Instant Mode: fast + cheap, used for most tasks
  (2) Thinking Mode: stronger model, cold start allowed, used only when needed

Key constraints:
- No third-party model API usage for the core assistant (no paid “LLM provider API” usage).
- Rented compute instead (GPU pod for instant, serverless GPU for thinking).
- Data and files are stored in Supabase with multi-tenant safety:
  - Supabase Auth for users
  - Postgres for data
  - Storage for uploads
  - RLS (Row Level Security) enforced at database-level
- Development is local-first now, but must be deployable later:
  - Local Supabase in development
  - Supabase Cloud in production (swap environment variables later)

Architecture shape (OpenClaw-like concept):
- A Gateway / Control Plane service that owns sessions, routing, tools, and safety policies.
- Model servers are “dumb inference endpoints” behind the Gateway.
- Optional future: add channel adapters (Telegram/Discord/Slack) that route into the same Gateway.

Success criteria:
- Instant replies feel responsive for daily work
- Thinking mode works reliably (cold start acceptable)
- User data isolation is correct and safe (RLS policies)
- Easy “Local → Staging → Production” transition

============================================================
2) HIGH-LEVEL SYSTEM DESIGN (THE ONE-PAGE PICTURE)
============================================================

A) Web App (Next.js)
- Chat UI, session history, deep-think toggle, file upload
- Auth via Supabase Auth
- Calls Gateway API for chat + tools + RAG

B) Gateway / Brain (FastAPI)
- Central control plane
- Validates user identity (Supabase JWT)
- Stores chats/sessions in Supabase
- Performs RAG (embeddings + vector search) to retrieve relevant chunks
- Routes requests:
  - Default → Instant Inference
  - Toggle or complex request → Thinking Inference
- Executes tools (web search, notes, tasks) safely in backend

C) Inference Tier 1: Instant
- GPU Pod (always-on-ish)
- Runs a fast instruct model
- Exposes a private endpoint (not public)

D) Inference Tier 2: Thinking
- Serverless GPU endpoint (scales-to-zero)
- Runs a stronger model
- Cold start tolerated
- Strict cost controls (token limits, concurrency)

E) Supabase (Local now, Cloud later)
- Auth (users, sessions, JWT)
- Postgres database (tables, RLS)
- Storage (documents)
- pgvector embeddings for RAG

F) Security & Networking
- During private dev: VPN (Tailscale) + firewall to keep inference private
- For multi-user later: strict API auth + rate limits + audits; inference still private to backend

============================================================
3) WHAT TO BUILD FIRST (MVP ORDER)
============================================================

Phase 1 — “Chat works end-to-end”
1) Create Obsidian “repo structure” notes (this vault).
2) Local Supabase setup plan (Auth, DB, Storage).
3) Gateway API plan:
   - one endpoint: /chat
   - stores messages
   - calls Instant inference endpoint
4) Web UI plan:
   - login
   - chat window
   - sessions sidebar

Phase 2 — “Two modes”
5) Add Thinking inference plan:
   - /think endpoint on serverless
6) Add UI toggle for Deep Think
7) Add routing heuristics (optional) and show “used_mode” in UI

Phase 3 — “Memory and documents”
8) Document upload pipeline plan:
   - upload → extract text → chunk → embed → store
9) RAG retrieval plan:
   - embed query → vector search top K → minimal context injection

Phase 4 — “Tools and safety”
10) Add a small tool set:
    - web_search
    - save_note
    - list_tasks / add_task
11) Tool safety rules:
    - explicit allowlist
    - backend executes tools, model only suggests
    - log everything

Phase 5 — "Production readiness"
12) Local → Production migration plan:
    - environment variable switching
    - migrations pushed to Supabase Cloud
    - operational checklists (auth, RLS, storage, backups, monitoring)

============================================================
4) DEFINITIONS (SO NOTES STAY CONSISTENT)
============================================================

- Gateway / Control Plane:
  The central backend service that owns: auth, routing, sessions, tools, memory (RAG), policies.

- Instant Mode:
  A fast/cheap model endpoint for most tasks. Always-on-ish. Low latency.

- Thinking Mode:
  A stronger model endpoint. Serverless scale-to-zero. Cold start ok.

- RAG (Retrieval-Augmented Generation):
  Retrieve relevant stored text chunks and send only minimal context to the model.

- RLS (Row Level Security):
  Database-level policies ensuring users can only access their own rows.

- Multi-tenant:
  A pattern where many users share one database; isolation is enforced via user_id + RLS.

============================================================
5) OBSIDIAN VAULT STRUCTURE (FOLDERS + FILES)
============================================================

PROJECT STRUCTURE:

/00_README
  overview.md
  glossary.md

/01_PRODUCT
  vision-and-goals.md
  mvp-scope.md
  success-metrics.md

/02_ARCHITECTURE
  system-overview.md
  data-flow.md
  environments-local-staging-prod.md
  openclaw-style-gateway-notes.md

/03_INFRA
  inference-instant.md
  inference-thinking.md
  networking-and-private-access.md
  cost-controls.md
  deployment-plan.md

/04_DATABASE_SUPABASE
  supabase-local-dev-plan.md
  schema.md
  rls-policies.md
  auth-and-jwt.md
  storage-and-files.md
  embeddings-and-pgvector.md
  migrations-and-release-process.md

/05_GATEWAY_API
  gateway-responsibilities.md
  endpoints-contracts.md
  routing-logic.md
  rag-flow.md
  tools-framework.md
  logging-and-auditing.md
  rate-limiting-and-abuse.md

/06_WEB_APP
  ui-pages-and-components.md
  chat-ux.md
  auth-ux.md
  file-upload-ux.md

/07_SECURITY
  threat-model.md
  prompt-injection-notes.md
  secrets-management.md
  production-hardening.md

/08_ROADMAP
  phase-1-chat-mvp.md
  phase-2-two-modes.md
  phase-3-rag.md
  phase-4-tools.md
  phase-5-production-launch.md

/09_DECISIONS (Architecture Decision Records)
  adr-0001-two-tier-inference.md
  adr-0002-supabase-local-then-cloud.md
  adr-0003-openai-compatible-inference-interface.md
  adr-0004-rls-multi-tenant-from-day-1.md

============================================================
6) STEP-BY-STEP BUILD PLAN (NO CODE, ONLY ACTION STEPS)
============================================================

STEP 0 — Decide “Local Dev Strategy”
- Choose local Supabase strategy:
  - Preferred: Supabase CLI local stack for dev parity (Auth + Storage + DB).
- Decide if spare MacBook is:
  - Dev box only (can sleep; minimal ops)
  - Always-on staging (disable sleep; backups; monitoring)

STEP 1 — Create Supabase data model (multi-user ready from day 1)
- Create the tables:
  - profiles
  - chat_sessions
  - chat_messages
  - documents
  - document_chunks (with embeddings vector)
- Add indexes and constraints (especially for session_id, user_id, created_at).
- Turn on RLS for every user-owned table.
- Write RLS policies using auth.uid() pattern:
  - Each table should only allow select/insert/update/delete where user_id = auth.uid()

STEP 2 — Define “API Contracts” BEFORE implementation
- Define request/response schemas for:
  - POST /chat
  - POST /files
  - GET /sessions
  - GET /messages
  - (Optional) POST /tools/...
- Include fields:
  - session_id
  - mode (instant|think)
  - tools_enabled
  - citations (if RAG retrieval used)
  - used_mode + latency metadata

STEP 3 — Build the Gateway responsibilities (as notes)
- Gateway must:
  1) Authenticate user (Supabase JWT)
  2) Create/append session + store message
  3) Optional: retrieve RAG context
  4) Route to inference tier
  5) Optional: run tools, log results
  6) Store assistant response
  7) Return response to UI

STEP 4 — Standardize the inference interface
- Make both inference tiers “look the same” to the Gateway.
- Use an OpenAI-compatible API shape for:
  - Instant inference server
  - Thinking serverless endpoint
- Document:
  - model name mapping
  - max tokens / temperature defaults per mode
  - safety defaults (no tool execution at model layer)

STEP 5 — Instant Inference (GPU Pod) plan
- Choose a GPU type and runtime.
- Document operational design:
  - always-on-ish for responsiveness
  - private endpoint only
  - process manager / auto-restart
  - simple health checks
- Define “Instant model selection criteria”:
  - good coding + planning + writing
  - fits VRAM
  - low latency

STEP 6 — Thinking Inference (Serverless) plan
- Document “serverless constraints”:
  - cold start allowed
  - scale-to-zero to save cost
  - strict token cap and concurrency cap
- Decide “cold start mitigation”:
  - accept cold start OR keep minimal warm workers for responsiveness when needed

STEP 7 — Private networking plan (dev + staging)
- Document threat model:
  - model endpoints must not be public
  - only Gateway can reach inference
- Document access model:
  - Tailscale mesh for private access (laptop + gateway + model servers)
  - firewall rules: only allow model ports from VPN subnet
- For multi-user production:
  - inference endpoints remain private to backend network
  - Gateway is the only public surface (HTTPS)

STEP 8 — Web App plan (Next.js)
- Pages:
  - /login
  - /app/chat
- UX requirements:
  - session list sidebar
  - deep think toggle
  - file upload
  - show which mode was used
  - show citations when RAG used
- Auth:
  - Supabase Auth login
  - store session token safely
  - call Gateway with JWT

STEP 9 — File pipeline and RAG plan
- Define supported file types for MVP (pdf, txt, md).
- Pipeline:
  1) upload file to Storage
  2) extract text
  3) chunk into consistent sizes
  4) embed chunks
  5) store chunks + embeddings
- Retrieval:
  - vector search top K
  - minimize context inserted
  - keep private info minimal

STEP 10 — Tools framework plan (start small)
- Tools list for MVP:
  - web_search(query)
  - save_note(title, content)
  - list_tasks()
  - add_task(title, due_date?)
- Safety rules:
  - backend-only execution
  - explicit allowlist
  - log every tool call
  - do not allow “arbitrary code execution” tools

STEP 11 — Cost controls plan
- Instant:
  - manual stop when idle OR scheduled shutdown
  - monitor GPU-hours
- Thinking:
  - token caps
  - timeouts
  - concurrency limits
  - usage dashboard in UI (later)
- Add an internal “budget guardrail” note:
  - max daily spend
  - max requests per hour
  - alerting triggers

STEP 12 — Local → Production release plan (future multi-user)
- Define environments:
  - Local: local Supabase stack
  - Staging: hosted Supabase project or always-on home stack
  - Production: Supabase Cloud
- Migration process:
  - migrations tracked in version control
  - apply migrations locally first
  - apply to staging
  - then production
- Security checklist for launch:
  - RLS verified on all tables
  - storage bucket permissions verified
  - service role key never exposed to client
  - rate limiting enabled
  - audit logs enabled

============================================================
7) WHAT THE AI SHOULD DO WHEN USER ASKS FOR "NOTES"
============================================================

When the user asks for any feature note, you must:
- Place the note into the correct folder/file from the structure above.
- If the file does not exist, create it and add it to the structure.
- Write in a "production architecture repo" style:
  - Goal
  - Non-goals
  - Requirements
  - Data model impact
  - API contracts impact
  - Security considerations
  - Cost considerations
  - Milestones / acceptance criteria

You must avoid:
- Writing code
- Writing configuration files
- Providing command sequences
- Overly long essays without structure

END OF PROMPT