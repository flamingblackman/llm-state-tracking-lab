# Resume Bullets — LLM State-Tracking Lab

Project title for resume: **LLM State-Tracking Lab — Prompt-Engineered State Management** (Personal Project)

- Engineered a complex system prompt (Google Gemini custom Gem) with conditional routing, boundary constraints, and combat rulesets to sustain coherent state management across long-context tabletop RPG sessions
- Designed a structured Markdown state-output schema capturing entity stats, resource levels, and background event clocks, emitted as a mandatory log block at the end of every session cycle
- Built a human-in-the-loop validation pipeline: audited LLM-generated state logs for hallucination and state drift, committing only verified records to a NotebookLM retrieval corpus serving as the persistent source of truth
- Curated a retrieval corpus (world bible, mechanics reference) in NotebookLM to ground model outputs and mitigate context-window degradation, keeping the system prompt lean
- Implemented per-turn structured state blocks with open-loop tracking (every unresolved thread tagged ADVANCING or PARKED), replacing coarser session-end logs so drift has nowhere to hide between commits
- Added a model self-audit protocol: the Gem re-reads its own last state block every 5 turns and flags contradictions openly, layered underneath the human audit gate
- Designed a canon-compression ritual that distills full session state into a portable canon block every 15–20 turns, countering context-window degradation without losing load-bearing state
- Built a versioned-canon system (numbered Delta Log): world-changing decisions are recorded as deltas against a canon version, letting spinoff/sequel scenarios branch from a declared anchor without cross-contamination
- Extended the lab to a second test scenario (an original Dragon Ball saga) to validate the same state-tracking apparatus across genres

Skills line: Prompt Engineering · LLM State Management · RAG Tooling (NotebookLM) · Human-in-the-Loop QA · Markdown Schema Design
