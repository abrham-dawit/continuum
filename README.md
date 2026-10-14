# Continuum

**A real-time knowledge graph of what your whole team is working on — captured as you work, connected automatically.**

Continuum watches work happen (with a privacy filter in the loop), keeps a persistent AI agent for every person, and draws the connections between teammates' work as they form — what builds on what, what contradicts what, and who is working on the same thing without knowing it.

## What it does

- **Ambient capture (desktop)** — an Electron app samples your screen on an interval, reads it with a Python vision sidecar (Moondream, with a local fallback), and runs everything through a privacy filter before anything leaves the device.
- **A Letta agent per person** — every user gets a persistent Letta agent that turns filtered observations into durable knowledge nodes.
- **Cross-person connection detection** — a scheduled job finds high-similarity node pairs across teammates, classifies the relationship with Claude Haiku, and writes the edge: `RELATED_TO`, `BUILDS_ON`, or `CONTRADICTS`.
- **Live graph dashboard (web)** — a Next.js app renders the shared graph as a D3 force layout that updates over SSE as new nodes and edges land.
- **Ask the graph** — ask a question (voice input supported) and it gets embedded, matched against the vector store, expanded over the subgraph, and answered by Claude Sonnet with citations.
- **Fetch.ai agents** — a coordinator + device agent pair in `fetchai/` that handle device-side registration and routing.
- **Vector search two ways** — Redis Stack (VSS) for search, with a pgvector fallback (`match_nodes` / `find_connection_candidates` RPCs) when Redis Search isn't available.
- **Tested where it counts** — the desktop capture/privacy/sync modules and the Supabase ingest/embedding helpers ship with their own test suites.

## Layout

| Path | What it is | Stack |
| :--- | :--- | :--- |
| `web/continuum` | Live graph dashboard + query UI | Next.js 16, React 19, Tailwind CSS, D3, ioredis, Supabase |
| `desktop` | Ambient capture agent | Electron 32, Node 20, Python sidecar (Moondream) |
| `supabase` | Schema, RLS migrations, 11 Edge Functions | Postgres + pgvector, Deno |
| `fetchai` | Coordinator + device agents | Python, Fetch.ai |
| `scripts` | Dev and smoke-test utilities | Node, shell |

## Run it

Each part has its own setup and env example:

```bash
# dashboard
cd web/continuum && cp .env.local.example .env.local && npm install && npm run dev

# desktop agent — see desktop/README.md
# backend — see supabase/README.md
```

Redis Stack (or Redis Cloud) powers vector search; the pgvector RPCs cover the same queries when it isn't available. The full design is written up in `docs/ARCHITECTURE.md`.

## How it flows

- **Observation:** desktop capture → Moondream → privacy filter → your Letta agent → `agent-sync` Edge Function → Postgres/Redis node → SSE broadcast → dashboard.
- **Connection detection:** the `connection-detect` function finds cross-person lookalike nodes, classifies them, and writes the edges.
- **Query:** dashboard → `voice-transcribe` → `query-synthesize` → embed → vector KNN → subgraph expansion → a citation-aware answer.

## Credit

Built by a four-person team: me, [@rathodkunj2005](https://github.com/rathodkunj2005), [@lukeeskinner](https://github.com/lukeeskinner), and [@Zablon-Tesfaye](https://github.com/Zablon-Tesfaye). This is my copy of the project — the original repo is [lukeeskinner/Continuum](https://github.com/lukeeskinner/Continuum).
