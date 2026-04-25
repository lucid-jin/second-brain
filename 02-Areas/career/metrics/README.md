---
title: Metrics Index
last_updated: 2026-04-26
---

# 📊 Metrics

정량 임팩트 전용. 도메인별 분리.

## 파일

- [`rtc.md`](rtc.md) — 실시간 통신 (PA mismatch, race condition)
- [`observability.md`](observability.md) — Sentry 노이즈 필터링
- [`infra.md`](infra.md) — CI/CD, self-hosted 러너, 인시던트 회복
- [`product.md`](product.md) — Quiz SEO, web vitals, 디자인시스템
- [`unverified.md`](unverified.md) — 측정 불가 / 약한 주장 (이력서 빼는 게 안전)

## 룰

- 검증된 숫자만. 추정은 명시(`(예상)`, `(추정)`)
- PR 머지 후 즉시 추가 → 1주 후 production 검증값으로 업데이트
- 측정 윈도우(N일)와 산출 방법(A/B vs 프로덕션) 명시
- 출처 PR/Linear 링크 필수

## 📝 갱신 이력

- 2026-04-26: 신규 작성. 5개 도메인으로 분리.
