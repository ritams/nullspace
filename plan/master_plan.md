# Nullspace: Master Plan

**Product Name**: Nullspace
**Domain**: `nullspace.life`
**Core Proposition**: An AI-native research workspace built for speed. It combines a radically fast collaborative LaTeX editor (beating Overleaf) with a deeply integrated AI agentic layer that structures chaotic research dumps into polished academic papers.

---

## 1. Product Vision & Essential Features

Nullspace aims to be the fastest, bloat-free LaTeX editor paired with a powerful, intelligent research workspace.
*   **Real-time collaborative LaTeX editor**: Powered by CodeMirror 6 with surgical DOM updates.
*   **Lightning-fast Compilation**: Leveraging pre-warmed TexLive container pools and true incremental compilation to bring compile wait times from ~30s down to <3s.
*   **Intelligent Knowledge Graph**: A sandbox where researchers drag-and-drop PDFs, voice memos, images, and notes, while AI automatically extracts meaning and structures a paper around them.
*   **Optimistic UI Principles**: Skeleton screens instead of spinners, Web Workers to offload heavy processes, and seamless cross-fading differential PDF rendering to completely eliminate visual "flashes".

---

## 2. Technical Architecture & Tech Stack

The architecture centers around the philosophy of **"Fast & Lean"**, avoiding legacy bloat (e.g., heavy SPA frameworks or blocking synchronous requests).

### Tech Stack Choices
*   **Frontend**: SvelteKit (Smaller bundle, faster runtime, surgical DOM updates, built-in SSR streaming).
*   **Editor**: CodeMirror 6 (Industry standard, off-main-thread workers for highlighting).
*   **API/BFF**: Hono.js on Bun (Fastest JS HTTP server, minimal overhead).
*   **Real-time Collaboration**: Yjs + Hocuspocus (CRDT over WebSockets with binary encoding).
*   **Database**: PostgreSQL + pgvector (Unified store for relational data and semantic vector search).
*   **Job Queue**: BullMQ + Redis (Battle-tested queue for routing long-running compile tasks).
*   **Compiler Engine**: TexLive + latexmk.
*   **AI Architecture**: Vercel AI SDK (Model-agnostic, streaming-first).
*   **ORM**: Drizzle.
*   **Auth**: Better-auth (or Lucia).
*   **Storage**: Cloudflare R2 (Cost-effective S3-compatible, no egress fees).
*   **Infrastructure**: Initially on Fly.io, migrating to managed Kubernetes (EKS/GKE) for scale.

---

## 3. High-Level Repository Structure

A monorepo structure (configured with Turborepo or Nx) built to scale from 10k to 1M+ users:

```text
research-workspace/
├── apps/
│   ├── web/                    # SvelteKit App (Frontend)
│   │   ├── src/routes/(app)/workspace/[id]/   # Research dump UI
│   │   └── src/routes/(app)/editor/[projectId]/ # CodeMirror editor + PDF.js
│   │
│   ├── compiler-service/       # LaTeX Compilation Engine
│   │   ├── queue/              # BullMQ job queue handler
│   │   ├── sandbox/            # Container management (gVisor/Firecracker)
│   │   ├── compiler/           # latexmk wrapper & logic
│   │   └── ws/                 # Real-time WebSocket log streaming
│   │
│   ├── ai-service/             # AI & Agentic Logic
│   │   ├── agents/             # Drafter, Citation, Figure, Voice agents
│   │   ├── rag/                # Embeddings, retrieval, chunking
│   │   └── latex/              # AST parser & writer
│   │
│   ├── collab-service/         # CRDT / Yjs
│   │   ├── crdt/               # Yjs document synchronization
│   │   └── presence/           # Live cursors & presence
│   │
│   └── api/                    # Main Gateway / BFF (Hono + Bun)
│       └── src/routes/         # projects, files, users, auth, billing
│
├── packages/                   # Shared Packages
│   ├── types/                  # Shared TypeScript interfaces
│   ├── ui/                     # Svelte Component library
│   ├── latex-utils/            # LaTeX AST handling utilities
│   └── db-schema/              # Drizzle Schemas
│
├── infra/                      # Infrastructure as Code
│   ├── terraform/              # DB, Redis provisioning (Prod/Staging)
│   ├── k8s/                    # HPA configurations for scale
│   └── docker/                 # Local dev setups & TexLive images
│
└── turbo.json                  # Turborepo configurations
```

---

## 4. The Compiler Service (Technical Differentiator)

The compiler is optimized to eliminate wait times that plague platforms like Overleaf:
1.  **Pre-warmed Container Pool**: Maintain 5-10 TexLive containers running at all times to bypass cold-start latency.
2.  **True Incremental Compilation**: Aggressively cache `.aux`, `.toc`, and `.bbl` files. Hashing detects exactly which `.tex` files changed, reducing `latexmk` passes significantly. 
3.  **Predictive Compilation**: Automatically initiate silent background compiles whenever a user pauses typing for 800ms.
4.  **WebSocket Streaming**: Compilation logs stream line-by-line in real-time.
5.  **Differential PDF Swap**: Never "flash" a blank screen on re-render. Detect changed pages, render them off-screen in a Web Worker, and cross-fade them in exactly at the user's current scroll position.

---

## 5. The AI Agentic Layer (The Moat)

A context-aware AI pipeline that organizes unstructured, messy inputs into academic structures:

### Ingestion & Processing
*   **Media supported**: PDFs, voice memos (Whisper), handwritten notes (OCR + Vision), data notebooks, and raw images.
*   **Action**: Everything dropped in the workspace undergoes automatic chunking, entity extraction, and embedding (pgvector) to build a unified project knowledge graph.

### Core Agents
*   **Project Understanding Agent**: Automatically analyzes new files and updates a project summary (e.g., "Identified 3 datasets and 1 proposed method comparing against 2 baselines.").
*   **Structure Proposal Agent**: Proposes academic structures (NeurIPS, IEEE, etc.) identifying which sections are ready, partial, or missing based on the repository content.
*   **Section Drafting Agent**: Grounded synthesis. The AI pulls specific user quotes, equations, and literature from the workspace and writes the LaTeX section directly, complete with generated citations.
*   **Figure Intelligence Agent**: Auto-captions uploaded images/plots and generates the appropriate `\begin{figure}` blocks.
*   **Citation Agent**: Parses a pasted abstract, DOI, or PDF directly into a valid BibTeX layout.
*   **Voice Memo Agent**: Transcribes "shower thoughts" and maps insights systematically to sections like Discussion/Limitations.
*   **Consistency Agent**: A continuous background loop catching discrepancies (e.g., "Figure 4 shows data inconsistent with your claim on line 47").

All LaTeX manipulation relies on **AST (Abstract Syntax Tree) parsing**, absolutely avoiding naive string replacement that breaks LaTeX builds.

---

## 6. Implementation & Scaling Strategy

### Frontend-First Milestone Plan
*   **Week 1-2: Frontend Shell (Zero Backend)**
    *   SvelteKit, CodeMirror 6, and PDF.js wired up with hardcoded mock data. Validate the cross-fade PDF transitions, lazy loading layouts, and overall application *feel*.
*   **Week 3-4: Auth + File System Baseline**
    *   Better-auth integration, Postgres/Drizzle setup, Cloudflare R2 file persistence. Create/rename/delete workflows mapped to real persistent storage.
*   **Week 5-6: The Compiler Service Engine**
    *   TexLive dockerization, BullMQ integration, and WebSocket event logging. The core "LaTeX Editor" MVP comes alive here.
*   **Week 7-8: Collaboration & Editor Polish**
    *   Yjs/Hocuspocus for live multiplayer, SyncTeX (click PDF -> jump to source code), and workspace error-highlighting interfaces.
*   **Week 9+: The Intelligent Moat (AI Integration)**
    *   Rolling out the ingestion pipelines, semantic search over the workspace, and the dedicated drafting agents.

### Evolution Phase Map
*   **Phase 1 (0–10k Users)**: Run monolithically deployed services scaling horizontally on **Fly.io** (Single Postgres read/write + Redis instance). ~$300–600/mo.
*   **Phase 2 (10k–100k Users)**: Migrate to **Managed K8s (EKS/GKE)**. Implement Horizontal Pod Autoscaling (HPA) strictly on compiler queue depths. Introduce **Qdrant** internally for complex vector search and Postgres read replicas. ~$3k–8k/mo.
*   **Phase 3 (100k–1M+ Users)**: Globally distributed compiler clusters (EU, US, Asia). Adopt Citus or Neon infrastructure for database horizontal sharding. Completely decouple billing/auth services.
