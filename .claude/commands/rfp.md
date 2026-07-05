---
description: VCT Partnership RFP 응답 MAS 시작
argument-hint: <팀명> <지역>
---

# /rfp <팀명> <지역>

당신은 RFP MAS의 **Moderator**다. VCT Partnership Phase 2 Questionnaire 37개 질문에 대한 응답을 5섹션 × 6라운드 토론 사이클로 합성한다.

## 인자
- `$1` = 팀명 (예: "Nongshim Esports")
- `$2` = 지역 (예: "Korea", "NorthAmerica", "Brazil", ...)

지역은 RFP "Home Region Declaration" 리스트의 값만 허용. 잘못된 값이면 거부.

## 시작 절차

### 1. 세션 디렉토리 확인
`.rfp-session/` 존재 여부 체크.
- 이미 있으면 사용자에게 AskUserQuestion으로 선택받음: "이어서 진행 / 새로 시작 (백업 후 초기화) / 취소"
- "새로 시작" 선택 시 기존 디렉토리를 `.rfp-session.bak-<timestamp>/`로 이동 후 새로 생성.

### 2. facts/ 자료 확인
`.rfp-session/facts/` 하위에 자료가 있는지 확인. (rfp-mas/facts/ 시드가 있으면 복사)
- 비어있으면 사용자에게 안내:
  ```
  facts/ 아래에 자료를 넣어주세요:
    org/         — 조직도, 투자자, 재무, 모회사 관계
    community/   — SNS/팔로워/뷰 수치, 콘텐츠 링크, 워치파티 사진
    competitive/ — 대회 이력, 로스터, 코치 프로필
    ops/         — 시설, 일과, 컴플라이언스, 직원 복지
    academy/     — (있다면) 아카데미·교육 자료
  ```
  명령 종료. 사용자 재실행 대기.

### 3. state.json 초기화
```json
{
  "team_name": "$1",
  "territory": "$2",
  "started_at": "<ISO8601>",
  "current_section": 0,
  "current_round": 0,
  "max_round_per_section": 6,
  "section_queue": ["commitment","community","business","competitive","operational"],
  "gates_run": {},
  "gate_extensions_used": {"commitment":0,"community":0,"business":0,"competitive":0,"operational":0,"global":0},
  "wiki_dirty": false,
  "done": false
}
```

### 4. Round 0 — Intake Auditor 실행
`skills/intake-auditor.md` 호출. Task tool로 `intake-auditor` 에이전트 1회 디스패치.
산출: `wiki/coverage.md`, `wiki/gaps.md`, `wiki/questions/q-S-N.md` (37개), `wiki/evidence/<fileid>.md`.

### 5. Intake 요약 사용자에게 보고
```
[Round 0 — Intake 완료]
  facts/ 파일 수: N개
  37개 질문 매핑:
    VERIFIED N건, PARTIAL N건, MISSING N건
  High 우선 갭 (상위 5건):
    - q-X-Y: ...
    ...
```

### 6. 진행 여부 확인
AskUserQuestion: "갭 보강 후 진행 (facts/ 추가) / 그대로 진행 / 종료"
- "그대로 진행" 시: `state.current_section = 1`, `current_round = 0` 저장.
- 다음 단계 안내:
  ```
  ▶ 다음 명령으로 토론 루프를 시작하세요:
       /loop /rfp-round-step
  ```

## 출력 요구
- 항상 한국어로 보고
- 사용자 결정 지점에서는 AskUserQuestion 사용
- 종료 전 다음 액션을 명시
