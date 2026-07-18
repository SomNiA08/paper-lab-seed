---
description: "[보관소 — 실행 금지] RFP MAS 라운드 1회 (paper-lab 착공 시 재설계 대상)"
---

> ⛔ **이 커맨드는 실행하지 마라 — paper-lab-seed는 보관소다** (README 최상단 벽 참조).
> 훅·SOUL·MODELS 없음 = 안전장치 0. 착공은 킷 스폰 절차로만.

# /rfp-round-step

당신은 RFP MAS의 **Moderator**. 한 번에 라운드 1개만 진행한다. `/loop`가 이 커맨드를 반복 호출한다.

## 의사코드

```
state = read(.rfp-session/state.json)

# ── 종료 조건 ──
if state.done:
    emit "STATE: done"; return

# ── Final Unification 라우팅 (current_section == 6) ──
if state.current_section == 6:
    N = state.current_round
    if N == 7:
        # U1 Cross-Consistency — Moderator 자체 수행 (서브에이전트 X)
        cross_check_5_sections()  # 모순/중복/누락 점검 → draft.md 갱신
        write rounds/final-u1-cross.md
    elif N == 8:
        # U2 CEO Vision Sweep
        dispatch_agent(ceo-brand-voice, mode="final_unification_u2",
                       input=draft.md + wiki/visions/)
        write rounds/final-u2-vision.md → draft.md 갱신
    elif N == 9:
        # U3 English Editor
        dispatch_agent(english-editor, input=draft.md + wiki/coverage.md)
        write final.md
        result = run skills/global-gate.md
        if result.passed:
            state.done = true
            promote final.md → "{territory}_{team}_Application.md"
        else:
            if state.gate_extensions_used.global < 2:
                state.current_round = 7   # U1부터 재실행
                state.gate_extensions_used.global += 1
            else:
                # 강제 출력 + Open Q 부록
                attach_open_questions_appendix(result.failed_items)
                state.done = true
    state.current_round = N + 1
    write_state(); emit_summary(); return

# ── 일반 섹션 라운드 ──
section = state.section_queue[state.current_section - 1]
N = state.current_round + 1   # 다음 라운드

# 디스패치 매트릭스
DISPATCH = {
  1: ["specialist", "ceo-brand-voice"],     # R1 Draft
  2: ["riot-evaluator", "devils-advocate"], # R2 Red Team I
  3: ["specialist", "ceo-brand-voice"],     # R3 Vision Refine
  4: ["riot-evaluator", "devils-advocate"], # R4 Red Team II
  5: ["specialist", "intake-auditor"],      # R5 Evidence Hardening
  6: ["specialist", "devils-advocate"],     # R6 Final Polish (Devil last word)
}
active = DISPATCH[N]
specialist_name = f"specialist-{section}"
active = [specialist_name if a == "specialist" else a for a in active]

# 병렬 디스패치 (skills/rfp-round-loop.md 사용)
outputs = parallel_dispatch(active, section=section, round=N)
# → rounds/sec-{S}-round-{N}-<role>.md 저장

# Wiki Ingest
run skills/wiki-ingest.md with outputs

# Graph Update
run skills/graph-update.md with outputs

# Section QA Gate (짝수 라운드 직후)
if N in [2, 4, 6]:
    gate_result = run skills/section-gate.md with section=section, round=N
    state.gates_run[f"{section}-r{N}"] = gate_result.passed
    if not gate_result.passed:
        if N == 6 and state.gate_extensions_used[section] < 1:
            state.max_round_per_section = 8  # +R7, R8 1회 연장
            state.gate_extensions_used[section] += 1
        elif N == 6:
            # 2회째 미통과 — 강제 통과, 잔존 항목 Open Q 강등
            demote_failed_items_to_open_q(gate_result.failed_items, section)

state.current_round = N

# 섹션 종료 → 다음 섹션
if N >= state.max_round_per_section and gate_passed:
    state.current_section += 1
    state.current_round = 0
    state.max_round_per_section = 6   # 다음 섹션은 다시 6
    if state.current_section > 5:
        state.current_section = 6     # Final Unification으로
        state.current_round = 6       # 다음 step에서 N=7부터

write_state()
emit_summary()
```

## 출력 형식
매 step 마지막에 사용자에게 1~2줄 요약:
```
[Sec {S} ({section_name}) · R{N}] {active_agents}
  → Riot Eval 평균: X.X  | Devil 적발: N건  | Open Q: N건
  Gate: PASS/FAIL (해당 시)
  다음: Sec {S+?} R{N+1} 또는 Final U{X}
```

`STATE: done` emit 시 `/loop` 자연 종료.
