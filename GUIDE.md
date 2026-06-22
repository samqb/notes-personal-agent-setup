# Building an AI-Native Second Brain

> A field note on a personal agentic system. One plain-text vault, a desktop AI that runs the
> automations, and an optional always-on agent you can message from your phone. This is the
> markdown companion to `index.html` — same content, genericised and redacted for public sharing.
> The value here is the **pattern and the thinking**, not anyone's live credentials.

**Contents:** [Overview](#01--the-pitch) · [The Vault](#02--the-store) · [Automations](#03--the-engine) · [Always-On Agent](#04--the-always-on-layer) · [Working Patterns](#05--practice) · [The Dashboard](#06--the-surface) · [Replicate This](#07--build-your-own) · [Where It's Going](#08--the-horizon)

---

## 01 — The Pitch

### Five apps, collapsed into one vault

A portfolio career — spanning advisory, fractional leadership, and angel investing across several
work streams — means hundreds of stakeholders, dozens of live threads, and meetings that jump
between completely different contexts. The fix wasn't a better app. It was deleting most of them and
putting everything in one place an AI could actually operate.

**The consolidation.** Before: notes, tasks, and years of context were scattered across five tools —
a chat assistant's project folders, a corporate note app, a personal note app, a dedicated task
manager, and assorted others. Everything got exported and imported into one Obsidian vault, filed
into archive folders. Then the old apps came off the dock and the phone home screen. What's left is
**Obsidian and Claude**.

> **The core idea.** The "app" is the vault plus a few agents that operate on it. There's no database
> and no backend — just plain Markdown that's portable, diffable, greppable, and future-proof. Every
> tool reads and writes the same files.

### What a typical morning looks like

By the time the vault is opened each day — before anything else is touched — it already holds:

| When | Brief | What's in it |
|------|-------|--------------|
| ~05:00 | **Meeting prep** | A brief for every meeting today: who's attending, the last notes with them, open items still in the air. |
| ~05:30 | **Tasks brief** | A smart lens over the single task file — what's flagged, what's gone stale, what's been waiting on someone too long. |
| ~04:00 | **Email gravity** | One ranked view of inbox pressure: who's waiting, what's urgent, what was flagged, what's safe to ignore. |
| ~06:00 | **Night-shift read** | A divergent overnight pass that connects threads across recently-touched notes and names blind spots. |
| ~00:00 | **Meeting sync** | Yesterday's meeting transcripts filed into the right project folder, with every attendee wikilinked. |
| Daily | **One daily note** | A single page that pulls all of the above in by transclusion. Open it and the whole day is there. |

> **Three surfaces, one brain.** The work is split by **job, not by data**. An always-on machine owns
> the scheduled work and backups. A laptop is for interactive sessions. A phone drives the always-on
> machine remotely. All three share the one synced vault.

---

## 02 — The Store

### The vault as second brain

Everything — notes, meetings, people, tasks, project context, even the AI's own memory — lives in one
Obsidian vault of plain Markdown. This is the most shareable, least sensitive layer, and the
foundation everything else assumes.

### The folder skeleton

```
VAULT/
  TASKS.md            single task file, sectioned by ### area
  NEW.md              shared session log — every agent writes here
  DECISIONS.md        rolling decision log + current priorities
  AGENTS.md           pointer file for the always-on agent
  dashboard.html      single-file browser dashboard over the vault
  daily/              one note per day — a "view" layer
  briefs/             generated outputs from scheduled tasks
  agent-drafts/       always-on agent writes drafts here for review
  projects/           one folder per work stream (each with its own meetings/)
  meetings/           only uncategorised + external / expert calls
  people/             one page per person — your CRM
  notes/              everything thematic (tech, ideas, learning, …)
    ideas/            an IDEAS index + dated idea archives
  templates/          note templates
  clippings/          web clipper saves
  attachments/        images, PDFs, email attachments
  _archive/           dated archives + monthly log rollups
  _claude/            agent context + memory (not for human use)
```

### The design decisions that make it work

- **People as pages.** Every person gets a page in `people/` with YAML frontmatter (org, role,
  aliases). Meeting notes wikilink attendees like `[[Jane Smith]]`. Obsidian's native backlinks then
  give you a free CRM — open anyone's page and see every meeting and note they appear in, with zero
  upkeep. This is the highest-leverage convention in the whole system.
- **Dataview for live queries.** Project hubs don't maintain meeting lists by hand. They run queries
  like `LIST FROM "projects/x/meetings" SORT file.name DESC LIMIT 15` that auto-update. People pages
  use `WHERE contains(file.outlinks, this.file.link)` to list every backlink dynamically.
- **Daily notes as a view layer.** Scheduled tasks write standalone files to `briefs/`. The
  daily-note template embeds them by transclusion (`![[meeting-prep]]`) and shows different briefs on
  different weekdays. Tasks stay simple — write one file — and the daily note is just a lens over them.
- **One task file.** No monthly pages, no nested system. A single `TASKS.md` at the root, sectioned
  by `###` area, plus `## Waiting On` and `## Recently Completed`. Flat and in one file is what lets a
  dashboard and the agents reason over it cleanly.
- **Meetings live with their project.** Each project folder has its own `meetings/` subfolder, so a
  work stream's hub, docs, and meetings stay together. Only genuinely uncategorised or external calls
  go in the root `meetings/`.
- **Agent memory is separate.** The `_claude/` folder holds structured context the AI reads at session
  start — people profiles, project context, a glossary. It's deliberately *separate* from the
  human-facing `people/` and `projects/` pages. Don't merge them.

### Two-tier AI memory

Agents don't remember across sessions, so memory is files. A hot cache and the detail it points to:

```
_claude/
  CLAUDE.md            working memory: who you are, setup, conventions (read every session)
  memory/
    context/           preferences, voice & style, the operating conventions
    people/            deeper profiles by domain (beyond the CRM pages)
    projects/          per-project context + the canonical scheduled-task list
    glossary.md        acronyms, nicknames, codenames the agent must decode
```

`CLAUDE.md` is kept tight — a working note, not an essay: who you are, your areas, your setup, your
preferences, and pointers into `memory/`. The glossary is what lets the AI understand shorthand like a
colleague would.

### Plugins doing the heavy lifting

**Dataview** (live queries) · **Templater** (daily-note template) · **Local REST API** (vault access
for tools) · **Smart Connections** (semantic search) · **Web Clipper** (article capture) ·
**Obsidian Sync** (cross-device).

---

## 03 — The Engine

### Automations that run while you sleep

Claude Cowork is a desktop agent that can read and write the vault folder and run tasks on a schedule.
It runs on the always-on machine. Each scheduled task writes one standalone, rolling file into
`briefs/`, and the daily note pulls them in. Most of the system's value lands here.

| Task | When | What it does |
|------|------|--------------|
| `meeting-sync` | ~midnight | Files yesterday's meeting transcripts into the right project folder, wikilinking attendees. Dedupes via a state file. |
| `email-gravity` | ~04:00 | One ranked view of inbox pressure — who's waiting, what's urgent, what's ignorable. Sweeps incrementally via a small state file; carries over yesterday's open items. |
| `meeting-prep` | ~05:00 | A prep brief for each of today's meetings: attendee context, last notes, open items. |
| `tasks-brief` | ~05:30 | A smart view onto the task file: flagged items, hot-by-area, what's stale, what's overdue. Reads only — never edits the task file. |
| `night-shift` | ~06:00 | A divergent overnight read: connects threads across recently-touched files, names blind spots, proposes a few surgical moves. |
| `capture-to-task` | every 15 min | Near-real-time companion: drops newly-flagged emails straight into the task file during the day. |
| `periodic-digest` | daily | An append-only, de-duplicated inventory of a recurring inbound stream (a running index, not a rolling file). |
| `vault-hygiene` | weekly | Auto-applies safe cleanup and proposes the rest — the housekeeping pass that keeps the vault from rotting. |

### The lessons baked into this model

- **Consolidate.** An earlier version had separate triage / stale-reply / action-extractor / Monday /
  Friday jobs. They were collapsed into one inbox sweep and one task brief. Fewer self-contained tasks
  beat many brittle chained ones.
- **Rolling files fail quietly.** A rolling output overwrites in place, so a dead pipeline just leaves
  yesterday's file with no error. Every brief prints its own date — you check the date, not mere
  presence, when something looks off.
- **State files for sweeps.** Tasks that sweep an inbox keep a small JSON state file (last-run
  timestamp) so each run only processes what's new, with per-item dedupe as the real safety net.

> **Model choice.** The scheduled tasks inherit the desktop's default model — currently the latest
> Claude. One concrete lesson: a different model family was trialled for "cognitive diversity" but its
> max output-token limit was far lower and it errored on real, tool-heavy runs. **Check a model's max
> output tokens against your actual workloads before committing.** Diversity is a nice-to-have; a model
> that errors on long tool turns is not.

---

## 04 — The Always-On Layer

### An agent you can message from anywhere

Cowork is great while a machine is awake. The optional final layer adds a 24/7 agent on a small,
hardened mini-PC that you reach from your phone — to capture a thought at 6am, pull a brief as a voice
note, or run research while walking. This kit uses an open, self-hostable agent framework; the shape
applies to any sandboxed, chat-capable agent. This is the most security-sensitive layer, so the
discipline matters most here.

> **Rule zero — do this first.** Split personal from work **before** putting an always-on agent in
> front of the vault. An agent you can DM from anywhere must never have personal content in its prompt.
> The work vault syncs to all devices and the always-on machine; a separate personal vault syncs to
> your devices only — the always-on machine never gets it.

### The security model

- **Least-privilege users.** A non-admin user (not in sudoers) runs the agent; a separate admin user
  handles system changes. The blast radius of a compromised agent is then just one home folder.
- **Every tool call is sandboxed.** The key boundary: tool calls run inside a hardened container, not
  on the host. Drop all capabilities and re-add only what's needed, no new privileges, a PID limit,
  and no host secrets forwarded in unless explicitly allowlisted.
- **Mount only what you intend.** Bind-mount the *work* vault into the container and nothing else. The
  agent reads freely within the mount; if you don't want it touching code repos, you don't mount them.
- **Network & secrets discipline.** Encrypted disk, firewall on, no public exposure — remote access
  only over a private VPN mesh. An outbound firewall catches anything phoning home. Secrets come from a
  password-manager CLI into the agent's environment, never pasted into a prompt.

### The drafts-only rule

> **Non-negotiable, baked into the agent's memory.** The always-on agent writes to `agent-drafts/`,
> `briefs/`, and (for quick capture) `TASKS.md` / `NEW.md` — and nothing else. **No moves, renames, or
> deletes outside those without explicit permission in the same message.** If it thinks something
> should be archived, it proposes and waits. This is a convention enforced via memory — which is
> exactly why the model choice and the memory instruction both matter. It exists because an over-eager
> model once moved and renamed files unprompted.

### What it does (and deliberately doesn't)

The always-on agent is intentionally **reactive — it runs no scheduled jobs**. All scheduled briefs
come from Cowork on the same machine. Its jobs are narrow:

- **Deliver briefs on request** — synthesise the Cowork-generated briefs and reply as a voice note
  ("brief me", "read the night-shift").
- **Capture** — turn a quick voice or text message into a task or session-log entry.
- **Ad-hoc** — research, drafting, and vault Q&A on demand.
- **Log** non-routine sessions to its own log file, which Cowork folds into the shared session log.

### Voice, in and out

- **Agent → you (text-to-speech).** The agent speaks replies and briefs. A good default rule: if you
  send a voice message, it always replies with a voice note.
- **You → agent (speech-to-text).** Let the messaging platform transcribe your voice messages before
  they reach the agent — it sees the transcript, not the audio. You rarely need to run
  speech-to-text yourself.

The chat gateway runs as a managed service generated by the framework's own command — hand-rolled
service files miss required environment entries and fail silently. The agent's job here is light
(summarise and read, not heavy generation), so a mid-tier model from the same family as the main agent
is the right call.

---

## 05 — Practice

### The working patterns that compound

A few conventions, written once into the AI's memory so every session obeys them. These are the
difference between an AI that helps occasionally and one that operates the vault reliably.

- **The staging convention.** New AI-generated files land in a `_staging/` subfolder first — never
  straight into a live project folder. Edits to existing files happen in place. You review, then
  promote the keepers or delete the rest. A 50-slide deck might have a bug on slide 34; staging is a
  clean review gate that keeps working folders uncluttered.
- **The shared session log.** Every agent session that changes the vault appends one entry to `NEW.md`
  — reverse-chronological, one entry per session, a dated header plus a short summary and artifact
  list. It's how multiple surfaces (the always-on agent, the laptop, scheduled tasks) see what the
  others did. Routine brief regenerations aren't logged.
- **Session-continuity files.** Sessions don't share memory. At a checkpoint, the AI writes a handoff
  note in the project's folder: what's done, what's still broken, what rules were discovered, what the
  next session needs. A fresh session reads it and picks up exactly where the last left off — including
  the hard-won "this breaks in that tool" knowledge. The single most important discovery for
  multi-session work.
- **Archive safety.** Learned the hard way: last-modified date is *not* a staleness signal — reference
  notes get linked, not edited, so they look old while being load-bearing. Before archiving anything,
  grep the vault for inbound links; if a live file links it, don't move it. Default to archive (move),
  not delete.

### Custom skills as reusable playbooks

Skills are instruction sets the AI auto-loads when a trigger phrase fires — brand guidelines, a
document format, an analysis playbook. Each is a folder with a `SKILL.md` (name, description, triggers)
and an optional `references/` folder of worked examples for few-shot grounding. You write one once
you've done the task manually a couple of times and know the exact patterns you want.

- **`/ideate` (promote).** Scans recent meetings, notes, and imports for ideas worth their own page —
  recurring themes and frameworks, not action items — and graduates each into a standalone note with an
  evidence trail and wikilinks.
- **`/crit` (filter).** The counterpart. Where ideate promotes, crit kills the weak ones — separates
  signal from noise, challenges assumptions, exposes blind spots, names the few threads actually worth
  pulling. Run them as a pair.
- **Branded output.** Per-style deck and document skills carry exact colours, fonts, and layout
  patterns, plus reference build scripts — the difference between "generic slides" and slides that match
  your house style.

> **Why this matters.** The ideate → crit cycle stops the vault from accumulating beautiful but
> untested frameworks. And because the conventions live in memory rather than in your head, they
> survive across sessions, surfaces, and even different agents.

---

## 06 — The Surface

### One HTML file that reads the vault

A single self-contained `dashboard.html` at the vault root that reads — and writes — vault files
directly in the browser via the File System Access API. No server, no build step, no backend. Open the
file, grant it access to your vault folder once, and it renders live content.

- **Tasks.** Parses the task file by `###` section. Board view with drag-drop between sections, list
  view, checkboxes that toggle completion, progress bars per area.
- **Briefs.** Renders the latest daily note with transclusions resolved, so embedded briefs show
  inline rather than as raw link syntax.
- **People.** A searchable CRM over the `people/` folder. Grid and table views, sortable columns,
  click-to-edit fields read from frontmatter.
- **Ideas.** Renders the ideas index as styled cards grouped by theme — the output of the `/ideate`
  skill, surfaced where you'll see it.
- **Memory.** Browse and edit the `_claude/` memory files directly — see what the AI "knows" and
  update it without leaving the dashboard.
- **No dependencies.** One HTML file with inline CSS and JS. The vault handle is persisted in
  IndexedDB so you don't re-pick each time, and every edit autosaves immediately. It keys off file
  paths and Markdown conventions, so it works on any vault that follows the same structure.

> **Built on, then beyond.** The base dashboard and the single task file ship from a Productivity
> plugin out of the box. This one was extended well past that — a People CRM, an Ideas tab, board view
> with drag-drop, transclusion resolution, and autosave — but the point stands: you can start with the
> generated version and grow it.

---

## 07 — Build Your Own

### Replicate this in three phases

Phases 1 and 2 deliver most of the value — you can stop after Phase 2 and have a working AI-native
second brain. Phase 3 (always-on) is optional and the most involved. Build phase by phase; don't begin
one until the previous one verifies.

**Decide these first:**

- **Your areas** — the 4–10 top-level work streams or life areas that become your task sections and
  project folders.
- **Personal vs. work** — if you'll ever add an always-on agent, plan a separate personal vault now.
- **Connectors** — which inboxes, calendars, and meeting tools the AI may use for the briefs.
- **How far** — stop at Phase 2, or go all the way to Phase 3?
- **Backups** — live sync is a given; pick a git host and an optional cloud mirror.

**Phase 1 — the Obsidian foundation**

1. **Create the folder skeleton** (from §02). Keep the folder names — the later automation keys off them.
2. **Install plugins** — Dataview, Templater, Local REST API, optionally Smart Connections and Web
   Clipper. Enable the core Daily Notes plugin.
3. **Wire daily notes + Templater** — new notes to `daily/`, date format `YYYY-MM-DD`, template at
   `templates/daily-note.md`, with "trigger Templater on new file creation" on.
4. **Seed the CRM** — make a couple of people pages with frontmatter and a Dataview backlinks block;
   write a meeting note that wikilinks them; confirm the backlink query shows it.
5. **Project hubs + task file** — one hub per work stream with a live Dataview meeting query; a single
   `TASKS.md` sectioned by area; establish the `_staging/` convention.

**Phase 2 — Cowork automation**

6. **Build the memory tree** — create `_claude/CLAUDE.md` and the `memory/` subtree; write the staging,
   session-log, and archive-safety conventions into `memory/context/` so every session obeys them.
7. **Set up scheduled briefs** — start with meeting-prep, tasks-brief, and email-gravity. Each writes
   one rolling file to `briefs/`, with its working folder set to the vault root and a state file for
   incremental work.
8. **Wire the daily note + dashboard** — have the daily-note template transclude today's briefs;
   optionally generate `dashboard.html`; create your first skill or two.

**Phase 3 — always-on (optional)**

9. **Confirm the personal/work split** — non-negotiable before going further. The always-on machine
   only ever syncs the work vault.
10. **Harden, sandbox, mount, wire** — non-admin agent user, encrypted disk, VPN-only access, outbound
    firewall; tool calls in a container; mount only the work vault; choose a model (check max output
    tokens); wire messaging + voice; bake in the drafts-only rule.
11. **Automate the git backup** — a daily cron on the always-on machine that commits and pushes the
    whole vault — the third backup layer alongside live sync and a cloud mirror.

> **Verification mindset.** Good verifications are concrete: create a test person and meeting and
> confirm the backlink query shows it; confirm a scheduled task actually wrote a dated brief; confirm
> the daily note transcludes today's briefs; and for Phase 3, ask the agent to "tidy up" something
> outside its write scope and confirm it proposes rather than acts. Prefer to *demonstrate* a
> capability over asserting it.

---

## 08 — The Horizon

### Where it's going next

Not committed — captured so the ideas don't get lost. Each one extends the same core: a plain-text
vault, operated by agents.

- **Conjoined brains.** Share part of a vault with a partner, colleague, or spouse and create a
  combined workspace that draws on everyone's collected meetings and notes. A co-brain, not just a
  shared folder.
- **Voice UI.** A voice agent on the phone that reads out tasks, briefs, and ideas while driving — and
  auto-generates a weekly ~30-minute podcast that reviews the past week and previews the next.
- **Auto-drafted action plans.** For each task, research in the background and draft the docs, emails,
  or messages needed to action it — shifting from tracking the work to actually doing it.
- **Task gamification.** Surface tasks one at a time in a mobile-first focus mode that makes knocking
  them off feel good — focus mode, not list mode.
- **Fractional EA integration.** Connect a human executive assistant into the vault — same context the
  AI has — so they can pick up tasks, update status, and draft on your behalf. The vault becomes the
  shared operating surface.
- **A second always-on agent.** When a more trusted self-hosted agent is available, run it on a
  separate machine and sync the vault to it — using the vault as the loosely-coupled, safe interface
  between you and a more autonomous worker.

---

> *The "app" is just the vault — and a few agents that operate on it.*
