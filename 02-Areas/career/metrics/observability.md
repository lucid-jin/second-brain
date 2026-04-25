---
domain: observability
last_updated: 2026-04-26
---

# 📊 Metrics — Observability / Sentry

## PR #2179 + #2178 — Sentry fingerprint + Safari regex 버그 fix

| 메트릭 | Before | After | Δ |
|---|---|---|---|
| quiz-app Sentry events (22일 윈도우) | 34,864 | 20,472 | **-41%** |

차단된 누수 이슈:
- QUIZ-APP-38 (8,978 users)
- QUIZ-APP-X (220 users)
- 기타 10+

## 추가 작업

- PR #2384 — axios 에러 컨텍스트 + HTTP status 필터 (ZEP-394)

## 📝 갱신 이력

- 2026-04-26: 신규. quiz-app 한정 윈도우. play-app은 트래픽 변동 영향으로 인과 격리 어려워 제외.
