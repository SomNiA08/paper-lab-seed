---
description: Writing, evidence, and gate conventions for the VCT RFP response
paths:
  - ".rfp-session/**"
---

# RFP Conventions

## Language
- Drafts: Korean, 350–500 words per answer (±10% tolerance — gates check this).
- Final deliverable: 100% English; Korean proper nouns may be romanized with hangul in parentheses.
- English output must contain zero emphasis words: very, really, absolutely, incredibly.
- Progress reports to the user: Korean.

## Evidence
- Never fabricate numbers, clients, tournament results, or partners. Every claim cites `facts/` or `wiki/evidence/` paths.
- Unverifiable claims: mark `<!-- GAP: ... -->` or demote to an Open Questions appendix.
- Cited `facts/` paths are filesystem-verified by gates — a typo fails the gate.
- Every final answer needs a `**Sources:**` block.

## Gates & thresholds
- Section gate (runs after R2/R4/R6): all questions answered, Riot Evaluator average ≥ 8.0, cited paths exist, zero unresolved contradictions for the section, word-count range respected.
- R6 additionally: Devil's Advocate FINAL VETO count 0, god-node question score ≥ 7.5, Sources blocks present, zero `<!-- GAP -->` markers (or explicitly demoted).
- Global gate (after U3): 37 individual answers (no merged answers), exact English section headers, required tables (investors q-1-5, brand q-2-7, platforms q-2-8, capital q-3-4, sponsors q-3-7, player comp q-3-8), god-node average ≥ 8.0, Executive Vision Brief page present.
- Extensions: failed R6 → one +2-round extension (R7/R8); second failure → forced pass, failures demoted to Open Questions. Failed global gate → re-run U2–U3 (max 2), then forced output with Open Questions appendix.

## Naming & structure
- Final file: `{Territory}_{TeamName}_Application.md` — territory and PascalCase team slug, spaces removed (e.g. `Korea_NongshimEsports_Application.md`).
- Wiki pages ≤ 400 lines; split into sub-pages beyond that. Page frontmatter: type/created/updated/related.
- `rounds/`, `drafts/`, `scores/` are an immutable source layer; only the Moderator updates `wiki/` core files.

## Style
- No marketing clichés in drafts (최고의, 혁신적인, 선도하는, 압도적인 — delete on sight; replace with numbers, proper nouns, years).
- 2027+ plans must be KPI-formatted ("which KPI to X by year N"), never bare intent ("we will strengthen").
