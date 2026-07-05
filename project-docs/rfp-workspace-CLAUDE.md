# CLAUDE.md

This is a workspace for producing a **VCT (VALORANT Champions Tour) Partnership Phase 2 Questionnaire response** — 37 questions across 5 sections — via a Multi-Agent System (MAS) of debate rounds. The directory **starts empty**; all working artifacts appear under `.rfp-session/` at runtime after `/rfp` runs. The MAS infrastructure itself lives at USER level (`~/.claude/`), not in this repo.

Key framing facts:

- Agents are pre-tuned with **Nongshim Esports (Korea)** context (academy programs, KeSPA academy, partnerships), but team name and region are passed as arguments to `/rfp <team> <region>`.
- User-facing progress reports are written in **Korean**; the final RFP deliverable is **100% English**.
- There is no build system, no package manager, and no test suite — work is driven entirely through the `/rfp` and `/loop` commands below.

## Project Map

Runtime tree (appears only after `/rfp` runs):

```
.rfp-session/
├── state.json              # single source of truth: section/round cursor, gate extensions, done flag
├── facts/                  # user-supplied source material (INPUT — never fabricated)
│   ├── org/
│   ├── community/
│   ├── competitive/
│   ├── ops/
│   └── academy/
├── wiki/                   # compiled knowledge (Moderator-maintained core)
│   ├── coverage.md
│   ├── gaps.md
│   ├── contradictions.md
│   ├── index.md
│   ├── questions/          # q-S-N.md — 37, one page per question
│   ├── claims/
│   ├── evidence/
│   ├── risks/
│   ├── visions/
│   └── decisions/
├── drafts/<section>/       # r{N}-specialist.md, r{N}-ceo.md
├── scores/<section>/       # r{N}-redteam.md, r{N}-devil.md
├── rounds/                 # final-u1-cross.md, final-u2-vision.md
├── gates/                  # sec-<section>-r{N}.md, global.md
├── graph/                  # graph.json, graph.html, GRAPH_REPORT.md
└── final.md → {Territory}_{Team}_Application.md
```

Layering rule: `rounds/`, `drafts/`, and `scores/` form an immutable source layer; only the Moderator updates the `wiki/` core.

MAS component locations (user level):

| Component | Path |
|---|---|
| Entry command | `~/.claude/commands/rfp.md` (`/rfp <team> <region>`) |
| Round step | `~/.claude/commands/rfp-round-step.md` (driven by `/loop`) |
| Dispatch skill | `~/.claude/skills/rfp-round-loop/` |
| Intake/evidence | `~/.claude/skills/intake-auditor/` |
| Gates | `~/.claude/skills/section-gate/`, `~/.claude/skills/global-gate/` |
| Knowledge | `~/.claude/skills/wiki-ingest/`, `~/.claude/skills/graph-update/` |
| Agents (10) | `~/.claude/agents/*.md` |

## Workflow

1. **`/rfp <team> <region>`** — Moderator initializes `.rfp-session/` (backs up any existing session to `.rfp-session.bak-<timestamp>/` on restart), checks `facts/` for source material, writes `state.json`, and runs Round 0 (Intake Auditor maps all 37 questions to available facts → coverage / gaps / question pages).
2. User reviews the intake summary, optionally adds material to `facts/`, then starts the loop: **`/loop /rfp-round-step`**.
3. Each section runs rounds R1–R6 with a fixed dispatch matrix (two agents dispatched in parallel per round):
   - R1 Draft: `specialist-{section}` + `ceo-brand-voice`
   - R2 Red Team I: `riot-evaluator` + `devils-advocate`
   - R3 Vision Refine: `specialist` + `ceo-brand-voice`
   - R4 Red Team II: `riot-evaluator` + `devils-advocate`
   - R5 Evidence Hardening: `specialist` + `intake-auditor`
   - R6 Final Polish: `specialist` + `devils-advocate` (Devil has the last word)

   Between and after rounds:
   - After every round: `wiki-ingest` then `graph-update`.
   - After R2 / R4 / R6: `section-gate`.
   - A failed R6 gate → one +2-round extension (R7 / R8); a second failure → forced pass, with unresolved items demoted to Open Questions.
4. Section order: **commitment → community → business → competitive → operational**.
5. **Final Unification**: U1 cross-consistency (Moderator itself), U2 CEO vision sweep (`ceo-brand-voice`), U3 English translation (`english-editor`) → `final.md` → `global-gate`. Gate pass promotes the file to `{Territory}_{TeamName}_Application.md` (PascalCase team slug). Gate fail → re-run U2–U3 (max 2), then forced output with an Open Questions appendix.
6. PDF conversion is a separate step: the **`/make-pdf`** skill or pandoc.

## Agent Roster

Ten agents; the "Rounds" column lists when each is dispatched.

| Agent | Role | Rounds |
|---|---|---|
| `specialist-commitment` | Section 1 — Commitment & Alignment (5 Q) drafts | R1·R3·R5·R6 |
| `specialist-community` | Section 2 — Community Resonance (12 Q) drafts | R1·R3·R5·R6 |
| `specialist-business` | Section 3 — Business Sustainability (8 Q) drafts | R1·R3·R5·R6 |
| `specialist-competitive` | Section 4 — Competitive Plan (7 Q) drafts | R1·R3·R5·R6 |
| `specialist-operational` | Section 5 — Operational Excellence (5 Q) drafts | R1·R3·R5·R6 |
| `ceo-brand-voice` | CEO vision / brand-consistency guardian | R1·R3·Final U2 |
| `riot-evaluator` | Red-team scorer, 0–10 from Riot Selection Committee POV | R2·R4 |
| `devils-advocate` | Skeptic: hidden assumptions, contradictions, legal risk | R2·R4·R6 (last word) |
| `intake-auditor` | Facts indexing, 37-question fact mapping, gap priorities | Round 0·R5 |
| `english-editor` | Korean → RFP-tone English + Sources & Evidence Trace appendix | Final U3 only |

## Key Rules

- Drafts are written in **Korean**, 350–500 words per answer (±10%); final output is 100% English; report to the user in Korean.
- **Never fabricate facts**: every claim cites `facts/` or `wiki/evidence/`; unverifiable claims get `<!-- GAP: ... -->` markers or are demoted to Open Questions.
- Gate thresholds: Riot Evaluator average ≥ 8.0; Devil FINAL VETO count 0; god-node scores ≥ 7.5 (section) / ≥ 8.0 avg (global).
- No marketing clichés in drafts; no emphasis words (very / really / absolutely / incredibly) in English output.
- Parallel-dispatched agents must never write the same file.

Full detail lives in the rules files:

- **Execution model** (Advisor / Worker delegation policy): see `.claude/rules/execution-model.md`.
- **RFP writing & gate conventions**: see `.claude/rules/rfp-conventions.md`.
- **Known pitfalls & failure log**: see `.claude/rules/lessons-learned.md`.

## Environment

- Windows 11; **PowerShell** is the primary shell (Git Bash is available).
- This directory is **not a git repository** (yet).
- User-facing progress reports in Korean; final RFP deliverable 100% English.
- PDF export via the `/make-pdf` skill or pandoc.
