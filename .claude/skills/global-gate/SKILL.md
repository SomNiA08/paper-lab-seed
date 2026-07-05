---
name: global-gate
description: Final Unification U3 직후 최종 영문 출력의 적합성 판정. RFP 제출 직전 마지막 검문.
---

# Global Gate 스킬

**언제 호출:** Final Unification U3 (English Editor) 직후, `commands/rfp-round-step.md`에서 호출.

## 체크리스트

### 구조·포맷
- [ ] 5섹션 모두 영문 변환 완료
- [ ] 37문항 모두 개별 답변 (RFP "Individual Answers" 요구사항 — 결합 0건)
- [ ] 영문 헤더 정확 매핑:
  - `# Commitment & Alignment`
  - `# Community Resonance`
  - `# Business Sustainability`
  - `# Competitive Plan`
  - `# Operational Excellence`
- [ ] 질문 번호 RFP 원문과 일치 (`## 1.`, `## 2.`, ...)
- [ ] 영문 답변 길이 350~500 단어/답변 (오차 ±10%)
- [ ] 파일명 규칙: `{Territory}_{TeamName}_Application.md` (Territory_Team Name_Application.pdf 규칙 영문 변환)

### 표 항목
- [ ] q-1-5 투자자 표 (Investor Legal Entity Name, Ownership %, Short Profile)
- [ ] q-2-7 팀 브랜드 표 (Team Name, Team Code, Style Guide, Trademark)
- [ ] q-2-8 플랫폼 표 (IG/X/FB/YT/Twitch + Others) 모든 컬럼 채워짐 또는 Open Q
- [ ] q-3-4 자본 표
- [ ] q-3-7 스폰서 표
- [ ] q-3-8 선수 보상 표

### 콘텐츠 일관성
- [ ] Executive Vision Brief 헤더 1페이지 존재 (CEO U2 산출)
- [ ] 비전 메시지 5섹션 통일 (CEO U2 검증 통과)
- [ ] 5섹션 교차 모순 0건 (U1 Cross-Consistency 통과)

### 증거·점수
- [ ] god-node Question 평균 Riot Eval 점수 ≥ **8.0**
- [ ] `Sources & Evidence Trace` 부록에 모든 주요 수치·고유명사 매핑
- [ ] `wiki/gaps.md` High 항목 모두 해소 또는 Open Q 부록으로 명시 강등
- [ ] 영문 본문에 `<!-- GAP -->` 마커 노출 0건

### RFP 제출 규칙
- [ ] Language: 100% English (인용한 한국 고유명사는 로마자 + 한글 병기 허용)
- [ ] Standalone Document (외부 설명 없이 이해 가능)
- [ ] 영문 강조어 (very/really/absolutely/incredibly) 노출 0건

## 출력

`gates/global.md`

```markdown
# Global Gate — Final Application Review

## 판정: **PASS / FAIL (재실행 N/2)**

## 체크리스트 (4 카테고리)

### 구조·포맷 (7)
- [✓] 5섹션 영문화
- [✓] 37문항 개별
- ...

### 표 항목 (6)
...

### 콘텐츠 일관성 (3)
...

### 증거·점수 (4)
- [✓] god-node 평균 8.4 ≥ 8.0
- [✗] gaps.md High 미해소: q-1-3 Game Changers ❌

### RFP 제출 규칙 (3)
...

## 종합
- 통과 카테고리: 3/5
- 실패 항목: 2건

## 다음 액션
- gate_extensions_used.global = 0 → U2~U3 재실행
- 우선 처리: q-1-3 Game Changers 항목 — CEO가 Open Q로 강등 또는 사용자 자료 보강
```

## 의사코드

```python
def global_gate(state, wiki, graph, final_md):
    checks = []

    # 구조
    checks += check_structure(final_md)
    checks += check_tables(final_md)
    checks += check_filename(state, final_md)

    # 콘텐츠
    checks += check_executive_brief(final_md)
    checks += check_vision_unity(wiki.visions, final_md)
    checks += check_cross_consistency(final_md)

    # 증거
    god_scores = graph.god_node_average_score()
    checks.append(("god_node_score", god_scores >= 8.0))
    checks += check_sources_trace(final_md, wiki.evidence)
    checks += check_high_gaps_resolved(wiki.gaps)
    checks.append(("no_gap_markers", "<!-- GAP" not in final_md.body))

    # RFP 규칙
    checks += check_language_english(final_md)
    checks += check_standalone(final_md)
    checks += check_no_emphasis_words(final_md)

    passed = all(result for _, result in checks)

    if not passed and state.gate_extensions_used["global"] < 2:
        # U2~U3 재실행 (current_round = 7로 롤백)
        state.current_round = 7
        state.gate_extensions_used["global"] += 1
        action = "RE_RUN_U2_U3"
    elif not passed:
        # 강제 출력 + Open Q 부록
        attach_open_q_appendix(final_md, failed_items)
        state.done = True
        action = "FORCE_OUTPUT_WITH_OPEN_Q"
    else:
        # 통과 → 파일명 승격
        promote_filename(final_md, state.territory, state.team_name)
        state.done = True
        action = "PASS_AND_PROMOTE"

    write_gate_report(checks, action)
    return GateResult(passed=passed, action=action)
```

## 파일명 승격 규칙

```python
def promote_filename(final_md, territory, team_name):
    # "Nongshim Esports" → "NongshimEsports" (공백 제거, 영문 PascalCase)
    team_slug = "".join(w.capitalize() for w in team_name.replace("-", " ").split())
    territory_slug = territory.replace(" ", "")  # "Korea", "NorthAmerica"
    new_name = f"{territory_slug}_{team_slug}_Application.md"

    # final.md → {new_name} 이동
    # PDF 변환은 별도 도구 (gstack make-pdf 또는 pandoc)
    os.rename(".rfp-session/final.md", f".rfp-session/{new_name}")
```
