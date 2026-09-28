# Resume Bullets — LLM State-Tracking Lab

Project title for resume: **LLM State-Tracking Lab — Prompt-Engineered State Management** (Personal Project)

- Engineered a complex system prompt (Google Gemini custom Gem) with conditional routing, boundary constraints, and combat rulesets to sustain coherent state management across long-context tabletop RPG sessions
- Designed a structured Markdown state-output schema capturing entity stats, resource levels, and background event clocks, emitted as a mandatory log block at the end of every session cycle
- Built a human-in-the-loop validation pipeline: audited LLM-generated state logs for hallucination and state drift, committing only verified records to a NotebookLM retrieval corpus serving as the persistent source of truth
- Curated a retrieval corpus (world bible, mechanics reference) in NotebookLM to ground model outputs and mitigate context-window degradation, keeping the system prompt lean

Skills line: Prompt Engineering · LLM State Management · RAG Tooling (NotebookLM) · Human-in-the-Loop QA · Markdown Schema Design
