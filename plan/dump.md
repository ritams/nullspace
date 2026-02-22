Hey, I am building AI native research workspace which is going to be extremely fast where people can dump their notes, research items, text figures and everything then using agentic manipulation they can create final report or paper by latex so it should have a latex compiler then the high selling feature will be the AI agentic stuff but the latex compilation is required and I want this to be extremely fast and efficient it the main criteria it has to be snappy, bloat free and everything. I want to compete with overleaf also at the same time I want to be much much much better than faster or leaner than overleaf, what the essential features of overleaf and what should the repository structure be for this sort of product and I want things to be scalable and potentially initially few thousand users actually initially I want to be this to be scalable to around 10 to 100 thousand users then we can scale up to millions yeah give me like the best ideas about how to build a fast scalable product like this what should be repository structure and everything, how many depository do we need and all of these things.

Show more
11:38 PM
This is a great product idea — essentially "Notion + Overleaf + AI agent" but fast and lean. Let me break this down properly.

What Overleaf Actually Does (Essential Features to Match/Beat)
Core:

Real-time collaborative LaTeX editor (CodeMirror-based)
LaTeX compilation (pdflatex, xelatex, lualatex via TexLive)
PDF preview with SyncTeX (click PDF → jump to source, and vice versa)
File tree (multi-file projects, \input, \include support)
Git integration / history / versioning
Templates gallery
Bibliography management (BibTeX/BibLaTeX)
Spell check, autocomplete for LaTeX commands
Where Overleaf is Slow/Bloated (your opportunity):

Compilation queues on free tier (30-60s waits)
Heavy frontend — Angular, tons of JS
No AI anywhere meaningful
No research workspace concept (it's purely a LaTeX editor)
Poor mobile experience
No notes/dump/research layer at all
Architecture Philosophy: Fast & Lean
The key decisions that make you fast:

1. Compilation Speed — This is your #1 differentiator

Run TexLive in isolated containers but keep them warm (pre-warmed pool)
Use incremental compilation — only recompile changed files via latexmk -pvc
Cache auxiliary files (.aux, .toc, .bbl) between compiles
Target: <3s for incremental, <10s for full compile
Use WebSockets to stream compilation logs in real time
2. Frontend — Bloat-free

SvelteKit or Next.js App Router — avoid heavy frameworks
CodeMirror 6 for the editor (same as modern Overleaf but configured lean)
PDF.js for rendering compiled PDFs
Target bundle: <200KB initial JS
3. AI Agentic Layer — Your real moat

Research dump → structured notes with AI tagging
AI can manipulate the LaTeX AST (not just chat)
"Write this section from my notes" → agent pulls relevant chunks, writes LaTeX
Citation agent: paste a DOI/URL → auto-generates BibTeX entry
Figure agent: upload image → generates \figure block with caption suggestions
Repository Structure
Monorepo using Turborepo or Nx. For 10k-100k users you want a monorepo — one repo, multiple deployable services, shared types/utilities. This scales to millions too with the same structure.

research-workspace/
├── apps/
│   ├── web/                    # SvelteKit or Next.js frontend
│   │   ├── src/
│   │   │   ├── routes/
│   │   │   │   ├── (auth)/
│   │   │   │   ├── workspace/[id]/
│   │   │   │   ├── editor/[projectId]/
│   │   │   │   └── api/        # API routes / BFF layer
│   │   │   ├── components/
│   │   │   │   ├── editor/     # CodeMirror setup
│   │   │   │   ├── pdf/        # PDF.js viewer + SyncTeX
│   │   │   │   ├── workspace/  # Notes, research dump UI
│   │   │   │   └── ai/         # AI chat panel, agent UI
│   │   │   ├── stores/         # Realtime state
│   │   │   └── lib/
│   │   └── package.json
│   │
│   ├── compiler-service/       # The LaTeX compilation engine
│   │   ├── src/
│   │   │   ├── queue/          # BullMQ job queue
│   │   │   ├── sandbox/        # Container/sandbox management
│   │   │   ├── compiler/       # latexmk wrapper, incremental logic
│   │   │   ├── cache/          # Aux file caching (Redis)
│   │   │   └── ws/             # WebSocket log streaming
│   │   ├── Dockerfile          # TexLive full + your service
│   │   └── package.json
│   │
│   ├── ai-service/             # All AI/agentic logic
│   │   ├── src/
│   │   │   ├── agents/
│   │   │   │   ├── writer.ts       # Section writing from notes
│   │   │   │   ├── citation.ts     # DOI → BibTeX agent
│   │   │   │   ├── figure.ts       # Figure handling agent
│   │   │   │   ├── reviewer.ts     # Paper review agent
│   │   │   │   └── orchestrator.ts # Multi-agent coordination
│   │   │   ├── rag/
│   │   │   │   ├── embeddings.ts   # Embed research notes
│   │   │   │   ├── retrieval.ts    # Semantic search over workspace
│   │   │   │   └── chunking.ts
│   │   │   ├── latex/
│   │   │   │   ├── ast-parser.ts   # Parse LaTeX to AST
│   │   │   │   └── ast-writer.ts   # AI edits via AST, not string replace
│   │   │   └── prompts/
│   │   └── package.json
│   │
│   ├── collab-service/         # Real-time collaboration
│   │   ├── src/
│   │   │   ├── crdt/           # Yjs document management
│   │   │   ├── presence/       # Cursor positions, awareness
│   │   │   └── persistence/    # CRDT → database sync
│   │   └── package.json        # Hocuspocus or y-websocket server
│   │
│   └── api/                    # Main REST/GraphQL API (gateway)
│       ├── src/
│       │   ├── routes/
│       │   │   ├── projects/
│       │   │   ├── workspace/  # Notes, research items
│       │   │   ├── files/
│       │   │   ├── users/
│       │   │   └── billing/
│       │   ├── middleware/
│       │   ├── db/             # Drizzle ORM or Prisma
│       │   └── auth/           # Better-auth or Lucia
│       └── package.json
│
├── packages/                   # Shared across apps
│   ├── types/                  # Shared TypeScript types
│   │   └── src/
│   │       ├── project.ts
│   │       ├── workspace.ts
│   │       ├── compiler.ts
│   │       └── ai.ts
│   ├── ui/                     # Shared component library
│   ├── latex-utils/            # Shared LaTeX parsing utils
│   ├── config/                 # ESLint, TS configs
│   └── db-schema/              # Single source of truth for DB schema
│
├── infra/                      # Infrastructure as code
│   ├── terraform/
│   │   ├── modules/
│   │   │   ├── compiler-pool/  # Warm container pool
│   │   │   ├── postgres/
│   │   │   ├── redis/
│   │   │   └── vector-db/
│   │   ├── staging.tf
│   │   └── prod.tf
│   ├── k8s/                    # Kubernetes manifests
│   │   ├── compiler/           # HPA for compiler pods
│   │   ├── ai-service/
│   │   └── collab/
│   └── docker/
│       ├── compiler/           # TexLive Dockerfile
│       └── dev/                # docker-compose for local dev
│
├── scripts/                    # Dev tooling, seed scripts
├── turbo.json                  # Turborepo config
├── pnpm-workspace.yaml
└── package.json
Tech Stack Choices (Lean & Fast)
Layer	Choice	Why
Frontend	SvelteKit	Smaller bundle than Next, faster runtime
Editor	CodeMirror 6	Industry standard, extensible
PDF Viewer	PDF.js	Standard, works well
API	Hono.js on Bun	Fastest JS HTTP server, minimal overhead
Realtime/CRDT	Yjs + Hocuspocus	Best CRDT, proven in production
Database	PostgreSQL + pgvector	One DB for relational + vector search
Queue	BullMQ + Redis	Battle-tested job queue for compile jobs
LaTeX	TexLive + latexmk	Standard, incremental compile support
AI	Vercel AI SDK	Model-agnostic, streaming built-in
ORM	Drizzle	Lightweight, type-safe, fast
Auth	Better-auth	Modern, not bloated
Infra	Fly.io → K8s	Start on Fly.io, migrate when needed
File Storage	Cloudflare R2	Cheapest S3-compatible, no egress fees
Search	pgvector initially, Qdrant later	pgvector handles 100k users fine
Scaling Strategy: 10k → 100k → Millions
Phase 1 (0–10k users): Fly.io

Single Postgres + Redis on Fly.io
Compiler service as a separate Fly app with 3–5 always-on machines
Everything in the monorepo, deploy via GitHub Actions
Cost: ~$300–600/month
Phase 2 (10k–100k users): Managed K8s

Move to EKS or GKE
Compiler pods with HPA (autoscale on queue depth)
Read replicas for Postgres
Separate Qdrant for vector search
CDN (Cloudflare) in front of everything
Cost: ~$3k–8k/month
Phase 3 (100k–1M): Full distributed

Compiler service becomes regionally distributed (EU, US, Asia)
Postgres → Citus or Neon for horizontal scaling
Separate billing/auth microservices
Cost: negotiate with cloud providers
The Compiler Service in Detail (Most Critical Piece)
User saves file
      ↓
WebSocket message → compile job enqueued (BullMQ)
      ↓
Worker picks up job → assigns to warm TexLive container
      ↓
latexmk runs incrementally (reuses .aux, .toc cache)
      ↓
Compilation logs stream via WebSocket in real-time
      ↓
PDF written to R2/temp storage
      ↓
Frontend receives "compile_complete" event → loads PDF
Key optimizations:

Keep a pool of 5–10 warm TexLive containers always running (no cold start)
Cache .aux files in Redis keyed by projectId + fileHash
Use latexmk with -interaction=nonstopmode for non-blocking errors
Isolate each compile in a gVisor sandbox or Firecracker microVM for security (users can run arbitrary LaTeX which can make network calls or read files)
AI Agentic Features (Your Moat)
Start with these in order of value/effort:

Research → LaTeX section writer — select notes, AI writes a section draft in LaTeX
Citation agent — paste DOI, arXiv URL, or even just paste raw text of a reference → get clean BibTeX
Semantic search over your entire workspace (pgvector)
Figure captioning — upload figure, get suggested caption + label
AI reviewer — "review this paper for clarity/logic/citations"
Outline agent — dump all your notes → AI proposes a paper outline → you approve → scaffolds LaTeX structure
Diff-aware editing — AI edits specific sections without touching the rest (via LaTeX AST, not raw string replacement)
The AST manipulation point is huge — most AI tools do dumb string replacement which breaks LaTeX. Parse to an AST first, edit nodes, serialize back.

Bottom Line
One monorepo, 5 deployable apps (web, api, compiler-service, ai-service, collab-service), shared packages for types/schema/utils. Start on Fly.io, move to K8s at 10k users. The compiler service with warm containers and incremental compilation is your primary technical differentiator over Overleaf. The AI agentic layer on top of a research workspace is your product differentiator. Keep the frontend lean with SvelteKit and Hono for the API — avoid Next.js API routes and Express bloat.

Why is Overlive so slow and bloated? How do I make the overall UI interaction fast? Compilation can be slow, slow in the sense comparison can be comparable to Overlive time. But the UI should feel extremely snappy, nice smooth animations and everything. Like UI should feel snappy. Why is Overlive slow?

12:13 AM
Why Overleaf Feels Slow
The core problem: Overleaf was built in 2012 and has been patched ever since.

1. Angular.js (the original sin)
Overleaf's frontend was built with AngularJS (v1), one of the heaviest frameworks ever made. They've been partially migrating to React but it's a hybrid mess — you're loading both frameworks simultaneously in some parts. The dirty-checking digest cycle in Angular is legendarily slow for any UI with frequent updates.

2. Monolithic bundle, no real code splitting
Their JS bundle is enormous. Everything loads upfront. When you open a project you're downloading and parsing megabytes of JavaScript before anything renders. Parse time alone on a mid-range laptop is 2–4 seconds.

3. The editor is not optimized
They use CodeMirror 5, not 6. CM5 does full DOM re-renders on changes. CM6 has a much more surgical update model — it only touches the DOM nodes that actually changed. This alone makes typing feel faster.

4. Compilation feedback is terrible UX
The spinner just spins. You don't know if it's queued, compiling, in what phase. So even if compilation takes the same time, it feels longer because there's no feedback. Dead silence + spinner = perceived slowness.

5. PDF re-render is jarring
When a new PDF arrives, Overleaf re-renders the whole PDF view. There's a flash, a scroll jump, and a delay. Users lose their position. This is a UX failure, not a performance one, but it makes the whole thing feel slow.

6. No optimistic UI anywhere
Every action — renaming a file, creating a folder, uploading — waits for a server round trip before the UI updates. There's no optimistic update pattern used.

7. The collaboration layer adds overhead
Their OT (Operational Transformation) implementation adds latency on every keystroke sync because it's older OT, not a modern CRDT. Every character you type goes server → OT resolution → broadcast. With CRDT (Yjs) the local state updates instantly and sync happens in the background.

How to Make Your UI Feel Snappy
This is the most important distinction: perceived performance ≠ actual performance. You can win on feel even if compilation time is identical.

Principle 1: Never block the UI thread
Everything async, everything off the main thread. Use Web Workers for:

LaTeX syntax highlighting (parsing is expensive)
PDF.js rendering (it supports off-screen canvas)
Search/indexing of the workspace
Diffing changes before sending to collab service
javascript
// PDF rendering in a worker — don't block typing
const pdfWorker = new Worker('/pdf.worker.js')
// PDF.js has native worker support, use it
```

### Principle 2: Optimistic UI everywhere

Never wait for the server. Update locally, sync in background, rollback only on failure.
```
User renames file "chapter1.tex" → "intro.tex"
  ↓
UI updates INSTANTLY (optimistic)
  ↓
API call fires in background
  ↓
On success: nothing changes (already correct)
  ↓
On failure: rollback + show subtle error toast
```

This applies to: file rename, create folder, delete file, toggle settings, save preferences — everything that isn't compilation.

### Principle 3: Skeleton screens, never spinners

Spinners communicate "something is happening, you don't know what." Skeletons communicate "content is loading here, in this shape." It feels faster because the user's brain already maps the layout.

On project open — show the file tree skeleton, editor skeleton, and PDF skeleton immediately. Fill them in as data arrives.

### Principle 4: CodeMirror 6 configured correctly

CM6 is fast but you can make it faster:
- Use `EditorView.updateListener` sparingly — every listener adds overhead
- Debounce anything that isn't typing (syntax validation, word count, etc.)
- Use `Decoration.set` with `{sort: true}` for large decoration sets
- Lazy-load language extensions (LaTeX grammar) — don't include in initial bundle
- Use `compartment` for toggling features without full editor recreation

### Principle 5: PDF viewer — no flash, no scroll jump

This is the biggest UX win over Overleaf:
```
Compile finishes → new PDF ready
  ↓
Render new PDF off-screen (in worker)
  ↓
Record current scroll position and zoom
  ↓
Swap in new PDF at exact same scroll position
  ↓
Cross-fade (100ms opacity transition)
```

The user sees content update in place with a gentle fade. No flash, no jump. This single change makes your product feel dramatically more polished than Overleaf.

### Principle 6: Streaming compile logs

Don't show "Compiling..." and then dump the log at the end. Stream it line by line via WebSocket. The user sees progress in real time. Even if compilation takes 15 seconds, it *feels* fast because something is always happening.

Add phase indicators:
```
[●●○○○] Processing bibliography...
[●●●○○] Running pdflatex pass 2...
[●●●●○] Finalizing...
[●●●●●] Done — 14.2s
```

This is pure perceived performance. Compilation time is identical but the experience is completely different.

### Principle 7: Instant project load with SSR + streaming

Use SvelteKit's streaming SSR. The file tree, project metadata, and editor scaffold render on the server and stream to the client. The user sees content before any JS executes.
```
Request hits server
  ↓
Stream HTML: shell + file tree + editor scaffold (server-rendered)
  ↓ (50ms)
Browser paints first content
  ↓ (200ms)
JS hydrates — editor becomes interactive
  ↓ (400ms)
Last-compiled PDF loads
  ↓ (600ms)
Fully interactive
Overleaf: blank screen for 2–4 seconds, then everything at once.

Principle 8: Virtual scrolling everywhere
File trees with hundreds of files, long LaTeX documents, compile logs with thousands of lines — all need virtual scrolling. Only render what's in the viewport.

Use svelte-virtual-list or build a simple one. Never render 10,000 log lines into the DOM.

Principle 9: Font and asset loading
Fonts are a silent killer. Use font-display: swap and preload your editor font (JetBrains Mono or similar) in the <head>. Nothing looks worse than the editor rendering in Times New Roman for 500ms before the monospace font loads.

html
<link rel="preload" href="/fonts/jetbrains-mono.woff2" as="font" crossorigin>
Principle 10: Animations that communicate state, not decorate
Don't add animations because they look nice. Add them because they communicate what happened:

File created → slide in from top (shows where it appeared)
Compile started → subtle pulse on the compile button (shows it's working)
AI agent writing → typewriter effect in the panel (shows it's generating)
PDF updated → cross-fade (shows it changed without jarring the user)
All animations: 100–200ms max. Anything longer feels laggy, not smooth. Use transform and opacity only — these are GPU-composited and never trigger layout.

css
/* Good — GPU composited */
.file-item {
  transition: transform 150ms ease-out, opacity 150ms ease-out;
}

/* Bad — triggers layout recalculation */
.file-item {
  transition: height 150ms, margin 150ms;
}
The Stack Choice that Changes Everything
Use Bun as your runtime. Bun's HTTP server benchmarks at 3–5x faster than Node/Express for simple request/response. Your API feels faster not because you optimized your code but because the runtime is faster.

Use Hono for routing. Hono on Bun is the fastest JS web framework available. For your BFF (backend for frontend) layer, this means API calls from the editor return in <5ms instead of 15–30ms. Over the course of a user session that's hundreds of interactions — it compounds.

Summary: What Beats Overleaf on Feel
Problem in Overleaf	Your Solution
Heavy Angular+React hybrid	SvelteKit — tiny runtime, surgical DOM updates
CodeMirror 5	CodeMirror 6 — workers, surgical updates
Blank screen on load	SSR streaming — content in <100ms
Full PDF re-render flash	Off-screen render + cross-fade swap
Spinner with no feedback	Streaming logs + phase progress bar
Optimistic UI nowhere	Optimistic UI everywhere
Blocking UI thread	Web Workers for everything heavy
Decorative animations	State-communicating GPU animations only
No perceived progress	Skeleton screens on every load
The compilation can take the same 10–15 seconds. The user just won't feel it the same way because everything around it is giving them feedback, staying responsive, and never making them wait for things that don't need a round trip.

What should we use next is our SVELT kit. point.

12:15 AM
SvelteKit is the right call. Here's the full reasoning and what to actually do with it.

Why SvelteKit Wins for This Product
Svelte compiles away the framework. React and Vue ship a runtime that lives in the browser and does virtual DOM diffing at runtime. Svelte does that work at build time — what ships to the browser is essentially vanilla JS that surgically updates the DOM. For an editor product where you have constant small updates (cursor position, syntax highlighting, compile status, collaboration cursors), this matters enormously. There's no reconciler running on every keystroke.

Bundle size comparison for a comparable app: React (Next.js) ships ~130KB of framework JS before your code. Svelte ships ~2KB of runtime. That's not a rounding error — that's a different category.

SvelteKit specifically gives you:

File-based routing with zero config
Streaming SSR out of the box (+page.server.ts with defer)
Built-in form actions (optimistic UI patterns are natural)
Edge-ready (deploy to Cloudflare Workers, Fly.io, Vercel Edge)
$state, $derived, $effect runes in Svelte 5 — reactive primitives that are faster and more predictable than React hooks
Project Structure for the Web App
apps/web/
├── src/
│   ├── app.html                 # Shell HTML — preload fonts here
│   ├── app.css                  # Global resets, CSS variables only
│   │
│   ├── routes/
│   │   ├── +layout.svelte       # Root layout — auth state, theme
│   │   ├── +layout.server.ts    # Session loading, user data
│   │   │
│   │   ├── (auth)/
│   │   │   ├── login/
│   │   │   └── signup/
│   │   │
│   │   ├── (app)/               # Authenticated app shell
│   │   │   ├── +layout.svelte   # App chrome — sidebar, nav
│   │   │   │
│   │   │   ├── dashboard/       # Project list
│   │   │   │   ├── +page.svelte
│   │   │   │   └── +page.server.ts
│   │   │   │
│   │   │   ├── workspace/[id]/  # Research workspace (notes dump)
│   │   │   │   ├── +page.svelte
│   │   │   │   └── +page.server.ts
│   │   │   │
│   │   │   └── editor/[projectId]/   # The LaTeX editor
│   │   │       ├── +page.svelte      # Editor shell
│   │   │       ├── +page.server.ts   # Load project, files, last PDF
│   │   │       └── +layout.svelte    # Editor-specific layout
│   │   │
│   │   └── api/                 # SvelteKit API routes (BFF only)
│   │       ├── compile/+server.ts
│   │       └── ai/+server.ts    # Streaming AI responses
│   │
│   ├── components/
│   │   ├── editor/
│   │   │   ├── CodeMirror.svelte      # CM6 wrapper
│   │   │   ├── FileTree.svelte        # Virtual scrolled file tree
│   │   │   ├── FileTreeItem.svelte
│   │   │   └── EditorTabs.svelte
│   │   │
│   │   ├── pdf/
│   │   │   ├── PDFViewer.svelte       # PDF.js wrapper with cross-fade
│   │   │   ├── PDFPage.svelte
│   │   │   └── SyncTexOverlay.svelte  # Click-to-source overlay
│   │   │
│   │   ├── compiler/
│   │   │   ├── CompileButton.svelte
│   │   │   ├── CompileLog.svelte      # Virtual scrolled, streaming
│   │   │   └── CompileProgress.svelte # Phase indicator
│   │   │
│   │   ├── workspace/
│   │   │   ├── NoteCard.svelte
│   │   │   ├── ResearchDump.svelte
│   │   │   └── FigureCard.svelte
│   │   │
│   │   ├── ai/
│   │   │   ├── AIPanel.svelte
│   │   │   ├── AgentStatus.svelte
│   │   │   └── StreamingText.svelte   # Typewriter effect component
│   │   │
│   │   └── ui/                  # Your design system
│   │       ├── Button.svelte
│   │       ├── Modal.svelte
│   │       ├── Toast.svelte
│   │       ├── Skeleton.svelte
│   │       └── Tooltip.svelte
│   │
│   ├── lib/
│   │   ├── editor/
│   │   │   ├── setup.ts         # CM6 extensions, keymaps
│   │   │   ├── latex-lang.ts    # LaTeX language support
│   │   │   ├── theme.ts         # Editor theme
│   │   │   └── workers/
│   │   │       ├── highlight.worker.ts
│   │   │       └── search.worker.ts
│   │   │
│   │   ├── pdf/
│   │   │   ├── viewer.ts        # PDF.js setup, off-screen rendering
│   │   │   └── synctex.ts       # SyncTeX coordinate mapping
│   │   │
│   │   ├── collab/
│   │   │   ├── yjs.ts           # Yjs doc setup
│   │   │   └── awareness.ts     # Cursor presence
│   │   │
│   │   ├── ws.ts                # WebSocket client (compile events)
│   │   ├── api.ts               # Typed API client
│   │   └── stores/              # Svelte 5 rune-based state
│   │       ├── project.svelte.ts
│   │       ├── compiler.svelte.ts
│   │       └── ai.svelte.ts
│   │
│   └── workers/                 # Vite worker entry points
│       ├── pdf.worker.ts
│       └── highlight.worker.ts
│
├── static/
│   └── fonts/                   # Self-hosted, preloaded
│
├── svelte.config.js
├── vite.config.ts
└── package.json
The Svelte 5 Patterns to Use
Runes for state — not stores for everything. Svelte 5 runes are the right model for this app.

typescript
// stores/compiler.svelte.ts
export const compiler = $state({
  status: 'idle' as 'idle' | 'queued' | 'compiling' | 'done' | 'error',
  phase: '',
  logs: [] as string[],
  pdfUrl: null as string | null,
  duration: null as number | null,
})

// Derived state — no manual subscription management
export const isCompiling = $derived(
  compiler.status === 'queued' || compiler.status === 'compiling'
)
svelte
<!-- CompileButton.svelte -->
<script>
  import { compiler, isCompiling } from '$lib/stores/compiler.svelte.ts'
</script>

<button 
  class="compile-btn"
  class:compiling={isCompiling}
  disabled={isCompiling}
  onclick={triggerCompile}
>
  {isCompiling ? compiler.phase : 'Compile'}
</button>
Streaming SSR for the editor route — critical for fast load.

typescript
// routes/(app)/editor/[projectId]/+page.server.ts
export const load = async ({ params }) => {
  const project = await getProject(params.projectId) // fast — just metadata

  return {
    project,
    // These stream in — don't block initial render
    files: getProjectFiles(params.projectId),      // deferred
    lastPdf: getLastCompiledPdf(params.projectId), // deferred
  }
}
svelte
<!-- +page.svelte -->
<script>
  let { data } = $props()
</script>

<!-- Renders immediately with project metadata -->
<EditorShell project={data.project}>

  <!-- File tree streams in — show skeleton until ready -->
  {#await data.files}
    <FileTreeSkeleton />
  {:then files}
    <FileTree {files} />
  {/await}

  <!-- PDF streams in separately — editor is already interactive -->
  {#await data.lastPdf}
    <PDFSkeleton />
  {:then pdf}
    <PDFViewer {pdf} />
  {/await}

</EditorShell>
The editor is fully interactive before the PDF even loads. The user can start typing immediately.

One Important Decision: Adapter
SvelteKit needs an adapter to deploy. Choose based on where you're hosting:

adapter-node — deploy as a Node/Bun server on Fly.io or K8s. This is the right choice for you because you need WebSocket support, and edge runtimes don't support long-lived connections.
Don't use adapter-vercel or adapter-cloudflare — they're serverless/edge, which means no persistent WebSocket connections for your compile log streaming.
Run SvelteKit on Bun with adapter-node. It's a one-line config change and you get the Bun runtime speed for free.

javascript
// svelte.config.js
import adapter from '@sveltejs/adapter-node'

export default {
  kit: {
    adapter: adapter()
  }
}
dockerfile
# Dockerfile for web app
FROM oven/bun:1
WORKDIR /app
COPY . .
RUN bun install
RUN bun run build
CMD ["bun", "build/index.js"]  # SvelteKit output, run with Bun
That's your frontend foundation. Fast by default, streams content, compiles away the framework overhead, and runs on the fastest JS runtime available.

What should I build first? The client or the back end or both should go hand in hand.

12:17 AM
Build them hand in hand, but frontend-first with mocked backends.

Here's the thinking: your product lives or dies on how it feels. The editor experience, the snappiness, the PDF swap animation, the streaming compile log — these are what users will judge in the first 30 seconds. You need to validate that feel early, not after 3 months of backend work.

But "frontend-first" doesn't mean backend later. It means you build the frontend against mocks, then swap mocks for real services one at a time.

The Sequence
Week 1–2: Frontend Shell, Zero Backend
Build the entire UI with hardcoded/mocked data. No API calls, no database, nothing.

What you're validating: does the layout feel right, does the editor feel snappy, does the split-pane resize smoothly, does the PDF viewer render correctly, do the animations feel good.

SvelteKit project setup, routing structure
CodeMirror 6 integrated with a LaTeX sample file hardcoded
PDF.js integrated with a sample PDF hardcoded
Split pane layout (editor left, PDF right) with drag-to-resize
File tree with hardcoded files
Compile button that fakes a 5 second compile with a progress animation
Streaming log simulation (just setInterval printing fake log lines)
PDF cross-fade swap (swap between two hardcoded PDFs to test the animation)
At the end of week 2 you should be able to sit down with the product and feel it. If it doesn't feel great at this point with zero network latency, it never will.

Week 3–4: Auth + Real File Storage
First real backend piece. This unblocks everything else.

Better-auth for auth (Google OAuth + email/password)
Postgres database, Drizzle schema for users/projects/files
R2 for file storage
Real file tree — create, rename, delete files
Real project creation and loading
Replace hardcoded file content with real content from R2
Now you have a real multi-user app where people can create projects and edit LaTeX files. Still no compilation.

Week 5–6: Compilation Service
The core technical challenge. Do this before AI because AI depends on it.

TexLive Docker container with latexmk
BullMQ + Redis job queue
WebSocket server for streaming logs
The actual compile pipeline: take files from R2, run latexmk, write PDF back to R2
Wire up the compile button to the real service
Remove the fake progress animation, replace with real streaming logs
At the end of week 6 you have a working LaTeX editor that compiles. This is your MVP. It's not better than Overleaf yet but it exists and works.

Week 7–8: Collaboration + Polish
Yjs + Hocuspocus for real-time collaboration
SyncTeX (click PDF → jump to source)
Auto-compile on save option
Error highlighting in editor from compile logs
The research workspace (notes dump area)
Week 9+: AI Features
Now you build the moat. By this point the foundation is solid, you know the codebase, and you're adding AI on top of something that already works.

Embeddings for workspace notes (pgvector)
Citation agent
Section writer agent
Streaming AI responses in the panel
The Rule of Thumb
Every week you should have something you can open in a browser and interact with. Never go more than a week without a visible, interactive result. Backend work that isn't yet connected to the frontend doesn't count — it's invisible and you can't feel whether it's right.

The moment you're tempted to spend two weeks purely on backend infrastructure with no UI to show for it, that's the moment to stop and ask what the frontend needs right now.

Can I make it faster than over lift?

12:17 AM
Yes, absolutely. And not just marginally — you can make it feel like a different category of product. Here's exactly how.

Where You Can Be Genuinely Faster
1. Compilation — Yes, You Can Beat Them Here Too
Overleaf's free tier queues jobs. Users wait in line. Even on paid tiers, cold container startup adds latency. You can beat this with one architectural decision: warm container pools.

Overleaf:
User clicks compile → find available server → 
cold start TexLive container (2–8s) → compile → done

You:
User clicks compile → assign to already-warm container → 
compile → done
Keep 5–10 TexLive containers always running, always warm. The moment a user clicks compile, a container is ready. You eliminate the cold start entirely. For a project with cached aux files, you can hit 2–4 second compilations for incremental changes vs Overleaf's 10–30 seconds on free tier.

The other trick is smarter incremental compilation. Overleaf recompiles the whole document every time. You cache aggressively:

User edits chapter3.tex only
  ↓
Detect which files changed (hash comparison)
  ↓
latexmk figures out only chapter3 needs reprocessing
  ↓
.aux, .toc, .bbl from last compile are reused
  ↓
Only 1 pdflatex pass needed instead of 3
This is latexmk's native capability but you have to architect your file caching around it correctly. Overleaf doesn't do this well.

2. Frontend — This Is Where You Win Most Visibly
Overleaf's frontend stack is genuinely, measurably slow. These are real numbers you can beat:

Time to Interactive on project open:

Overleaf: 3–6 seconds on a decent connection
You (with SSR streaming + Svelte): 400–800ms
Keystroke to screen latency:

Overleaf (CM5 + Angular): 16–32ms
You (CM6 + Svelte): 4–8ms
Users can't consciously measure these numbers but they absolutely feel them. The editor feeling "sticky" vs "instant" is entirely this difference.

3. The Network Layer
Overleaf's API is Node.js + Express, which is fine but not fast. You on Bun + Hono will handle the same requests in roughly a third of the time. For an editor product this matters because you're making constant small API calls — file saves, presence updates, settings changes.

More importantly: use binary protocols where it matters. Collaboration sync (Yjs) should go over WebSocket with binary encoding, not JSON. Yjs does this natively. The collab messages are tiny and fast.

The Specific Things to Build That Overleaf Simply Doesn't Have
Differential PDF Rendering
When a new PDF arrives after compilation, don't re-render the whole thing. Detect which pages actually changed and only re-render those. The user sees the updated pages fade in while unchanged pages stay perfectly stable.

typescript
async function swapPDF(oldPdf: PDFDocument, newPdf: PDFDocument) {
  const changedPages = await detectChangedPages(oldPdf, newPdf)
  
  // Unchanged pages: do nothing, they're already rendered
  // Changed pages only: render off-screen, cross-fade in
  for (const pageNum of changedPages) {
    const newPage = await renderPageOffscreen(newPdf, pageNum)
    await crossFade(currentPages[pageNum], newPage, 120)
  }
}
Overleaf blanks the whole PDF and re-renders everything. Yours updates surgically.

Predictive Compilation
Start compiling before the user clicks the button. When the user stops typing for 800ms, spin up the compilation in the background silently. When they click compile, it's already done or nearly done.

typescript
let compileTimeout: Timer

editorView.on('change', () => {
  clearTimeout(compileTimeout)
  
  compileTimeout = setTimeout(() => {
    // Start compiling silently, don't show UI yet
    backgroundCompile().then(result => {
      // Cache the result
      pendingResult = result
    })
  }, 800)
})

function onCompileButtonClick() {
  if (pendingResult) {
    // Already done — show instantly
    displayResult(pendingResult)
    pendingResult = null
  } else {
    // Fall back to normal compile with UI feedback
    foregroundCompile()
  }
}
```

This is the same trick Google uses with search predictions. The computation happens before the user asks for it.

### Local-First Editor State
The editor state should never depend on the network for basic operations. File content, cursor position, unsaved changes — all local, synced to the server in the background. If the network drops, the user keeps working. When it reconnects, state reconciles via Yjs CRDT automatically.

Overleaf loses your work if the connection drops. Yours doesn't. That's not just a performance win, it's a reliability win that users will remember.

### Edge Deployment for the Frontend
Deploy your SvelteKit frontend to Cloudflare's edge network — 300+ locations worldwide. The HTML and assets are served from a server that's physically close to the user. Overleaf serves from a small number of data centers.
```
User in Singapore hitting Overleaf (US servers):
  DNS → 200ms → TCP → 200ms → HTML → 200ms → assets
  Total: 600ms+ before anything renders

User in Singapore hitting your app (Cloudflare edge):
  DNS → 5ms → TCP → 5ms → HTML → 5ms → assets
  Total: ~50ms before content renders
The compiler service still needs to be in a real data center (can't run TexLive on edge workers) but the frontend and static assets can be globally distributed.

The Honest Ceiling
There's one place you can't beat them structurally right now: TexLive itself. The actual LaTeX engine — pdflatex, xelatex, biber — is the same software. A complex 50-page document with lots of packages will take roughly the same time to compile on your infrastructure as on theirs, hardware being equal. The engine is the engine.

What you can do is make everything around the engine faster. The queue, the container startup, the incremental caching, the PDF delivery, the UI response — all of that is yours to optimize and none of it is limited by TexLive.

Summary: What Wins
Area	Overleaf	You
Time to interactive	3–6s	<1s with SSR streaming
Cold compile start	2–8s queue + startup	~0s warm containers
Incremental compile	Full recompile	True incremental with aux cache
PDF update	Full re-render + flash	Differential page swap + crossfade
Keystroke latency	16–32ms	4–8ms
Network drops	Lose work	Keep working, CRDT reconciles
Global latency	Few data centers	Edge-deployed frontend
Predictive compile	None	Background compile on pause
You can be faster. The architecture described here isn't theoretical — every piece of it is proven technology used in production by other fast products. You're just combining them in a way Overleaf, constrained by a 12-year-old codebase, physically cannot do without a full rewrite.

think about how to add the AI features. This is what I am thinking right now. The researcher just drag and drop files to some projects. So, researchers can create projects within that projects they can just drag and drop bunch of files. Files can be either nodes, initial drafts, some recordings of their thought, it can be figures, it can be some small data and files can be bunch of things. They can dump all of these things and there are AI to make sense of them. There are AI that can analyze those images. There are AI which can draft the initial things. So, we need to structure the project, give structure to a project for the user so that they can just start using it properly. We give them structure. This is how we should structure, ideally structure the project. Paper should go like this. We you give them template and everything. Is that possible?

12:20 AM
Yes, this is not only possible — this is the actual moat. This is what makes you not just "faster Overleaf" but a completely different product category. Let me think through this properly.

The Core Concept: Intelligence Layer on Top of Chaos
The researcher's mental model is:

I have a bunch of stuff → I want a paper
Everything in between — organizing, structuring, drafting, formatting — is friction. Your product eliminates that friction. The AI's job is to be the bridge between "dump of stuff" and "structured paper."

What "Files" Actually Are (You Need to Handle All of These)
When a researcher drags and drops, they're throwing in:

Text-based:

PDF papers they're citing or building on
Word docs with rough notes
Markdown notes
Plain text brain dumps
Email threads with collaborators
Meeting notes
Their own previous papers
Audio/Video:

Voice memos recorded while thinking in the shower
Lecture recordings
Conference talk recordings
Zoom call recordings with collaborators
Visual:

Figures and plots (PNG, SVG, PDF)
Handwritten notes (photos)
Whiteboard photos
Screenshots of papers or equations
Raw data visualizations
Data:

CSV/Excel datasets
Jupyter notebooks
R scripts with output
MATLAB files
Structured:

BibTeX files
Reference manager exports (Zotero, Mendeley)
Existing LaTeX files
Each of these needs a different processing pipeline but they all feed into the same knowledge graph.

The Processing Pipeline Per File Type
File dropped
     ↓
Type detection (MIME type + content sniffing)
     ↓
┌────────────────────────────────────────┐
│           Extraction Layer             │
│                                        │
│  PDF → extract text + figures          │
│  Audio → Whisper transcription         │
│  Image → vision model analysis         │
│  Handwritten → OCR + vision            │
│  Notebook → execute + capture output   │
│  Data → statistical summary + charts   │
│  Word/MD → clean text extraction       │
└────────────────────────────────────────┘
     ↓
Chunking + Embedding (pgvector)
     ↓
Entity extraction (methods, results, 
claims, citations, figures, equations)
     ↓
Added to project knowledge graph
     ↓
AI analyzes new file in context of 
everything already in the project
     ↓
Surfaces insights to user
The Intelligence Layer: What the AI Actually Does
1. Project Understanding Agent
Every time a new file is added, this agent re-evaluates the whole project and updates its understanding.

"Based on your files, this appears to be a paper about 
transformer attention mechanisms applied to protein folding. 
I've identified:

→ 3 datasets you're working with
→ 2 baseline methods you're comparing against  
→ 1 proposed method (from your notes and notebook)
→ 47 papers in your reference pool
→ 8 figures ready to use
→ 4 recordings with unextracted insights

Your main argument seems to be: [extracted claim]
Your key result seems to be: [extracted from notebook output]"
This runs automatically. The researcher doesn't ask for it.

2. Structure Proposal Agent
This is your killer feature. After analyzing all files, the AI proposes a paper structure.

typescript
interface PaperStructure {
  type: 'conference' | 'journal' | 'workshop' | 'thesis'
  template: string  // e.g. 'neurips2024', 'ieee', 'acl', 'nature'
  sections: Section[]
}

interface Section {
  name: string
  status: 'ready' | 'partial' | 'missing'
  availableContent: ContentSource[]  // which files feed this section
  suggestedOutline: string[]
  estimatedLength: string
}
```

What the user sees:
```
┌─────────────────────────────────────────────┐
│  Suggested Structure: NeurIPS 2024 Format   │
│                                             │
│  ✅ Abstract       → can draft now          │
│  ✅ Introduction   → 3 notes + 2 papers     │
│  ✅ Related Work   → 47 refs identified     │
│  ⚠️  Methodology   → partial (voice memo)   │
│  ✅ Experiments    → notebook has results   │
│  ⚠️  Discussion    → needs your input       │
│  ✅ Conclusion     → can draft from above   │
│  ✅ References     → 47 papers ready        │
│                                             │
│  [Use This Structure]  [Modify]  [Different Template]  │
└─────────────────────────────────────────────┘
```

User clicks "Use This Structure" → SvelteKit creates the LaTeX scaffold instantly.

### 3. Section Drafting Agent
User clicks on any section → AI drafts it from the relevant source material.
```
User clicks "Draft Introduction"

Agent:
1. Pulls all content tagged as relevant to Introduction
   - Their brain dump notes
   - Key papers they uploaded
   - Relevant voice memo transcripts
   - Their stated main claim

2. Identifies the narrative arc:
   - What's the problem?
   - Why does it matter?
   - What have others done?
   - What's your contribution?

3. Writes LaTeX directly into the editor
   with proper citations \cite{} already filled in
```

This isn't "AI writes your paper." It's "AI drafts from YOUR material." Every sentence is grounded in something the researcher actually uploaded. That's the crucial distinction — it's synthesis, not hallucination.

### 4. Figure Intelligence Agent
Researcher uploads a PNG of a plot. The agent:
```
Analyzes the figure visually
  ↓
"This appears to be a line chart comparing 
accuracy vs training steps for 3 models.
Your model (blue) outperforms baselines after step 5000."
  ↓
Suggests caption: "Figure 3: Accuracy curves..."
  ↓
Suggests where in the paper this figure belongs
  ↓
Generates LaTeX \begin{figure} block with caption + label
  ↓
Cross-references it in the relevant section text
```

### 5. Voice Memo Agent
This one is underrated. Researchers have their best ideas while walking, driving, showering. They record a rambling 8-minute voice note. Your agent:
```
Whisper transcription (fast, local or API)
  ↓
"This recording contains:
 - A critique of the baseline approach (2:14)
 - An idea for an ablation study (4:30)  
 - A concern about reviewer objections (6:45)
 
Key insight extracted: [summary]
Suggested section: Discussion / Limitations"
```

### 6. Citation Agent
User uploads a PDF of a paper, or pastes a DOI, or even just pastes the abstract:
```
Extract metadata (title, authors, year, venue, DOI)
  ↓
Generate BibTeX entry
  ↓
Analyze: how does this paper relate to yours?
  ↓
"This paper is cited as a baseline — 
 you should compare against Table 2"
 
"This paper supports your claim in section 3 —
 I've added \cite{} in the relevant location"
```

### 7. Consistency Agent (Background, Always Running)
This runs silently and flags issues:
```
→ "Figure 4 is referenced in section 3 but 
   the figure shows data inconsistent with 
   your claim on line 47"

→ "You mention 'our dataset has 10,000 samples' 
   in the introduction but your notebook shows 9,847"

→ "Related work doesn't cite Smith et al. 2023 
   which you uploaded and directly compares 
   against your method"
```

---

## The UI for All of This

### The Workspace View (Before LaTeX Editor)
```
┌─────────────────────────────────────────────────────┐
│  Project: Protein Folding Attention                 │
│                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐          │
│  │  Files   │  │ AI View  │  │  Draft   │          │
│  └──────────┘  └──────────┘  └──────────┘          │
│                                                     │
│  Drop Zone                                          │
│  ┌─────────────────────────────────────────────┐   │
│  │                                             │   │
│  │     Drag files here or click to upload      │   │
│  │                                             │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  Processing                                         │
│  ├── 📄 attention_paper.pdf      ✅ Analyzed        │
│  ├── 🎙️  idea_voice_note.m4a     ✅ Transcribed     │
│  ├── 📊 results_figure.png       ✅ Analyzed        │
│  ├── 📓 experiments.ipynb        ⏳ Processing      │
│  └── 📝 rough_notes.md           ✅ Indexed         │
│                                                     │
│  AI Understanding                                   │
│  ┌─────────────────────────────────────────────┐   │
│  │ "This looks like a paper about X. I've      │   │
│  │  identified your main contribution and       │   │
│  │  found 3 gaps in your current material."    │   │
│  │                                             │   │
│  │  [See Full Analysis]  [Structure Paper →]   │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
The Structure Proposal (The Magic Moment)
This is the moment the product earns its keep. The AI says "here's your paper" and shows a visual structure with traffic lights — what's ready, what's partial, what's missing. The researcher feels like they're 40% done before writing a single word.

The Editor View (After Structure Accepted)
The LaTeX editor opens with the scaffold already there. Each section has a comment block:

latex
\section{Introduction}
% AI: Ready to draft from your notes and voice memo
% Sources: rough_notes.md (lines 1-45), idea_voice_note.m4a (2:14-3:30)
% [Click to draft this section]

\section{Related Work}  
% AI: 47 references identified. Ready to draft.
% [Click to draft this section]
```

The researcher can draft section by section, accept/reject/edit AI output, and the LaTeX is always valid and compilable.

---

## Technical Architecture for the AI Layer
```
apps/ai-service/
├── src/
│   ├── ingestion/
│   │   ├── pdf.ts           # PDF text + figure extraction
│   │   ├── audio.ts         # Whisper transcription
│   │   ├── vision.ts        # Image/figure analysis (GPT-4V)
│   │   ├── notebook.ts      # Jupyter parsing + output capture
│   │   ├── handwriting.ts   # OCR for photos of notes
│   │   └── dispatcher.ts    # Routes files to correct pipeline
│   │
│   ├── agents/
│   │   ├── understanding.ts  # Project-level analysis
│   │   ├── structure.ts      # Paper structure proposal
│   │   ├── drafter.ts        # Section drafting
│   │   ├── citation.ts       # Reference management
│   │   ├── figure.ts         # Figure analysis + captioning
│   │   ├── voice.ts          # Voice memo insight extraction
│   │   └── consistency.ts    # Background consistency checker
│   │
│   ├── rag/
│   │   ├── embeddings.ts     # Text → vectors
│   │   ├── retrieval.ts      # Semantic search
│   │   └── graph.ts          # Knowledge graph (entities + relations)
│   │
│   ├── latex/
│   │   ├── templates/        # NeurIPS, IEEE, Nature, ACL etc.
│   │   ├── scaffold.ts       # Generate LaTeX structure
│   │   └── ast.ts            # AST manipulation for AI edits
│   │
│   └── queue/
│       └── jobs.ts           # BullMQ jobs for async processing
```

---

## The User Flow End to End
```
1. Create project
2. Drop all files (5 seconds of work)
3. Watch AI process them (grab coffee)
4. Read AI's understanding — correct if wrong
5. Accept paper structure (or modify)
6. Template selected → LaTeX scaffold created
7. Click into sections → AI drafts from your material
8. Edit, refine, compile, iterate
9. Export final PDF
Steps 1–6 currently take researchers days or weeks of organizing. You compress it to minutes. That's the product.

The researchers aren't being replaced — they're being unblocked. Every word in the final paper came from their material, their ideas, their experiments. The AI just found it, organized it, and gave it shape.