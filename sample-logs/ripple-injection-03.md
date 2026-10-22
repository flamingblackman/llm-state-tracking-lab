# Sample Log 03 — Guarded Ripple Injection + Drift-Scan Excerpt

*Sanitized sample. Faction names, places, and events are illustrative stand-ins, not material from any running line. The formats are the real ones, copied from the production protocol.*

## Part 1 — The injection, exactly as delivered

```text
[SHADOW DIRECTOR OVERRIDE]
Background only — do not advance the current scene or resolve any pending player decision.
Ripple: Word reaches the harbor district that the Compact's grain convoy was turned
back at the north gate. Prices tighten by morning. Quartermaster clock advances
4/6 → 5/6. No on-screen character knows the cause yet; treat as background
pressure only (market prices, crowd mood, guard alertness).
```

## Part 2 — How the target chat handled it

- Logged to background state under the Quartermaster clock entry, with the new position (5/6) and the trigger ("convoy turned back, rumor tier: confirmed by market effect").
- The live scene did not move. The player's pending decision (whether to sell the cargo or hold it) stayed open and unaddressed by the ripple.
- Background pressure surfaced only through narration texture the next time the scene touched the market: prices and mood, never the cause.
- Injection log entry: `target: line-2 · outcome: landed as background · scene touched: no · decision spent: no`.

For contrast, the pre-guard failure this format replaced: an unguarded version of the same event was once played as narration, the scene cut to the gate, and the player's pending sell/hold decision was resolved by the crowd's reaction before the player ever made it. That outcome is what the header and guard line exist to prevent.

## Part 3 — Drift-scan excerpt (same sweep)

| # | Check | Finding | Disposition |
|---|---|---|---|
| 1 | Clock consistency | Brief cited the Quartermaster clock at 4/6; the source-of-truth JSON said 5/6 after the ripple | FLAG: brief corrected to the JSON value; clocks are cited from the JSON, never from prose |
| 2 | Cross-line terminology | A term belonging to another line's setting appeared in this line's session text | FLAG: marked as contamination, proposed fix sent to the human; text not silently rewritten |
| 3 | Unearned reveal audit | A background NPC reacted to information sealed until a later reveal phase | FLAG: reaction struck from the archive draft; reveal ladder unchanged; noted for the next session brief |
| 4 | Voice ledger check | Two recurring NPCs matched their ledger entries (speech pattern, current want) | PASS: no action |

Drift scans flag and propose. They never rewrite canon on their own; every fix above waited for the human gate.
