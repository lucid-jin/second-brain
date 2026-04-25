---
domain: rtc
last_updated: 2026-04-26
---

# 📊 Metrics — RTC / 실시간 통신

## PR #2310 — PA mismatch 소리 유출 fix (머지 2026-04-14)

| 메트릭 | Before | After | Δ | 산출 |
|---|---|---|---|---|
| 평균 해소시간 | 1,400ms | 321ms | **-77% (4.4×)** | A/B (500ms 지연) |
| 최소 해소시간 | 1,300ms | 15ms | -99% (86×) | A/B |
| `pa_persistent_mismatch` per 1k active user | 645.55 | 577.11 | **-10.6%** | 프로덕션 8일 윈도우 |
| 머지 후 `=0`인 날 | — | 5일간 발생 | — | ClickHouse |

## PR #2367 — 카메라 publish race Mutex (ZEP-387, 머지 2026-04-21)

| 메트릭 | Before (3w) | After (4d) | Δ |
|---|---|---|---|
| 카메라 중복 publish unique users (평일 평균) | 306/day | 27.7/day | **-91%** |
| Dup buckets | 882/day | 68/day | **-92%** |

베이스라인: **1,371 users/week, 최악 22 tracks/session**

## PR #2378 — livekit 2.4.0 + setMetadata await (예상, 머지 미완)

- PA mismatch 모달 false-positive **41.8% 감소 예상**
- 머지 후 `SET_METADATA_FAILED` trace로 실측 가능 (인프라 이미 깔림)
- 검증값은 머지 후 1주 모니터링 시 업데이트

## 📝 갱신 이력

- 2026-04-26: 신규. PR #2310/#2367/#2378(예상) 통합.
