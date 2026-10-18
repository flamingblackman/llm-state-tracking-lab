# Living World Pipeline — full case study

*The night shift of the [LLM State-Tracking Lab](../). Four original RPG sagas run on the lab's state contract; this pipeline keeps their worlds alive between sessions.*

## The sagas

| Saga | Genre | Notes |
|---|---|---|
| Cursed Resonance | Grimdark Lovecraftian mecha | Original setting |
| Unwritten Age | Dragon Ball | Original saga |
| The Hollow Vow | Jujutsu Kaisen | Original saga |
| Pale Fire | Digimon | Original saga |

A fifth "Shadow Director" Gem sits above them all, firing crossover ripple events between sagas.

## The problem it solves

The lab's state contract (per-turn state blocks, self-audits, canon compression) covers what happens *during* play. It had no answer for what happens *between* sessions: faction clocks stalled the moment a session ended, off-screen NPCs froze mid-scheme, and canon only advanced while someone was at the table. A world that only exists when observed isn't a living world — it's a paused video game.

## How the system is updated

There are two update paths: **canon changes** (the world's source of truth) and **play changes** (what happens in sessions). Both are mechanical, repeatable, and auditable.

### Path 1 — Canon sync

Canon lives as Markdown files, one directory per saga: series bibles, combat codices, NPC voice ledgers. When the world changes:

1. Edit the `.md` source of truth.
2. Run `canon-sync.sh` — one command rebuilds every canon `.docx` through a Markdown-to-Word converter.
3. Upload the rebuilt files **in place** to each saga's Google Drive folder (same file IDs — no link ever breaks, no re-sharing).
4. Google NotebookLM auto-syncs the per-saga notebooks from Drive.
5. The Gemini Gems read the refreshed files as knowledge.

No manual reformatting. No drift between the source and what the model sees. The whole path is one script plus an upload.

### Path 2 — Nightly showrunner loop

A scheduled run (~9:40 PM ET) does six things, in order:

1. **READ** — new messages from each saga's play chat since the last stored watermark.
2. **ARCHIVE** — session filed by date: raw log, cleaned chronicle delta, final turn state.
3. **WORLD-MOVES** — every faction clock advanced based on what happened (or didn't happen); off-screen NPC actions resolved with clear fictional consequences, written into the clock ledger.
4. **DRIFT SCAN** — new material checked against canon: contradictions, unmarked new facts, wrong-saga terminology. Each flagged with a proposed fix — never silently rewritten.
5. **BRIEF** — per-saga brief: clock positions, what's about to detonate, 2–3 hooks for the next session.
6. **DELIVER** — a report goes out for every saga with new play. If nothing was played anywhere, the run stays silent.

## Operating rules

- **Read-only on play chats.** The pipeline never types into, sends in, or acts inside a session. Anything outbound needs explicit per-message human approval.
- **The human alone decides canon.** New facts from play are marked; conflicts are flagged with a proposed fix, never auto-resolved.
- **Mysteries are off-limits.** Seeded secrets the player hasn't discovered are never resolved, hinted at, or spent by the system — even if a clock detonation would touch one. That thread stops and gets flagged instead.
- **State is files, not vibes.** Watermarks, clock ledgers, briefs, and archives are plain Markdown — auditable by eye, diffable, portable. No database required to verify a commit.

## Roadmap

- **November 2026** — Gems migrate to Skills; self-narrating "meanwhile" interludes delivered by scheduled tasks invoking those Skills (wiring already staged).
- **Voice play** via Gemini Live — sessions move from text to speech.
- **Recap podcasts** — NotebookLM turns session archives into audio briefings.
- **Illustrated play + found-document props** — generated visuals and in-world artifacts.
- **Public Gem sharing** — packaged, shareable versions of the saga Gems.

## Why this matters

This is applied AI systems work, not a toy demo:

- **Multi-agent orchestration** — five Gems plus an unattended showrunner, coordinated through files rather than a message bus.
- **Long-context state management** — Markdown as the durable memory layer; delta-only updates; portable canon blocks that survive context resets.
- **Retrieval-augmented canon** — Drive → NotebookLM → Gem knowledge, with a one-command sync path that keeps the corpus identical to the source of truth.
- **Human-in-the-loop at every load-bearing decision** — audits, canon commits, and anything outbound all pass a human gate. The automation runs the world; it doesn't get the final word.

*Story content itself is private and stays local. This documents the system, not the fiction.*
