---
name: wiki-ingest
description: 매 라운드 산출물을 Karpathy llm-wiki 스타일로 인제스트. 질문·주장·증거·리스크·비전·결정 페이지 갱신 + 모순 린트.
---

# Wiki Ingest 스킬

**언제 호출:** 매 라운드 종료 직후, `commands/rfp-round-step.md`에서 호출.

## 철학 (Karpathy llm-wiki)

> 인간은 큐레이션, LLM은 유지보수. Wiki는 RAG처럼 매번 합성하지 않고 한 번 컴파일된 지식을 영속 유지한다.

본 시스템에서는 Moderator만 wiki/ 코어를 갱신. 라운드 산출물(`rounds/`, `drafts/`, `scores/`)은 source 계층으로 불변.

## 페이지 종류

| 디렉토리 | 페이지당 내용 |
|---|---|
| `wiki/questions/q-S-N.md` | RFP 37문항 각각. 원문 + 신뢰도 + 후보 근거 + 누적 답변 링크 |
| `wiki/claims/claim-<slug>.md` | 답변 내 검증 가능한 주장. supported-by/weakened-by 관계 |
| `wiki/evidence/<fileid>.md` | facts/ 파일별 요약 + 인용된 질문 역참조 |
| `wiki/risks/risk-<slug>.md` | Devil's Advocate가 제기한 리스크. 심각도·해소 상태 |
| `wiki/visions/vision-<slug>.md` | CEO가 제시한 비전·차별화 포인트. 매핑된 답변 역참조 |
| `wiki/decisions/decision-<round>.md` | 합의된 또는 강제된 의사결정. 라운드/근거 기록 |
| `wiki/contradictions.md` | 린트가 감지한 미해결 모순 목록 |
| `wiki/coverage.md` | 37×사실 매핑표 (Intake가 초기 생성, 매 라운드 갱신) |
| `wiki/gaps.md` | 미충족 갭 우선순위 |
| `wiki/index.md` | 모든 페이지 entry point + 카테고리별 목록 |

## 인제스트 절차

### Step 1: 산출물 분류
- Specialist 산출 → 새 Claim 추출 (각 답변 안의 검증 가능 문장)
- Riot Evaluator 산출 → 새 RedTeamScore 노드 (Graph에서 처리)
- Devil's Advocate 산출 → 새 Risk
- CEO 산출 → 새 Vision (또는 기존 vision 강화)
- Intake Auditor 산출 → Evidence 페이지 신규/갱신

### Step 2: 페이지 갱신
각 추출 항목에 대해:
1. 기존 페이지 있으면 병합 (라운드별 갱신 로그 추가)
2. 없으면 신규 생성
3. 교차참조 자동 추가 (`Related: [[q-S-N]] [[claim-slug]] [[evidence-fileid]]`)

### Step 3: 모순 린트
- 동일 슬롯에 충돌 값이 들어가면 `!CONTRADICTION` 마커 + `wiki/contradictions.md`에 기록
- 예: q-2-1 답변에서 "팔로워 22.9만"과 q-2-8 표에서 "팔로워 25만" 충돌 → contradiction

### Step 4: 고아 린트
- Evidence 없는 Claim → "근거 부족" 플래그 (다음 라운드 의제로 푸시)
- Risk가 어떤 Decision에도 연결되지 않음 → `wiki/gaps.md`의 "Open Question 후보"로 승격

### Step 5: index.md 갱신
- 신규 페이지 entry 추가
- 카테고리별 페이지 수 카운트

### Step 6: state.wiki_dirty = true → false 리셋
Moderator가 다음 라운드 컨텍스트 구성 시 wiki/에서 읽으므로 일관성 보장.

## 페이지 크기 제약

- 1 페이지 ≤ 400 라인. 초과 시 sub-page로 분할 (예: `q-2-8/platforms-table.md`)
- 페이지 frontmatter:
  ```yaml
  ---
  type: question | claim | evidence | risk | vision | decision
  created: <round>
  updated: <round>
  related: [[slug1]] [[slug2]]
  ---
  ```

## 출력 (호출자에게)

```json
{
  "pages_created": 3,
  "pages_updated": 12,
  "new_contradictions": 1,
  "orphan_claims": 2,
  "wiki_dirty_reset": true
}
```
