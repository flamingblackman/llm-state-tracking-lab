# Living World Pipeline — full case study

*The night shift of the [LLM State-Tracking Lab](../). Six narrative lines run on the lab's state contract; this pipeline keeps their worlds alive between sessions.*

## The sagas

| Saga | Genre | Notes |
|---|---|---|
| Cursed Resonance | Grimdark Lovecraftian mecha | Original setting — designated cross-saga integration partner |
| Unwritten Age | Dragon Ball | Original saga |
| The Hollow Vow | Jujutsu Kaisen | Original saga — **reference implementation**: formal validation concentrates here (active faction clocks, recurring NPCs, permanent consequences, and the system's flagship sealed-knowledge case) |
| Pale Fire | Digimon | Original saga |
| The Spin-Off | Spin-off line | Fed by a rumor-propagation sheet: main-story events mutate as they travel into this line. The distortion is a designed mechanic, not an error source |
| Second Author's Saga | Original saga, second author | Built from the author's own source texts. Formal policy (ratified October 23, 2026): on-request only, with zero scheduled processing — no sessions or sweeps run for it, and no automatic cross-saga propagation into it |

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

### Path 2 — Showrunner sweep (session close / on request)

Sweeps are triggered by a session's close (the standing "that's a wrap" ritual) or run on request; a saga accumulating roughly 12–15 unswept turns raises a flag in the next debrief. There are no silent scheduled runs. A sweep works in seven stages — capture, normalize/reconcile, evaluate, validate, digest, commit, queue — which unfold as six steps:

1. **READ** — new messages from each saga's play chat since the last stored watermark.
2. **ARCHIVE** — session filed by date: raw log, cleaned chronicle delta, final turn state.
3. **WORLD-MOVES** — every faction clock advanced based on what happened (or didn't happen); off-screen NPC actions resolved with clear fictional consequences, written into the clock ledger.
4. **DRIFT SCAN** — new material checked against canon: contradictions, unmarked new facts, wrong-saga terminology. Each flagged with a proposed fix — never silently rewritten. The scan also runs the Epistemic Promotion Audit (below): a belief narrated as objective fact is flagged as a contradiction.
5. **BRIEF** — per-saga brief: clock positions, what's about to detonate, 2–3 hooks for the next session.
6. **DELIVER** — a report goes out for every saga with new play. If nothing was played anywhere, the run stays silent. Ripples and handshakes are queued locally and delivered when the target saga is next opened (Lazy Pull-On-Play); deliberate cross-saga broadcast events on the operator's order are the carve-out.

**Run record and acceptance.** Every run records its input watermark, source versions, actions taken, errors, and approval status. A run counts as successful only if: no duplicated deltas, no unapproved canon changes, correct clock arithmetic, no known premature reveals, and failures recoverable from the run record. Prose quality is not a state-correctness metric.

### Field-level authority

One declared authority per data type, with an explicit sync direction. Everything else is a view.

| Data | Authority | Notes |
|---|---|---|
| Canon lore + approved rulings | Versioned Markdown files | The `.md` sources Path 1 syncs |
| Numeric state + faction clocks | Versioned state ledger (clock JSON) | Dashboards render it; they never own it |
| Raw session evidence | Archived session logs | The record reconciliation checks against |
| Pending decisions | Decision queue | Surfaced in the operator digest; nothing auto-commits |
| Secret content | Restricted storyteller store (local files) | The pipeline tracks reveal-eligibility metadata only |
| Test expectations | Version-controlled fixtures | Canon Test Suite inputs and expected outputs |

Google Sheets and NotebookLM are representations and retrieval layers. Neither is ever an authority.

### Epistemic typing

Ledger entries carry one of two types. **[FACT]** — verified on-screen in play, or established by an operator ruling. **[BELIEF: holder]** — a position held by a named NPC or faction, attached to that holder's posture or next move. Possibilities are never tracked: a possibility is either sealed lore (the reveal ladder governs it) or transcript chatter. Rumor traffic is auto-typed [BELIEF: Street_Rumor], which is what structurally protects the rumor-feedback loop (LW-2) from promoting its own distortions.

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

Faction clocks are the pipeline's most-touched state, so they stopped living in prose. A single source-of-truth JSON holds every line's clocks (name, position, size, last trigger), and a Google Sheets dashboard renders it for a fast human read. The sweep updates the JSON during WORLD-MOVES; the dashboard is a view, never a second source. When a brief cites a clock position, it cites the JSON's value.

**Ledger vs. surface.** Clock ticks in the ledger are uncapped: every earned advance lands in the data layer, under the evaluate-always rule below. A separate world-beat budget — one major beat plus up to two minor — caps only what gets promoted into the Session-Start State Handshake for dramatization, so slow-burn clocks are never starved while the narrative surface stays focused. A clock completing, or crossing its midpoint, is major-eligible.

## Sealed knowledge

Some material is secret by design: identities, plans, and reveals the player has not earned yet. It comes in two classes with two treatments. **Storyteller-held secrets** live in Gem instruction text, never in knowledge files, because the storyteller has to know a secret to pace its reveal in-story. That boundary is behavioral guidance, not access control — nothing technical stops a model from blurting — so it runs with compensating controls: a phased reveal ladder (what may be hinted, what may be shown, what may be confirmed), drift scans that audit for reveals ahead of phase, a reaction-proportionality check that catches leaks by implication, and a human brake. **Sealed material** never enters any Gem or corpus at all: it is held in local files only, and the pipeline tracks nothing about it except reveal-eligibility metadata — which phase a reveal has reached, never the content. A secret that surfaces ahead of its phase, in either class, is flagged like any other contradiction.

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

### Consolidated build order (October 22, 2026)

This order supersedes the sequencing in today's earlier roadmap notes. It came out of a two-round design spar over the whole project; what follows is the settled version. Everything here is planned work, labeled with its build position.

**Step 1 — Item 0: Pre-Migration Skill Packaging Spec (highest urgency, due before November 17, 2026).**
*Problem:* Gems migrate to Skills on November 17. Migrated units arrive as inactive drafts, under stricter knowledge-format constraints. An unprepared cutover risks six live sagas waking up in a new format with unverified instructions and no sync path.
*Design:* (a) Instruction mapping: a SKILL.md structure per saga, with explicit boundaries for persona and mechanics, the standing rituals, and the sealed reveal ladder. (b) A knowledge-format and sync-fallback audit, including the known trap: if Skills do not auto-sync live Google Docs by ID, the canon-sync script gains a fallback branch that emits clean .md bundles alongside the .docx output. (c) A cutover runbook on fixed dates: all six Gems' instruction blocks exported to local, versioned backups by **October 31**; a compatibility test by **November 7**; the Markdown-bundle fallback validated in a fresh session by **November 12** (a backup of instructions is not a reproduced environment — the fallback has to be proven, not assumed); freeze and verify **November 15**; platform migration from **November 17**; a verification pass **November 18**. Every Gem-instruction addition from the recent design rounds (Sensory Cold Open, the fun layer below, natural stopping points, callback acceptance via handshake, state-block echo) batches into this item's single pre-migration edit window per Gem, so the October 31 backups capture the improved text and no saga's editor is fought twice.
*Why it matters:* every other item on this roadmap assumes the sagas survive the platform move. This one makes sure they do.

**Step 2 — Baseline Canon Test Suite, the dirty run.** The suite has four test classes: exact state, canon conflicts, stale-source retrieval, and premature secret disclosure (including indirect-leakage probes — leaks by implication, not just blurts). Golden questions run against the corpus exactly as it stands in NotebookLM, plus adversarial probes: 3 per saga, run only on canon commits, in disposable fresh Gem chats that are deleted afterward (never in live play chats; context pollution). The first full run centers on the reference saga (The Hollow Vow), whose corpus carries the known stale-document contamination the quarantine in step 3 exists to fix. The baseline's failures are not hidden. They are recorded, and they are portfolio evidence: this is what turns the lab's central claim (designed corpus plus retrieval strategy on a managed RAG platform) into a checked metric instead of a hope. One caveat, kept on the record: improvement on a small suite demonstrates improvement on those tests, not universal retrieval accuracy.

**Step 3 — File-Level Corpus Quarantine.** Superseded documents are removed from the active source set. Lifecycle tags alone do not work here: NotebookLM chunking ignores document headers, so a tag the retriever cannot see is bookkeeping, not governance. Tags (Active / Historical / Archive) stay as bookkeeping only. After quarantine, the suite re-runs and the clean pass is recorded next to the dirty baseline. The live problem this fixes is real: corpus documents still describe a deleted continuity's protagonist and a retired power system, and retrieval surfaces that dead material as if it were current.

**Step 4 — Portfolio Proof Pack.** (a) A swimlane boundary in the README: deterministic code in one column, LLM-in-the-loop orchestration in the other, no blurring. (b) One redacted end-to-end diff artifact: raw play excerpt, drift scan output, the exact clock JSON diff, then a session brief carrying a guarded injection. (c) The before/after quarantine test scores from steps 2 and 3.

**Step 5 — Living State Card, redesigned as a Session-Start State Handshake.** The card is injected into the play chat at session start: a superseding header, diffs stated explicitly, and the Gem echoes the card ID in its state blocks so the handshake is verifiable in the log. It never goes in Gem instructions and never rides a mounted Doc. The handshake also carries the LW-5 texture block and cycle state (below). The Sensory Cold Open, a two-sentence pan at scene transitions (one clock symptom, one texture detail), ships as a one-time standing instruction in the same step.

**Step 6 — Base Rule v2 + LW-3 (Debt & Favor) + LW-4 Grudge Drift only.** The base rule ships in its amended form: every faction is *evaluated* each sweep; it acts only when goal + capability + trigger + material effect justify it, otherwise the no-action reason is recorded (spec below). Borrowed Cadence is cut as a mechanic (parrot risk); it survives only as a drift-scan observation. Specs in the cluster below.

**Step 7 — LW-1 (Scavenger Cascade), capped at one active node per saga; then LW-2 (Weaponized Rumor Feedback) last — decision closed October 23, 2026: KEEP.** The dispute record: one spar position was to cut LW-2 entirely (its payoff is hostage to two sagas' play cadence); the dissent noted the one-way rumor engine already runs, so the marginal cost is a briefs rule. The settled design keeps LW-2 last in the living-world queue under implementation conditions (spec below): the rumor must persist through two qualifying sessions in its source saga; it arrives in the target saga as a *pending* false-belief event the player can accept, reject, or defer — never as a canon edit; one controlled seeded test runs before the mechanic is trusted; and it is cut if it ever needs frequent manual sync.

**Integrity additions (planned).** The drift scan gains a reaction proportionality check: NPC reactions that exceed publicly established stakes get flagged, which catches leaks by implication that category gates miss. A schema and consistency gate on state writes (clock bounds, no silent decreases) is planned as the named fix for the chat-to-state extraction boundary. Both are governed by the State Reconciliation Contract (adopted October 23, 2026), which also governs the parser build:

- Every state delta carries a unique ID, its saga ID, its source session and turn, and the expected base-state version it was computed against.
- Re-applying an accepted delta is a no-op (idempotent commits).
- A delta that arrives against a stale base version is flagged, never silently applied.
- Conflicts are never auto-resolved by newest-wins.

**Deterministic pre-sweep parser (planned).** Before a sweep's reconciliation, deterministic code parses captured session text for the Gems' per-turn state blocks, diffs first against last, and matches NPC names against the voice ledgers, handing the model a pre-digested delta instead of raw transcript. The extraction and validation code is deterministic for supported inputs; malformed or ambiguous outputs are rejected into an exception queue, never guessed at. First target: the reference saga (The Hollow Vow), whose per-turn state block is standardized as part of reference setup; the parser covers state-block-bearing sagas first.

**Sequencing logic.** Pipeline-side items (the base rule, LW-1/2/3, grudge drift, the LW-5 math) are migration-proof. NotebookLM-side items (quarantine, golden questions) are near-proof. Only delivery mechanisms sit on the Gem surface, and only those carry November 17 risk. The order spends effort where the platform cannot take it away first.

### Portfolio translation track (added October 23, 2026)

The consolidated build order above keeps the story system correct. A second track turns that correctness into career evidence, sequenced by dependency and written up in full in [`../ROADMAP.md`](../ROADMAP.md):

1. **Deterministic pre-sweep parser, first target the reference saga (The Hollow Vow).** Its state block is standardized as part of reference setup, and the parser is built under the State Reconciliation Contract in the integrity additions above: unique delta IDs, idempotent replay, stale-base flagging, conflicts preserved instead of newest-wins, malformed input rejected to an exception queue.
2. **Business reconciliation demo.** Once the parser demonstrates the contract on session text, the same engine is extracted and run against 30–50 fictional CRM-style update events: duplicate leads, stale versions, conflicting fields, missing required data, unsupported claims. Expected outcomes are defined before the run; the evaluation is published either way, with denominators, pass rates, exception counts, and known failures. The records are fictional and local; no CRM integration is claimed.
3. **Portfolio presentation, gated on the measured results.** LinkedIn and resume framing updates wait until the demo's numbers exist. The story lab stays the real use case; the demo is the labeled transfer evidence.

### The Living World cluster (planned)

Five mechanics that run on one base rule. All planned, none shipped yet. Build positions are set by the consolidated order above.

**Base rule: evaluate always, act when justified.**
*Problem:* faction clocks measure pressure, but a countdown is not an opponent. Factions can feel static between detonations.
*Design:* every living-world sweep, each active faction is *evaluated* against its agenda. It takes a concrete off-screen action (a seizure, a purchase, a recruitment, a burial) only when four tests pass: a goal the action serves, the capability to do it, a trigger in recent events, and a material effect worth logging. When any test fails, the faction holds — and the no-action reason is recorded in its clock ledger. Forced activity on every sweep guaranteed churn, not quality; evaluation is universal, action is earned. Every item below runs on this rule.
*Why it matters:* it is the difference between a world that waits and a world that moves on reasons. Simple, auditable, and the engine under the rest of this cluster.

**LW-1. The Scavenger Cascade.** (Build step 7, capped at one active node per saga.)
*Problem:* the player's battlefields and abandoned sites sit frozen in the archive after he leaves them.
*Design:* the sweep tags sites of wreckage, casualties, or power vacuums as carrion nodes, each spawning a short clock. Minor factions and opportunists strip the site; when the clock fills, a low-tier group returns upgraded with the player's own leftovers (salvaged gear, claimed micro-turf, mutated capability).
*Why it matters:* consequences with a half-life. The player's past violence literally re-enters the story wearing his discarded equipment.

**LW-2. Weaponized Rumor Feedback (closed-loop mutation).** (Build step 7, last. Decision closed October 23, 2026: KEEP, under the conditions in step 7 and below.)
*Problem:* the spin-off's rumor engine runs one way: main-saga events mutate as they travel outward, then stop.
*Design:* close the loop, under governance. A distorted version (the Myth) must first persist through two qualifying sessions in its source saga. It then arrives in the target saga as a *pending* false-belief event — typed [BELIEF: Street_Rumor] under the epistemic rules above — which the player can accept, reject, or defer when that saga is next opened. It is never written in as a canon edit. Accepted, it becomes a Foreign Misconception: factions react to the exaggeration instead of the truth, fortifying against capabilities the player does not have, repricing bounties on a legend. One controlled seeded test runs before the mechanic is trusted in live play, and the mechanic is cut if it ever needs frequent manual sync.
*Why it matters:* the player ends up fighting his own mythology. Emergent, cheap, and unique to this system's two-layer saga structure.

**LW-3. Debt & Favor Economy (NPC off-screen agency).** (Build step 6.)
*Problem:* NPCs freeze when the player is not looking at them; neglected contacts wait forever in narrative stasis.
*Design:* key NPCs carry one pressure tag (hunted, broke, desperate). If the player ignores a pressured NPC for two sessions while a related faction clock advances, the NPC resolves their own crisis off-screen via a Bad Bargain with a rising faction, logged as a compromised-by tag in their voice ledger.
*Why it matters:* relationships decay and mutate on their own clock. Returning to a contact whose problems are mysteriously solved is a story beat no static NPC system produces.

**LW-4. Voice Ledger Contagion: Grudge Drift.** (Build step 6. Borrowed Cadence is cut as a mechanic; see below.)
*Problem:* voice ledgers keep NPCs consistent but static; consistency without drift reads as robotic over a long campaign.
*Design:* one dynamic tag updated at drift scan. Grudge drift: a neglected NPC's registered voice shifts colder (clipped, formal, cynical) before any story event explains why. The companion idea, Borrowed Cadence (allies absorbing the player's own phrases and rationalizations), was cut as a mechanic in the October 22 spar: systematizing it risks parroting. It survives only as a drift-scan observation, noted when it appears naturally, never engineered.
*Why it matters:* social momentum you can hear. Relationship state becomes audible in dialogue before it becomes plot.

**LW-5. Sensory Weather (environmental memory).**
*Problem:* faction dominance gets announced in exposition ("Faction X is winning") instead of felt in description.
*Design:* a three-line District Texture Block tops each saga brief (signage and architecture, street smell and sound, civilian micro-behavior). When a clock crosses its halfway mark or a bounty posts, one line is rewritten. Gems describe from the block; the world shows its politics through bartenders demanding hard currency and locals crossing the street. Delivery: the texture block and cycle state ride the Session-Start State Handshake (build step 5), and the Sensory Cold Open ships with that step.
*Why it matters:* near-zero cost, constant payoff. Texture as a state variable, versioned like everything else in the lab.

*Cycles layer (the designer's own addition):* texture should not only react to faction clocks. Some of it runs on world-owned schedules the player can learn and plan around, so Sensory Weather is threshold rewrites PLUS tracked cycles. Every saga has candidate cycles: lunar phases, tides, market days, school terms, a frozen dawn that refuses to advance. Worked example, from the Dragon Ball line: a **moon tracker**. Each planet the protagonist has visited gets its moon phases tracked as pipeline state, advanced by in-story elapsed time at each sweep, with positions never invented before the story establishes a sky. In that universe a full moon changes what a night means for a Saiyan body, so a full moon falling within one in-story day of a scene gets flagged in the brief: the sky enters the description before it enters the fight.

**Horizon.** After the November 17, 2026 Gems-to-Skills migration, scheduled tasks can invoke a saga directly, making true between-session "meanwhile" interludes possible: the platform unlock this cluster is designed to grow into.

### The fun layer (ratified October 23, 2026 — designed, not yet built)

The machinery above keeps state *correct*. This layer exists because correctness is not the same as fun, and the two designs were deliberately kept from taxing each other.

**Forked Consequence engine.** The per-turn state block gains two fields: [SCENE_TENSION: Low | Rising | Snap-Point] and [ACTIVE_TRADE_OFF]. On a decisive move in a Rising or Snap-Point scene, the Gem may not resolve cleanly: the objective is achieved, and a cost is paid in exactly one currency — time/heat, resource/condition, or an NPC's trust/safety. The fork is a resolution *shape* for decisive moves, not a menu recited on every action, and its harshness is tuned per saga. Two guardrails from the operator govern it:

- *Costs must be earned by established stakes.* A plan that genuinely deserves a clean success resolves clean. The fork never fires merely because a move was clever; punishing cleverness itself would make the engine a disguised penalty rule, not a source of dramatic choices.
- *The six-turn Snap-Point is a pacing trigger, not a punishment.* A scene that runs six turns without a decision or a state change hits Snap-Point, and the NPC's Default Next Move (below) fires in the room and *changes the situation*. Nothing is automatically lost; losses happen only when the fiction earns them.

**NPC voice ledgers absorb the NPC model.** No third table: each arc-active NPC's ledger row carries Voice_Cadence, Core_Want (what they want, in their own terms), Active_Pressure, Default_Next_Move (what they do within roughly two sessions if nobody intervenes), and Compromised_By. A proposed Leverage field was cut as standing data — it is scene-local, and tracking it per NPC across six sagas is empty-cell maintenance. Only the 3–5 arc-active NPCs carry live Pressure/Next_Move values into the Session-Start Handshake; everyone else stays cold in the ledger.

**Cycles carry two halves.** Every cycle entry handed to a Gem carries a [Threat_Vector] *and* a [Player_Exploit_Window] — one concrete way the player can surf the cycle (cover, weapon, shortcut). Threat-only cycles are a lore lecture; the pair makes the cycle a tactical surface.

**Callbacks are pipeline-selected.** At session start the sweep selects 1–2 canon callbacks and delivers them in the handshake. Gems never browse their own corpus hunting for callbacks — left to pick their own, they repeat favorites and invent the rest.

**Natural stopping points.** Scenes may end at a low pulse. The forced-cliffhanger ending is named and banned as a default; tension is earned by state, not by cutting every scene on a gasp.

## Why this matters

This is applied AI systems work, not a toy demo:

- **Multi-agent orchestration** — the saga Gems, a Director layer, and the showrunner pipeline, coordinated through files rather than a message bus.
- **Long-context state management** — Markdown as the durable memory layer; delta-only updates; portable canon blocks that survive context resets.
- **Retrieval-augmented canon** — Drive → NotebookLM → Gem knowledge, with a one-command sync path that keeps the corpus identical to the source of truth.
- **Human-in-the-loop at every load-bearing decision** — audits, canon commits, and anything outbound all pass a human gate. The automation runs the world; it doesn't get the final word.

*Story content itself is private and stays local. This documents the system, not the fiction.*
