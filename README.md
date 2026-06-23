# 🏗️ Sidekick Sandbox Hackathon

**Theme: sandboxing untrusted AI agents at scale — and wiring them into Slack without melting the rate limits.**

A focused build sprint. Pick a challenge, hack on it with your AI agent of choice
(Claude Code, Cursor, Codex — whatever you fly), and **show us what you ship**. Design it,
prototype it, or both — depth beats breadth.

This is a sprint, not a marathon: **budget ~half a day.** We care as much about *how you
drive and verify your AI agent* as about the artifact.

---

## The mission

We run autonomous AI agents on behalf of users. Each user gets a long-lived **room** — a
sandbox on a shared host where an agent runs continuously: it watches Slack, reads repos,
and takes real actions (posts to Slack, pushes to Git, deploys to hosts). To do its job it
holds **real credentials** (Slack bot token, Git token, sometimes cloud access).

Two things make it hard:

1. **Untrusted inputs.** Slack messages, repo contents, web pages — all can carry prompt
   injection. So treat each agent's *behavior* as potentially adversarial, even when it's
   acting for a legitimate user.
2. **Scale & multi-tenancy.** We want **~300 rooms on one host**, for users who don't trust
   each other.

Today: rooms are plain containers, agents hold their tokens directly, and every room polls
Slack on its own. That doesn't survive 300 tenants. Your job is to rethink it.

---

## The challenges

Grab **one** and go deep, or take a swing at both. You don't have to cover everything in a
challenge — nail the part you find juiciest and note what you'd defer.

### 🔒 Challenge 1 — Isolate untrusted agents at scale
Run ~300 mutually-untrusting agents on one host without them stepping on each other (or the
host). Hack on:
- **The boundary.** Is a container enough for untrusted execution? Threat-model it. Where do
  you land — hardened containers, micro-VMs, something else — and what does it cost you in
  density at 300/host?
- **Credentials.** Agents currently hold long-lived tokens in their own env. Acceptable at
  this trust level? If not, what replaces it?
- **Egress.** Stop an agent from reaching arbitrary destinations — and especially the
  host's cloud metadata endpoint. Make it *structural*, not advisory.
- **Blast radius.** One room gets fully owned. What can it touch, and how do you bound it?

### ⚡ Challenge 2 — Slack at 300 rooms, under rate limits
Per-room polling dies at scale (Slack's limits are per token per workspace). Decouple cost
from room count and build:
1. **Mention-aware wake** — wake a room's agent only when its owner *or* the room's bot is
   `@`-mentioned, across many channels. Not on every message.
2. **Channel-change wake** — a watched channel changes → notify the rooms that care.
3. **Shared Slack corpus** — a workspace-wide, **normalized, read-only** Slack dataset all
   rooms can read (history, threads, who-said-what) *without each room hitting the API.*

Don't dodge the hard parts: central ingestion under one rate budget, the normalized schema
+ storage + query path, freshness vs. consistency for a live shared dataset, and — since
the corpus is *shared* — **how tenant A never reads tenant B's private channels.**

---

## What to ship

Push it to a repo (fork this, or your own — link it back to us):

- **The build** — `src/` if you wrote code (with a one-liner on how to run it), and/or a
  **`DESIGN.md`** with your architecture, diagrams (ASCII / mermaid / images all fine), and
  the tradeoffs you weighed. A design-only ship is totally valid.
- **`DEMO.md`** — 5 lines or a short clip: what you built and what it proves.
- **`BUILD-LOG.md`** — short. How you drove your AI agent: what you delegated, where it was
  wrong and how you caught it, and what calls you made yourself.

---

## How we judge

- **Threat-model first.** Name what you're protecting and from whom before reaching for a
  tool.
- **Isolation chops.** Real depth on container-vs-micro-VM and the density/cost reality at
  300/host.
- **Distributed-systems judgment.** Central ingestion, fan-out, caching, freshness, and the
  shared-corpus tenant-isolation puzzle.
- **AI fluency.** Do you steer the agent and verify it, or does it steer you? We read
  `BUILD-LOG.md` closely.
- **Taste & prioritization.** What you chose to build vs. defer, and why.

No hidden "right" architecture. Sharp, well-argued tradeoffs win the day.

---

*Have fun with it. Ship something you'd want to demo.* 🚀
