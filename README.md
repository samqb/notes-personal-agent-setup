# Building an AI-Native Second Brain

A single-file static site documenting a personal agentic knowledge system: an Obsidian
vault as the single source of truth, a desktop AI (Claude Cowork) that runs the daily
automations, and an optional always-on agent reachable from a phone.

Everything is genericised and redacted — no real names, clients, credentials, or
home-network detail. The value here is the **pattern and the thinking**, not anyone's
live setup.

## Contents

- `index.html` — the entire site. Inline CSS and JS, no build step. Google Fonts is the
  only external request.
- `GUIDE.md` — the same content as plain Markdown (renders in Obsidian / on GitHub).

## Run locally

Open `index.html` in any browser. That's it.

## Deploy

Deploys to Vercel (or any static host) with zero config — point it at this folder:

```bash
vercel deploy
```

## Sections

Overview · The Vault · Automations · Always-On Agent · Working Patterns · The Dashboard ·
Replicate This · Where It's Going.
