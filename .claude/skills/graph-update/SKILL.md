---
name: graph-update
description: graphify 스타일 지식 그래프 갱신. Question/Claim/Evidence/Risk/Vision/RedTeamScore 노드 + 신뢰도 태깅 엣지.
---

# Graph Update 스킬

**언제 호출:** Wiki Ingest 직후, 매 라운드.

## 노드 유형

| 노드 | 출처 | 핵심 속성 |
|---|---|---|
| `Question` | wiki/questions/ | section, number, current_confidence, current_score (god-node 후보) |
| `Claim` | Specialist 답변에서 추출 | text, asserted_in_rounds, status (active/retracted) |
| `Evidence` | facts/ 파일 인용 | path, page/sheet, verifiability (DIRECT/INFERRED) |
| `Risk` | Devil's Advocate 산출 | severity (High/Med/Low), addressed_in_rounds, status |
| `RedTeamScore` | Riot Evaluator 산출 | section, round, question, score, deductions |
| `Vision` | CEO 산출 | slug, sweep_status (independent/unified) |
| `Decision` | 합의·강제 결정 | round, rationale, dissenting_voices |

## 엣지 유형

- `Claim —SUPPORTED_BY→ Evidence` (신뢰도 태그: DIRECT_QUOTE / DERIVED / PARAPHRASE)
- `Claim —WEAKENED_BY→ Risk`
- `Question —ANSWERED_BY→ Claim`
- `RedTeamScore —APPLIES_TO→ Claim`
- `Vision —EMBODIED_IN→ Claim`
- `Decision —RESOLVES→ Risk`
- `Decision —SUPERSEDES→ Claim` (이전 주장 폐기 시)
- 신뢰도 태깅 (graphify 채택): `EXTRACTED` / `INFERRED` / `AMBIGUOUS`

## 절차

### Step 1: 신규 노드 추출
Wiki Ingest가 만든 페이지로부터 graph 노드 추출. 페이지 frontmatter `type:` 기준 자동 분류.

### Step 2: 엣지 생성
- 페이지 본문의 `[[slug]]` 링크 → 엣지
- `**Sources:**` 블록 → Claim → Evidence 엣지
- Devil 산출의 인용 → Risk → Claim (WEAKENED_BY)
- Riot Eval의 채점 → RedTeamScore → Claim (APPLIES_TO)

### Step 3: god-node 재계산
god-node = 가장 많은 Claim·Evidence가 모이는 Question.
- 본 시스템에서 god-node 후보는 37개 Question 전체
- 매 라운드 god-node degree 갱신
- Top-K (예: K=5) god-nodes의 평균 RedTeamScore = "답변 임팩트 핵심 지표"

### Step 4: 고아 탐지
- Evidence 없는 Claim → `orphan_claims/<slug>.md`에 플래그
- Decision에 연결 안 된 Risk → `wiki/gaps.md`의 "Open Question 후보"

### Step 5: 출력 3종 (graphify 포맷)

```
graph/
├── graph.json    # 노드·엣지 표준 JSON (Cytoscape.js·D3 호환)
├── graph.html    # 인터랙티브 시각화 (god-node 하이라이트, 카테고리별 색상)
└── GRAPH_REPORT.md  # 사람이 읽는 요약
```

#### graph.json 스키마
```json
{
  "nodes": [
    {"id": "q-1-1", "type": "Question", "section": "commitment", "score": 7.8, "degree": 12},
    {"id": "claim-academy-pipeline", "type": "Claim", "round": 3, "status": "active"},
    {"id": "ev-academy-promo", "type": "Evidence", "path": "facts/academy/promo-deck.pdf"}
  ],
  "edges": [
    {"source": "q-1-1", "target": "claim-academy-pipeline", "type": "ANSWERED_BY", "confidence": "EXTRACTED"},
    {"source": "claim-academy-pipeline", "target": "ev-academy-promo", "type": "SUPPORTED_BY", "confidence": "DIRECT_QUOTE"}
  ],
  "god_nodes": ["q-2-1", "q-3-2", "q-4-1"],
  "metadata": {
    "round": 7,
    "total_nodes": 156,
    "total_edges": 312,
    "orphan_claims": 4,
    "unresolved_risks": 6,
    "average_god_node_score": 8.2
  }
}
```

#### GRAPH_REPORT.md 예시
```markdown
# Graph Report — Section 1 R3

## Overview
- 노드: 156 (Q 37, Claim 48, Evidence 32, Risk 18, Vision 7, Score 14)
- 엣지: 312
- 평균 god-node 점수: 8.2 / 10 (R2 6.4 대비 +1.8)

## god-nodes (Top-5)
1. q-1-1 (degree 14, score 8.4) — 투자 전략
2. q-2-1 (degree 13, score 7.9) — 팬베이스 개요
3. ...

## 고아 Claim (4건)
- claim-japan-academy-mou — Evidence 부재. R5에서 facts/academy/japan-mou.md 확인 필요
...

## 미해결 Risk (6건)
- risk-game-changers-absence (High) — 어떤 Decision에도 연결 안 됨
...
```

## 출력 (호출자에게)

```json
{
  "graph_round": 7,
  "nodes_added": 14,
  "edges_added": 32,
  "god_node_avg_score": 8.2,
  "orphan_claims": 4,
  "unresolved_risks": 6
}
```
