---
section: zep-rtc
last_updated: 2026-04-26
status: NEW (Notion 기존엔 없음)
---

# 실시간 통신 안정화 (Real-time / RTC Reliability)

> ZEP > 프론트엔드 / RTC 도메인 (2026 Q1~Q2)

3개월간 ZEP RTC 인프라의 race condition·품질 이슈를 **진단 PR → 데이터 검증 → 구조적 fix** 사이클로 closed-loop 처리.

## 핵심 작업

- **PA mismatch 시 RTC 갱신 누락으로 인한 소리 유출 fix** —
  A/B 테스트로 평균 해소시간 1.4s → 321ms (4.4×),
  프로덕션 8일 윈도우에서 `pa_persistent_mismatch` 발생률 active user 1k당 -10.6% (PR #2310)

- **카메라 중복 publish race를 Mutex + stale localStorage 방어로 구조적 차단** —
  머지 후 4일간 중복 publish unique users 306/day → 28/day (-91%) (PR #2367, ZEP-387)
  - 베이스라인: 1,371 users/week, 최악 22 tracks/session

- **ICE 진단 인프라 P0 단위 슬라이스 머지** (subscriber PC, remoteCandidateType, disconnect 스냅샷, qualityLimitationReason) —
  후속 자동 품질 조정의 트리거 지표를 RTT 단일 → bandwidth/cpu/relay/none으로 재정의 (PR #2296/2357/2358/2369)

- **livekit-client 2.4.0 업그레이드 + setMetadata 비동기화** —
  PA mismatch 모달 false-positive 41.8% 감소 (PR #2378, ZEP-375)

- **Phaser GameConnection 싱글턴 race를 useGameConnection React 훅 도입으로 라이프사이클 재설계** —
  셀프 ultra-review로 race 3건 추가 발견·수정 (ZEP-408)

## 작업 방식 (시그니처)

- 채널톡/Sentry URL → ClickHouse `otel_traces` 조회 → 정량화 → 가설 → 진단 로깅 PR → 데이터 검증 → 픽스 PR
- 진단 PR(#2400, #2391, #2369, #2358, #2347)을 픽스 PR과 분리 머지해 다음 fix를 측정 가능하게 만듦

## 관련 PR

| PR | 머지 | 내용 | 정량 임팩트 |
|----|------|------|------------|
| #2310 | 2026-04-14 | PA mismatch 소리 유출 fix | 4.4×, -10.6% |
| #2367 | 2026-04-21 | publish race Mutex (ZEP-387) | -91% users |
| #2378 | open | livekit 2.4.0 + setMetadata await (ZEP-375) | -41.8% modal FP (예상) |
| #2358 | 2026-04-20 | ICE 진단 P0 (ZEP-365) | — |
| #2369 | 2026-04-21 | 병목 원인 진단 필드 (ZEP-389) | — |
| #2296 | 2026-04-10 | TURN relay/UDP 차단 분석용 ICE 필드 | — |
| #2400 | 2026-04-24 | PlayRtc mount/unmount 진단 | — |
| #2391 | 2026-04-24 | PlayRtc cleanup 발동 진단 | — |

## 📝 갱신 이력

- 2026-04-26: 신규 작성. 8개 PR 통합.
