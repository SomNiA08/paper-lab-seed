---
name: section-gate
description: 짝수 라운드(R2·R4·R6) 직후 섹션 통과 가능성 자동 판정.
---

# Section Gate 스킬

**언제 호출:** R2·R4·R6 직후 (총 섹션당 3회). `commands/rfp-round-step.md`에서 호출.

## 체크리스트

### 항상 (R2·R4·R6)
- [ ] 본 섹션의 모든 질문(q-S-1 ~ q-S-N)에 답변 페이지 존재 (빈 답·TBD·N/A만 있는 답변 0개)
- [ ] 직전 Riot Evaluator 채점 평균 ≥ **8.0**
- [ ] 인용된 모든 `facts/` 경로 실제 존재 (파일 시스템 검증)
- [ ] 본 섹션 관련 `wiki/contradictions.md` 미해결 항목 0
- [ ] 답변별 한국어 길이 350~500 단어 범위 (오차 ±10%)

### R6 전용 (Final Polish 후)
- [ ] Devil's Advocate FINAL VETO 0건
- [ ] god-node Question 점수 ≥ 7.5 (섹션 내 god-node가 있을 경우)
- [ ] 본 섹션 모든 답변에 `**Sources:**` 블록 존재
- [ ] 본 섹션 답변에 `<!-- GAP -->` 마커 0건 (또는 Open Q로 명시 강등)

### R2·R4 전용
- [ ] 채점된 질문 수 = 본 섹션 질문 수 (전수 채점 확인)

## 출력

`gates/sec-{section}-r{N}.md`

```markdown
# Section {section_name} — R{N} Gate

## 판정: **PASS / FAIL**

## 체크리스트 결과
- [✓] 모든 질문 답변 존재 (N/N)
- [✗] Riot Eval 평균 7.6 < 8.0 ❌
- [✓] facts/ 경로 검증
- [✗] contradictions.md 본 섹션 관련 1건 미해결 ❌
- [✓] 답변 길이 범위
- (R6) [✓] Devil FINAL VETO 0건

## 실패 항목 → 다음 라운드 의제
1. **Riot Eval 평균 미달 (7.6)** — q-2-4, q-2-8 점수가 6점 이하. 가장 큰 감점: Specificity (SNS 수치 출처 부재).
2. **모순 미해결**: q-2-1 "팔로워 22.9만" vs q-2-8 표 "25만". → wiki/contradictions.md#sec2-followers 해소 필요.

## 다음 액션
- 본 섹션 R+2 (연장) 실행 OR (R6 미통과 시) +2 라운드 연장 1회 발동
- 의제는 위 실패 항목 우선
```

## 의사코드

```python
def section_gate(section, round, state, wiki, graph):
    section_questions = list_questions(section)
    drafts = read_drafts(section, latest_round=round)
    scores = read_scores(section, latest_round=round)

    checks = []

    # 항상 검사
    checks.append(("all_questions_answered",
                   all(has_answer(d, q) for q in section_questions for d in drafts)))
    checks.append(("riot_eval_avg",
                   scores.riot_eval_avg >= 8.0))
    checks.append(("facts_paths_exist",
                   all(os.path.exists(p) for p in cited_paths(drafts))))
    checks.append(("no_contradictions",
                   not has_unresolved_contradictions(wiki, section)))
    checks.append(("word_count_range",
                   all(350*0.9 <= word_count(a) <= 500*1.1 for a in drafts.answers)))

    # R6 전용
    if round == 6:
        checks.append(("no_final_veto",
                       scores.devil_final_veto_count == 0))
        checks.append(("god_node_min_score",
                       graph.section_god_node_min_score(section) >= 7.5))
        checks.append(("sources_block_present",
                       all(has_sources_block(a) for a in drafts.answers)))
        checks.append(("no_gap_markers",
                       all(not has_gap_marker(a) for a in drafts.answers)))

    # R2/R4 전용
    if round in (2, 4):
        checks.append(("all_questions_scored",
                       len(scores.scored_questions) == len(section_questions)))

    passed = all(result for _, result in checks)

    write_gate_report(section, round, checks, passed)
    return GateResult(passed=passed,
                      failed_items=[name for name, ok in checks if not ok],
                      next_agenda=derive_agenda(scores, wiki))
```

## 연장 트리거

- R6 미통과 + 본 섹션 `gate_extensions_used` < 1
  → `state.max_round_per_section = 8`, R7·R8 추가 (R7 R3-스타일, R8 R6-스타일)
- 연장 2회째 미통과
  → 강제 통과 + 미충족 항목을 `wiki/decisions/forced-pass-{section}.md` 기록 + 답변에 Open Q 단락 흡수
