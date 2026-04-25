---
domain: unverified
last_updated: 2026-04-26
note: 이력서엔 빼는 게 안전. 면접에서 deep-dive 시 정성 사례로만 활용.
---

# ⚠️ Metrics — 측정 불가 / 약한 주장

이력서 외부 노출은 위험. 산출 근거가 약하거나 검증 불가.

## PR #2143 — PA guard 카메라 차단 fix

- **측정 불가**: ClickHouse retention이 2026-04-05 이후만 존재 → before/after 비교 불가
- **정성 사례**: 코딧 부트캠프에서 카메라 최대 3.6시간 영구 차단 → 머지 후 동일 패턴 후속 회차 0건
- 활용: 면접에서 "디버깅 사례 하나 자세히" 질문 시 사용

## PR #2378 — livekit 2.4.0 + setMetadata await

- **검증 불가**: 머지 안 됨 → production A/B 불가
- "PA 모달 false-positive 41.8% 감소"는 추정값
- 활용: 머지 후 1주 모니터링 시 `metrics/rtc.md`로 이동

## PR #2235 — code-review-lenses skill (+2,177 LOC)

- **토큰 -48% 자체 A/B 부재** (외부 reference만)
- "PR #2147에서 20+ 구조 이슈 발굴" → 헤딩 47개 ≠ 실제 이슈 수
- 활용: 정성 사례로만. "5-lens + 2-layer prompt caching 구조" 같은 설계 측면만 강조.

## play-app Sentry events

- play-app 22일 윈도우 events 변동 +67%
- 트래픽/instrumentation 변동 혼입으로 **인과 격리 불가**
- 활용: 절대 인용 금지. quiz-app 한정 -41%만 사용.

## 📝 갱신 이력

- 2026-04-26: 신규. 검증 안 된 항목 분리 보존.
