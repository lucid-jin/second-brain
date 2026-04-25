---
title: Resume Enhancement Backlog
last_updated: 2026-04-26
---

# 이력 보강 백로그

## 🥇 TOP 3 (3개월 내, 가장 시간효율 높음)

### 1. RTC E2E 스모크 테스트 PoC #2385 마무리 머지 ⭐⭐⭐
- Linear: ZEP-393
- 현재: 드래프트 +442 LOC
- 임팩트: livekit/PA/Mutex 회귀 자동 방어 → 시간 지나면 "회귀 N건 사전 차단" 정량값 자동 누적
- 소요: 1~2주 (테스트 안정화)
- STAR 후보: "ZEP-393 RTC E2E smoke로 livekit·PA·Mutex 회귀 N건 사전 차단"

### 2. 블로그 1편 + `/sentry-noise-filter` OSS 공개 (병행) ⭐⭐⭐
- 블로그 주제: "PA mismatch 소리유출 4.4× 단축 — ClickHouse trace로 race condition 잡는 법"
  - 재료 99% PR #2310 본문에 있음
  - 채널: 본인 블로그 + Medium + velog + DEV.to 동시
  - 소요: 4~6시간
- OSS 후보: `/sentry-noise-filter` (가장 generic)
  - 사내 의존성 분리 + README + 데모 GIF
  - 소요: 1~2일
- 합쳐서 임팩트: 외부 가시성 0 → 1, GitHub 스타 누적 시작

### 3. CS 티켓 정량화 자동 리포트 ⭐⭐
- 채널톡 MCP로 본인 RTC fix(#2310/#2367/#2143) 머지 전후 키워드 카운트 자동화
- 소요: 4~6시간
- 임팩트: 비즈니스 메트릭 빈자리(이력서 가장 큰 약점) 채움

## 🥈 추가 후보

### 4. livekit 2.4.0 #2378 머지 후 production 검증
- 데이터 인프라(`SET_METADATA_FAILED` trace) 이미 깔려있음
- 머지 + 1주 모니터링 → "PA 모달 false-positive 41.8% → X%" 검증값 자동 생성

### 5. 컨퍼런스 발표 1건 신청
- 후보: FEConf, JSConf Korea, 인프콘, 토스 SLASH, AI 밋업
- 주제 후보:
  - "AI 도구 사용자에서 워크플로우 빌더로"
  - "RTC race condition 정량 디버깅"
  - "Claude Code Skill 5개로 디버깅 사이클 표준화"
- 소요: 자료 8~12시간 + 발표

### 6. `/code-review-lenses` 추가 OSS 공개
- 5-lens + 2-layer prompt caching 구조 공개
- AI 워크플로우 차별점 외부 검증

### 7. 자체 PostHog 이벤트로 비즈니스 임팩트 1개 만들기
- Quiz 또는 Play 한 화면 골라서 직접 짠 개선 → DAU/conversion 실측
- 가장 부족한 영역(비즈니스 메트릭) 풀 사례 1건 확보

## 📋 메모

- 위 모든 항목의 공통 목적: **외부 가시성 0 → 1**
- 메타 평가(meta-evaluation.md)에서 가장 큰 구조적 약점으로 지적된 항목들
- 분기마다 1~2개씩 닫는 것 권장 (욕심내면 유지 안 됨)

## 📝 갱신 이력

- 2026-04-26: 신규 작성. 7개 후보 정리.
