---
name: intake-auditor
description: Round 0 facts/ 인덱싱 + 37개 질문 사실 매핑 + 갭 식별. 또는 R5 갭 보강 보조.
---

# Intake Auditor 스킬

**언제 호출:** `/rfp` 진입점 Round 0, 또는 Section R5 (Evidence Hardening).

## Mode A: Round 0

1. `.rfp-session/facts/` glob — 모든 파일 (.md/.pdf/.csv/.xlsx/.json/.png) 리스트업
2. Task tool로 `intake-auditor` 에이전트 1회 디스패치:
   - input: facts 전체 + RFP 37문항 (아래 §질문 목록)
   - 산출:
     - `wiki/evidence/<fileid>.md` (파일별 요약)
     - `wiki/coverage.md` (37×사실 매핑표)
     - `wiki/gaps.md` (우선순위)
     - `wiki/questions/q-S-N.md` (37개 페이지 시드)
3. 결과를 Moderator가 받아 사용자에게 1쪽 요약 보고

## Mode B: Section R5 (Evidence Hardening 보조)

1. Specialist가 식별한 신규 갭 리스트를 input으로
2. `intake-auditor` 에이전트에게 "이 갭에 대해 facts/에 추가 인용 가능한 근거가 있는가?" 질문
3. 발견 시 해당 evidence 페이지 신규 작성 + 질문 페이지 업데이트
4. 미발견 시 "여전히 MISSING" 응답 → Specialist가 Open Q로 강등

## RFP 37문항 목록 (Section.Number)

```
[Commitment & Alignment — 5 questions]
1.1 VCT·VALORANT 투자 전략 + 2027+ 진화
1.2 VALORANT 지원 이니셔티브 + 성취
1.3 CSR + Game Changers 참여 + 2027+
1.4 오너십 그룹 기여 리소스
1.5 9.99%+ 투자자 표

[Community Resonance — 12 questions]
2.1 팬베이스 개요 (규모·인구·게임별 분포)
2.2 홈 지역 로컬 팬덤 성장 전략 + 2027+
2.3 지난 4년 홈 지역 VCT 이니셔티브 + 성취
2.4 팬 획득·인게이지먼트 전략 (SNS/콘텐츠/co-stream) + 2027+
2.5 Esports Digital Goods (Champions Bundle, Team Capsule) 프로모션
2.6 VALORANT in-game moment co-promotion 샘플 실행계획
2.7 팀 브랜드 표 (Name, Code, Style Guide, Trademark)
2.8 플랫폼 표 (IG/X/FB/YT/Twitch + Others)
2.9 12개월 비디오 콘텐츠 + 베스트 + 메트릭
2.10 12개월 라이브 스트리밍 활성화
2.11 12개월 co-streams
2.12 12개월 오프라인 워치파티

[Business Sustainability — 8 questions]
3.1 비즈니스 모델 + 2027+ 진화
3.2 2년 예상 예산 (선수 연봉·마케팅 등)
3.3 지분 펀딩 플랜
3.4 자본 표 (Capital/Amount/Source/Conditions)
3.5 부채 금융 (사용 시)
3.6 기타 장기 부채·리스·페이아웃
3.7 스폰서 표
3.8 선수 보상 표 (2024-2026)

[Competitive Plan — 7 questions]
4.1 4년 경쟁 이력
4.2 broader 경쟁 목표 → 로스터·디벨롭먼트·플레이스타일
4.3 경쟁 전략 (스카우팅·리크루팅·디벨롭먼트) + 2027+
4.4 신인·베테랑 디벨롭먼트 + 2027+
4.5 경쟁 스태핑·인프라 + 2027+
4.6 트레이닝·일과 운영 (도시·허브·거주)
4.7 커리어 이후 기회 + 직원 복지

[Operational Excellence — 5 questions]
5.1 조직도·운영 구조·의사결정
5.2 VCT 담당 팀·인력 (shared service %)
5.3 2027+ 생태계 변화 정렬 (구조·스태핑)
5.4 선수 conduct·discipline·integrity (정책·처리)
5.5 시설·이전·트레이닝 환경 업그레이드
```

## 출력 예시 (사용자 보고)

```
[Round 0 — Intake Auditor 완료]
  facts/ 파일: 47개
    org: 12  community: 18  competitive: 11  ops: 6
  37문항 매핑:
    VERIFIED 21건 (57%)
    PARTIAL  11건 (30%)
    MISSING   5건 (13%)
  High 우선 갭 (상위 5건):
    1. q-1-5 9.99%+ 투자자 표 — 법인등기부 등본 필요
    2. q-3-2 2년 예상 예산 — 재무팀 작성 필요
    3. q-3-8 선수 보상 표 2024-26 — 부분만
    4. q-1-3 Game Changers 참여 이력 — 자료 부재
    5. q-2-7 USPTO 상표 등록 — 자료 부재

  → wiki/coverage.md, wiki/gaps.md, wiki/questions/ (37개) 생성됨
```
