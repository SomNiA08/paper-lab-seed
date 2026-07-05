# paper-lab-seed — RFP MAS 자산 보관소 (paper-lab 착공 전 씨앗)

2026-07-05 워크스페이스 피벗(VCT RFP 파트너십 → 아카데미/콘텐츠)으로 용도가 사라진
RFP MAS 인프라를 유저 레벨(`~/.claude/`)에서 그대로 이관한 것. 전역에 두면 모든 프로젝트의
이름 공간을 오염시키므로(예: 실제로 겪은 "동명 스킬 오발사" 실패류) 이 프로젝트 레벨 리포로
격리했다. 원본은 삭제 전 사용자 확인 대기 중 — 아직 `~/.claude/`에도 남아있다.

## 무엇이 들어있나

- `.claude/commands/` — `/rfp`, `/rfp-round-step` (2)
- `.claude/skills/` — `global-gate`·`graph-update`·`intake-auditor`·`rfp-round-loop`·
  `section-gate`·`wiki-ingest` (6)
- `.claude/agents/` — `ceo-brand-voice`·`devils-advocate`·`english-editor`·`intake-auditor`·
  `riot-evaluator`·`specialist-{commitment,community,business,competitive,operational}` (10)

## 이 자산의 다음 행선지: paper-lab

`somnia-hub/ROADMAP.md`의 대기열 6번. 근거 기반 심사 통과형(모양 C) 파이프라인 —
facts → 초안 → 레드팀 → 증거 경화 → 게이트 — 를 논문 작성용으로 재취업시킬 계획.

**이식 시 반드시 고칠 것**: 원본 RFP MAS는 R1~R6 고정 라운드 매트릭스를 쓰다가
게이트 실패 시 +2라운드 연장 → 그래도 실패하면 강제 통과라는 탈출구가 필요했다.
고정 라운드가 현실과 싸운다는 신호다. 논문 파이프라인은 **게이트가 통과를 선언할 때까지
도는 가변 라운드**로 재설계할 것 (harness-kit의 `/loop` + 합의 종료 패턴 참고).

**킷 v0.20 표준 위에 올릴 것**: 이 씨앗은 RFP MAS 계보(훅 없음·스킬 기반)라 킷 프리미티브가 없다 —
paper-lab 착공 시 킷 v0.20 표준 하네스(engine 3층·사건 장부·걸음 게이트·TOP-WALLS·MODELS)를 바닥으로
깔고 그 위에 이 자산(에이전트·스킬)을 가변 라운드로 재배선한다 (somnia-hub/ROADMAP.md 인프라 기반 참조).

팀명·지역(`Nongshim`/`Korea`)에 튜닝된 부분은 도메인 특화 예시이니, 논문 주제·분야에 맞게
에이전트 페르소나를 교체한다.
