# Living World Pipeline — full case study

*The night shift of the [LLM State-Tracking Lab](../). Six narrative lines run on the lab's state contract; this pipeline keeps their worlds alive between sessions.*

## The sagas

| Saga | Genre | Notes |
|---|---|---|
| Cursed Resonance | Grimdark Lovecraftian mecha | Original setting |
| Unwritten Age | Dragon Ball | Original saga |
| The Hollow Vow | Jujutsu Kaisen | Original saga |
| Pale Fire | Digimon | Original saga |
| The Spin-Off | Spin-off line | Fed by a rumor-propagation sheet: main-story events mutate as they travel into this line. The distortion is a designed mechanic, not an error source |
| Second Author's Saga | Original saga, second author | Built from the author's own source texts; runs on the same state contract under the cross-line decontamination rules below |

A Shadow Director layer sits above them all, turning session deltas into cross-line ripple events. It only ever speaks in the guarded format documented below.

## The problem it solves

The lab's state contract (per-turn state blocks, self-audits, canon compression) covers what happens *during* play. It had no answer for what happens *between* sessions: faction clocks stalled the moment a session ended, off-screen NPCs froze mid-scheme, and canon only advanced while someone was at the table. A world that only exists when observed isn't a living world — it's a paused video game.

## How the system is updated

There are two update paths: **canon changes** (the world's source of truth) and **play changes** (what happens in sessions). Both are mechanical, repeatable, and auditable.

### Path 1 — Canon sync

Canon lives as Markdown files, one directory per saga: series bibles, combat codices, and NPC voice ledgers (character voice maintained as structured data for four sagas: speech patterns, wants, and drift notes, checked the same way numeric state is). When the world changes:

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

## The Shadow Director protocol

The Director reads session deltas across the lines and generates "ripple" events: consequences that travel from one line's play into another line's background. Every ripple is delivered in exactly one format:

```text
[SHADOW DIRECTOR OVERRIDE]
Background only — do not advance the current scene or resolve any pending player decision.
Ripple: <the background event, written to land as world state, not as scene>
```

The guard line is load-bearing, and it exists because the failure happened first. Early unguarded injections read as ordinary narration to the receiving chat, which did the helpful thing: played them. Scenes advanced without the player, and pending player decisions got resolved by background events. The fix was format, not exhortation: a marked header the receiving model treats as a different message class, plus an explicit background-only instruction. Since the guard went in, ripples land as logged background state and scenes stay untouched.

Every injection is written to an injection log with its per-target outcome (landed as background / scene touched / decision spent), so the protocol's track record is auditable rather than anecdotal. A sanitized sample, with a drift-scan excerpt, lives at [`../sample-logs/ripple-injection-03.md`](../sample-logs/ripple-injection-03.md).

## Faction dashboards: clocks as data

Faction clocks are the pipeline's most-touched state, so they stopped living in prose. A single source-of-truth JSON holds every line's clocks (name, position, size, last trigger), and a Google Sheets dashboard renders it for a fast human read. The nightly loop updates the JSON during WORLD-MOVES; the dashboard is a view, never a second source. When a brief cites a clock position, it cites the JSON's value.

## Sealed knowledge

Some material is secret by design: identities, plans, and reveals the player has not earned yet. That material is held in instruction text only, never in knowledge files, because anything in a knowledge file is one retrieval away from surfacing early. Reveals follow a phased ladder (what may be hinted, what may be shown, what may be confirmed), and every drift scan includes an audit for unearned reveals: a secret that appears ahead of its phase is flagged like any other contradiction.

## Cross-line decontamination

Six lines on one apparatus will contaminate each other if the boundaries are vibes. The rules are mechanical instead: canon is namespaced per line, each line's state blocks carry its own canon version, and terminology from one line appearing in another line's scan is a drift flag. The single sanctioned crossing is the rumor sheet: main-story events may enter the spin-off line only as rumors, mutated by travel, with the original and the rumor kept side by side so the distortion is tracked instead of mistaken for canon.

## API tandem line (October 2026)

A second model API now works alongside the pipeline as a production assistant: consults on world-moves, first drafts of briefs, and audit passes. Nothing it produces ships directly. Every output passes the same human QC gate as model output from the play chats, and canon decisions stay with the human.

Three operational lessons, all learned in production:

- **Verify stored configuration, not intended configuration.** The target chats' *stored* model silently differed from the model the operator had picked. A five-target injection run failed 5/5 on the stored model and landed 5/5 after switching the chats to the intended one. The content was never the problem; the config layer was. Multi-target runs now start with a stored-settings check.
- **Session regeneration reverts to stored state.** A regenerated session goes back to what is stored, not what the last session had live. Any fix that is not written to the stored configuration is a fix that will quietly un-happen.
- **Usage exhaustion means stop and skip.** When a target hits its usage limit, the run skips it and moves on. Retrying against an exhausted quota churns time and produces partial, inconsistent state.

## Operating rules

- **Read-only on play chats.** The pipeline never types into, sends in, or acts inside a session. Anything outbound needs explicit per-message human approval.
- **The human alone decides canon.** New facts from play are marked; conflicts are flagged with a proposed fix, never auto-resolved.
- **Mysteries are off-limits.** Seeded secrets the player hasn't discovered are never resolved, hinted at, or spent by the system — even if a clock detonation would touch one. That thread stops and gets flagged instead.
- **State is files, not vibes.** Watermarks, clock ledgers, briefs, and archives are plain Markdown — auditable by eye, diffable, portable. No database required to verify a commit.
- **Guarded injections only.** The Director never sends an unguarded ripple. If the guard format cannot be applied, the ripple waits.
- **Tandem output is draft-only.** The second model's consults, drafts, and audits pass the human QC gate before anything treats them as state.
- **Stop and skip on exhaustion.** A target at its usage limit is skipped for that run, never retried into partial state.

## Roadmap

- **Skills migration (platform change lands November 17, 2026)** — the Gems platform migrates to Skills. Researched prep, from the migration playbook: migrated units arrive as *inactive drafts*, so activation is a deliberate step, not an assumption; knowledge format constraints change (no .docx; 100MB cap per skill), so canon knowledge files are being converted to supported formats ahead of the move; self-narrating "meanwhile" interludes delivered by scheduled tasks invoking those Skills (wiring already staged).
- **API tandem expansion** — more first-draft and audit work routed to the tandem line behind the same QC gate, with the stored-configuration check as a standing pre-flight step.
- **Voice play** via Gemini Live — sessions move from text to speech.
- **Recap podcasts** — NotebookLM turns session archives into audio briefings.
- **Illustrated play + found-document props** — generated visuals and in-world artifacts.
- **Public Gem sharing** — packaged, shareable versions of the saga Gems.

### The next three builds (planned)

Sequencing: Item 2 enables Item 1 (a quarantined corpus is testable). Item 1 gates Item 3 (the state card earns trust once the suite passes). All three are planned work, not shipped features.

**1. Canon Test Suite (evaluation harness).**
*Problem:* the lab claims retrieval strategy as a designed skill, but retrieval quality is never measured. Canon can rot silently (stale sources, superseded material) and nobody knows until a session goes sideways.
*Design:* (a) Golden questions: a fixed set of questions per saga with known-correct answers, run against the NotebookLM notebooks after every canon sync, with pass/fail logged. (b) Adversarial probes: deliberately canon-contradicting statements fed to the Gems to find where the models fold or hallucinate; weak spots get logged as high-risk lore nodes. Together they turn canon accuracy from a hope into a checked metric.
*Why it matters:* this is the item that makes the portfolio's central claim (designed corpus plus retrieval strategy on a managed RAG platform) falsifiable, and therefore credible. It is real LLM-ops work: evaluation, regression checks, adversarial testing.

**2. Corpus Quarantine + Lifecycle Tags.**
*Problem:* superseded canon stays in the active retrieval corpus. Live example: two corpus documents still describe a deleted continuity's protagonist and a retired power system. The Gem layer carries override instructions, but NotebookLM retrieval surfaces the dead material as if it were current.
*Design:* every corpus section carries a lifecycle tag (Active / Historical / Archive), and superseded material is quarantined out of the active corpus rather than merely overridden in prompts. Retrieval weight follows the living story by structure, not by instruction-fighting.
*Why it matters:* it demonstrates corpus governance, the unglamorous discipline that separates a designed corpus from a pile of uploads. It pairs with the test suite: quarantine is what the golden questions verify.

**3. Living State Card.**
*Problem:* each Gem reconstructs "current state" from a long, drifting chat history. State errors compound across sessions.
*Design:* after each pipeline sweep, generate one compact, versioned state block per saga (clocks, open threads, pending decisions, pilot condition) and append it to that Gem's instructions, superseding the previous card. The append-and-verify mechanism already exists (it is how correction blocks are delivered); this item turns it into a rhythm.
*Why it matters:* it shows state management moving from prose memory to controlled, versioned artifacts: the lab's thesis in miniature.

### The Living World cluster (planned)

Five mechanics that run on one base rule. All planned, none shipped yet.

**Base rule: one action per faction per sweep.**
*Problem:* faction clocks measure pressure, but a countdown is not an opponent. Factions can feel static between detonations.
*Design:* every living-world sweep, each active faction takes ONE concrete off-screen action toward its agenda, logged in its clock ledger (a seizure, a purchase, a recruitment, a burial). Every item below runs on this rule.
*Why it matters:* it is the difference between a world that waits and a world that moves. Simple, auditable, and the engine under the rest of this cluster.

**LW-1. The Scavenger Cascade.**
*Problem:* the player's battlefields and abandoned sites sit frozen in the archive after he leaves them.
*Design:* the sweep tags sites of wreckage, casualties, or power vacuums as carrion nodes, each spawning a short clock. Minor factions and opportunists strip the site; when the clock fills, a low-tier group returns upgraded with the player's own leftovers (salvaged gear, claimed micro-turf, mutated capability).
*Why it matters:* consequences with a half-life. The player's past violence literally re-enters the story wearing his discarded equipment.

**LW-2. Weaponized Rumor Feedback (closed-loop mutation).**
*Problem:* the spin-off's rumor engine runs one way: main-saga events mutate as they travel outward, then stop.
*Design:* close the loop. A distorted version (the Myth) that survives two spin-off sessions unchallenged is exported back into the main saga as a Foreign Misconception: factions react to the exaggeration instead of the truth, fortifying against capabilities the player does not have, repricing bounties on a legend.
*Why it matters:* the player ends up fighting his own mythology. Emergent, cheap, and unique to this system's two-layer saga structure.

**LW-3. Debt & Favor Economy (NPC off-screen agency).**
*Problem:* NPCs freeze when the player is not looking at them; neglected contacts wait forever in narrative stasis.
*Design:* key NPCs carry one pressure tag (hunted, broke, desperate). If the player ignores a pressured NPC for two sessions while a related faction clock advances, the NPC resolves their own crisis off-screen via a Bad Bargain with a rising faction, logged as a compromised-by tag in their voice ledger.
*Why it matters:* relationships decay and mutate on their own clock. Returning to a contact whose problems are mysteriously solved is a story beat no static NPC system produces.

**LW-4. Voice Ledger Contagion.**
*Problem:* voice ledgers keep NPCs consistent but static; consistency without drift reads as robotic over a long campaign.
*Design:* two dynamic tags updated at drift scan. Borrowed cadence: allies absorb the player's own phrases and rationalizations over time. Grudge drift: a neglected NPC's registered voice shifts colder (clipped, formal, cynical) before any story event explains why.
*Why it matters:* social momentum you can hear. Relationship state becomes audible in dialogue before it becomes plot.

**LW-5. Sensory Weather (environmental memory).**
*Problem:* faction dominance gets announced in exposition ("Faction X is winning") instead of felt in description.
*Design:* a three-line District Texture Block tops each saga brief (signage and architecture, street smell and sound, civilian micro-behavior). When a clock crosses its halfway mark or a bounty posts, one line is rewritten. Gems describe from the block; the world shows its politics through bartenders demanding hard currency and locals crossing the street.
*Why it matters:* near-zero cost, constant payoff. Texture as a state variable, versioned like everything else in the lab.

**Horizon.** After the November 17, 2026 Gems-to-Skills migration, scheduled tasks can invoke a saga directly, making true between-session "meanwhile" interludes possible: the platform unlock this cluster is designed to grow into.

## Why this matters

This is applied AI systems work, not a toy demo:

- **Multi-agent orchestration** — five Gems plus an unattended showrunner, coordinated through files rather than a message bus.
- **Long-context state management** — Markdown as the durable memory layer; delta-only updates; portable canon blocks that survive context resets.
- **Retrieval-augmented canon** — Drive → NotebookLM → Gem knowledge, with a one-command sync path that keeps the corpus identical to the source of truth.
- **Human-in-the-loop at every load-bearing decision** — audits, canon commits, and anything outbound all pass a human gate. The automation runs the world; it doesn't get the final word.

*Story content itself is private and stays local. This documents the system, not the fiction.*
