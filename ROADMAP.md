# Roadmap

This page is the public plan for the LLM State-Tracking Lab. It separates what already runs from what is designed, planned, or proposed, and it sequences the work in dependency order.

The lab has two goals that share one core pattern:

1. Keep long-running narrative state correct across sessions.
2. Prove the same pattern on business-style records, so the work reads as operations and automation evidence, not only as a creative project.

The pattern in both cases: **ingest, normalize, validate, reconcile, route, commit, report.** Messy input goes in one end. Validated records, routed exceptions, and an audit trail come out the other.

## Rules for this roadmap

- Status labels are literal. Implemented means running. Planned means sequenced. Proposed means the design is ratified and the build has not started.
- Nothing becomes an official record without human approval. Uncertain or high-impact changes route to a person instead of guessing.
- AI-assisted work is labeled as AI-assisted. Code and copy in this repo do not claim hand authorship they do not have.
- No metric is published before it is measured. Test results will report denominators, pass rates, and failures, not only wins.
- Private story content stays local. This repo documents the system, never the fiction, and the business demo uses fictional records only.
- NotebookLM provides the retrieval layer (embeddings, chunking, search). The lab designs the corpus and the retrieval strategy; it does not claim to have built a retrieval engine.

## What already runs

| Component | Status |
|---|---|
| Per-turn state contract, self-audit turns, human audit gate | Implemented |
| Canon sync (Markdown to Drive to NotebookLM to Gems) | Implemented |
| Showrunner sweep: archive, world moves, drift scan, briefs | Implemented, session-close and on-request triggers |
| Shadow Director guarded injections, with an injection log | Implemented |
| Faction dashboards (clock JSON rendered to a Sheets view) | Implemented |
| API tandem line behind a human QC gate | Implemented |

## Phase 1: Item 0, migration packaging

**Status: Planned. Build step 1.**

The Gems platform migrates to Skills on November 17, 2026. Everything else on this roadmap assumes the sagas survive that move, so this phase comes first.

- Instruction blocks for all six Gems exported to local, versioned backups by **October 31, 2026**.
- Compatibility test by **November 7, 2026**.
- Markdown-bundle fallback validated in a fresh session by **November 12, 2026**. A backup file is not a working fallback until it has been run.
- Freeze and verify **November 15, 2026**. Platform migration from **November 17, 2026**. Verification pass **November 18, 2026**.
- Every pending Gem-instruction addition (Sensory Cold Open, fun-layer fields, natural stopping points, callback acceptance through the handshake, state-block echo) goes in through one edit window per Gem, so the October 31 backups capture the improved text.

## Phase 2: Reference-saga validation (JJK: The Hollow Vow)

**Status: Planned. Build steps 2 and 3.**

Formal validation concentrates on one reference implementation instead of spreading thin across six sagas. The Hollow Vow was selected because it has active faction clocks, recurring NPCs, permanent on-page consequences, enough canon to test retrieval, and the system's strongest sealed-knowledge case. Cursed Resonance is the designated partner for bounded cross-saga integration tests.

- **Canon Test Suite, dirty baseline first.** Four test classes: exact state, canon conflicts, stale-source retrieval, and premature secret disclosure, including indirect-leakage probes. The first run goes against the corpus exactly as it stands. Failures are recorded, not hidden.
- **File-level corpus quarantine.** Superseded documents leave the active source set. After quarantine, the suite runs again and the clean result is recorded next to the dirty baseline.
- **Honesty bound:** improvement on this suite demonstrates improvement on these tests. It is not a claim of universal retrieval accuracy.

## Phase 3: Proof Pack and Session-Start State Handshake

**Status: Planned. Build steps 4 and 5.**

- **Proof Pack.** A swimlane that keeps deterministic code and LLM-in-the-loop work in separate columns. One redacted end-to-end diff artifact: raw input, validation output, the exact state change, the human decision, and the resulting report. The before and after quarantine scores from Phase 2.
- **Session-Start State Handshake.** A state card injected at session start: superseding header, explicit diffs, pipeline-selected callbacks, cycle state, and a world-beat budget (one major beat, up to two minor) that caps what gets promoted for dramatization. Ledger clock ticks stay uncapped; the budget governs the narrative surface only.

## Phase 4: Deterministic parser under the State Reconciliation Contract

**Status: Proposed. Contract adopted October 23, 2026; build not started.**

This is the phase that turns the lab's rules into code, and the phase the business demo in Phase 5 is extracted from.

- Standardize the reference saga's per-turn state block as part of reference setup.
- Build a deterministic pre-sweep parser: it reads captured session text, extracts the per-turn state blocks, diffs first against last, matches NPC names against the voice ledgers, and hands the model a pre-digested delta instead of a raw transcript.
- Normalize each update into a common record: saga, session, turn, base-state version, proposed changes, unresolved decisions, audit status.
- Apply the State Reconciliation Contract to every proposed change:
  - Every delta carries a unique ID and the base-state version it was computed against.
  - Re-applying an accepted delta is a no-op. The same update never lands twice.
  - A delta built on a stale base version is flagged, never silently applied.
  - Conflicting values are preserved as a conflict and routed for resolution. Newest does not win by default.
  - Malformed or ambiguous input goes to an exception queue. The parser rejects it; it does not guess.
- The extraction and validation code is deterministic for supported inputs. Epistemic typing ([FACT] versus [BELIEF: holder]) and the Epistemic Promotion Audit land with this phase's drift-scan work.

Acceptance tests for this phase: replay one delta twice and show a single commit; submit a stale-base delta and show the flag; submit conflicting values and show the preserved conflict; submit a malformed block and show the exception queue entry.

## Phase 5: Business reconciliation demo (fictional CRM-style records)

**Status: Proposed. Approved in principle October 23, 2026; career-team review of sequencing and framing requested the same day.**

Dependency: Phase 4 must demonstrate the reconciliation contract on reference-saga session text first. This demo is an extraction of that working engine, not a separate invention.

The point of this phase is transfer evidence. The story system proves the pattern on narrative state. This demo runs the same pattern on records a revenue operations team would recognize, so a hiring manager does not have to take the mapping on faith.

- Build a fictional dataset of 30 to 50 update events against fictional lead, account, and opportunity records. No real customer data, no real company data.
- Define the expected outcome for every event before the run. The test set deliberately includes:
  - Valid updates, which accept or queue for the defined approval step.
  - The same event replayed twice, which must not apply twice.
  - Updates based on stale data, which must flag for review.
  - Conflicting field values, which must preserve the conflict and request resolution.
  - Missing required fields, which route to the exception queue.
  - Unsupported factual claims, which flag instead of promoting silently.
  - Approved updates, which apply and record the change.
  - Rejected updates, which leave the original record untouched.
- Publish the evaluation either way: events tested, validation pass rate, duplicate handling, exception detection, manual reviews required, and the known failures with their causes.
- When built, the demo adds clearly labeled repository areas: `workflow/` (process definition, validation rules, approval gates), `data/` (fictional inputs, baseline records, expected outputs), `tests/` (cases and assertions), `evaluation/` (measured results), and `docs/demo/` (a walkthrough of one input, one conflict, one approval decision, and the resulting audit record).
- No CRM integration is claimed. The prototype runs on local fictional records unless a later, labeled update says otherwise.

## Phase 6: Living-world mechanics

**Status: Planned. Build steps 6 and 7.**

- **Base faction rule v2, LW-3 (Debt and Favor), LW-4 (Grudge Drift).** Every faction is evaluated each sweep. It acts only when goal, capability, trigger, and material effect justify it; otherwise the no-action reason is recorded.
- **LW-1 (Scavenger Cascade)**, capped at one active node per saga.
- **LW-2 (Weaponized Rumor Feedback)**, last in the queue, decision closed October 23, 2026: keep, under conditions. A rumor must persist through two qualifying sessions in its source saga; it arrives in the target saga as a pending false-belief event the player can accept, reject, or defer; one controlled seeded test runs before the mechanic is trusted; it is cut if it ever needs frequent manual sync.
- The fun layer (Forked Consequence engine, voice-ledger merge of Core Want and Default Next Move, cycle Threat Vector plus Player Exploit Window, natural stopping points) is designed and ratified. Its Gem-instruction pieces ship through the Phase 1 edit window; its pipeline pieces build after the correctness phases above.

## Phase 7: Portfolio presentation

**Status: Gated on measured proof.**

Only after Phase 5 publishes real results:

- Update the LinkedIn project description and resume bullets using the business framing: a human-in-the-loop automation system that turns messy inputs into validated records, detects conflicts, routes exceptions, and keeps an audit trail.
- Present the two views of one system side by side: the story lab as the real, long-running use case, and the CRM-style demo as the labeled transfer evidence.
- The roles this speaks to, with different emphasis for each: Automation Specialist (triggers, validation, exception routing, duplicate prevention), RevOps and Sales Operations (record hygiene, reconciliation, reporting), Business Systems and Operations Analyst (requirements, versioning, test cases, governance).

## Sequence in one view

Migration safety first, because every later phase depends on the sagas surviving November 17. Reference-saga validation next, because parser claims need a tested baseline. Parser before the business demo, because the demo is an extraction of the parser's engine. Measured results before portfolio claims, always.
