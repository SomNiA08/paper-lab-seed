---
description: Known pitfalls and failure log — append new entries when an approach fails
---

# Lessons Learned

The pitfalls below are extracted from the MAS design docs; the failure log starts empty because this workspace is new (created 2026-07-05).

## Known Pitfalls (from system design)
1. Parallel dispatch: agents launched in the same message must never write the same file — outputs are role-separated by path (`r{N}-specialist.md` vs `r{N}-ceo.md`).
2. Sequential dependencies (e.g. U2 → U3) must be separate `/rfp-round-step` invocations, never same-step sequential Task calls.
3. Verify cited `facts/` paths exist before a gate runs — section-gate does a filesystem check and one bad path fails the whole gate.
4. The same figure must match everywhere it appears (e.g. follower count in q-2-1 prose vs q-2-8 table) — the wiki contradiction lint blocks gates on mismatch.
5. Never invent facts to fill a weak answer — use `<!-- GAP: ... -->`; red-team rounds and gates specifically hunt fabrications.
6. `state.json` is the single source of truth for the round cursor — read it before each step, write it after; never advance the round in memory only.
7. Windows environment: skills use POSIX-style relative paths — run shell steps via Git Bash, not PowerShell, when following skill scripts.

## Failure Log
Append entries newest-first using the template below.

<!-- Template:
### YYYY-MM-DD — <short title>
- What failed:
- Root cause:
- Guardrail adopted:
-->
