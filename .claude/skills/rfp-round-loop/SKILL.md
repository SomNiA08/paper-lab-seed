---
name: rfp-round-loop
description: 섹션 라운드 1개 진행 — 활성 에이전트 병렬 디스패치 + 산출물 수집
---

# RFP Round Loop 스킬

**언제 호출:** `commands/rfp-round-step.md`가 매 step마다 1회 호출. 본 스킬은 (section, round) 인자를 받아 활성 에이전트 디스패치를 수행한다.

## 디스패치 매트릭스

```python
DISPATCH = {
    1: ["specialist", "ceo-brand-voice"],     # R1 Draft
    2: ["riot-evaluator", "devils-advocate"], # R2 Red Team I
    3: ["specialist", "ceo-brand-voice"],     # R3 Vision Refine
    4: ["riot-evaluator", "devils-advocate"], # R4 Red Team II
    5: ["specialist", "intake-auditor"],      # R5 Evidence Hardening
    6: ["specialist", "devils-advocate"],     # R6 Final Polish (Devil last word)
}
```

## 절차

### 1. 활성 에이전트 산출
```
active = DISPATCH[round]
section_key = section  # "commitment", "community", "business", "competitive", "operational"
active = [f"specialist-{section_key}" if a == "specialist" else a for a in active]
```

### 2. 입력 컨텍스트 구성 (에이전트별)

각 에이전트에게 다음 입력을 동적 구성:

- `wiki/questions/q-{S}-*.md` (본 섹션 질문 페이지)
- `wiki/coverage.md`, `wiki/gaps.md`
- 직전 라운드 산출물:
  - R2·R4 (Red Team): R{prev}-specialist.md, R{prev}-ceo.md
  - R3·R5·R6 (Specialist 작업): R{prev}-redteam.md, R{prev}-devil.md
- `wiki/visions/` (CEO 페르소나 호출 시)
- `wiki/contradictions.md` (Devil 호출 시)
- `facts/` 전체 (Intake Auditor 호출 시)

### 3. 병렬 디스패치

Task tool을 사용하여 활성 에이전트를 **동일 메시지 내 병렬 호출**:

```python
for agent_name in active:
    Task(
      subagent_type=agent_name,
      description=f"Section {S} R{N} {agent_name}",
      prompt=build_prompt(agent_name, section, round, contexts)
    )
```

### 4. 산출물 수집

각 에이전트 산출:
- Specialist: `drafts/<section>/r{N}-specialist.md`
- CEO: `drafts/<section>/r{N}-ceo.md` (R1·R3), `rounds/final-u2-vision.md` (U2)
- Riot Evaluator: `scores/<section>/r{N}-redteam.md`
- Devil's Advocate: `scores/<section>/r{N}-devil.md`
- Intake Auditor: `wiki/evidence/<신규>.md` + 질문 페이지 업데이트
- English Editor: `final.md` (U3만)

### 5. 호출자에게 반환

호출자(rfp-round-step)에게:
```json
{
  "section": "commitment",
  "round": 3,
  "outputs": ["drafts/commitment/r3-specialist.md", "drafts/commitment/r3-ceo.md"],
  "summary": "R3 Vision Refine 완료. R2 감점 12건 중 10건 해소. 미해소 2건은 R5 갭 보강 대기."
}
```

## 에이전트 호출 프롬프트 빌더

각 에이전트의 `agents/<name>.md`에서 정의된 5블록 구조에 동적 컨텍스트 주입:

```
# 정체성
{frontmatter에서 description + body 정체성 블록}

# 입력 컨텍스트
- 현재 섹션: {section_name}
- 현재 라운드: R{N} ({라운드 이름})
- 본 섹션 질문 페이지: [목록]
- 직전 라운드 산출: {경로 목록}
- (해당되면) 누적 비전: [wiki/visions/* 목록]

# 임무
{agent.md의 "임무 (라운드별)" 블록 중 현재 라운드 항목}

# 출력 형식
{agent.md의 출력 형식 블록, 경로는 동적 치환}

# 행동 규칙
{agent.md의 행동 규칙 블록 그대로}
```

## 병렬 실행 주의

- 동일 메시지 내 Task 호출들은 병렬 — 서로의 출력 의존하면 안 됨
- 예: R1에서 Specialist와 CEO/Brand는 병렬 가능 (CEO는 톤 가이드만, Specialist 본문에 직접 의존 X)
- R3에서 Specialist와 CEO는 병렬 가능 — CEO가 Specialist 본문 받아 Tie-in 추가하지만, 별도 패스. 산출이 같은 파일에 쓰지 않도록 분리.

순차 처리가 필요한 경우 (예: U2 → U3) 는 rfp-round-step의 step 분리로 처리 (같은 step 내 순차 X).
