---
domain: infra
last_updated: 2026-04-26
---

# 📊 Metrics — CI/CD & Build Infrastructure

## PR #2234 — GitHub Actions self-hosted 러너 전환 (머지 2026-04-15)

- 13개 워크플로우 19곳 → 사내 Docker 러너 8개 풀
- **연 ~$6,000 절감 예상** (월 ~$550, PR 본문 명시)
- 빌드 시간 단축/네트워크 격리/한도 회피 → PR 본문 미명시 (추정 안 함)

## PR #2304 + #2403 — dev 환경 통합 배포 파이프라인

- main 머지 → 6개 앱 자동 배포 + concurrency group
- core 앱 publish/dev → publish/dev-1 매트릭스 일관화
- `/deploy-dev` 스킬이 branch-sync 흡수 (+226/-96)

## PR #2393/2394/2395 — dev-N alias 인시던트 closed-loop (2026-04-23)

| 단계 | 시각 (UTC) | 누적 |
|---|---|---|
| #2393 머지 | 06:57Z | 0 |
| #2394 revert | 07:15Z | **18분** |
| #2395 forward fix | 08:06Z | **1h 8m** |

원인: `.next/cache` stale Tailwind JIT.
시그널: 가설 검증 실패 → 즉시 revert → 진짜 원인 재진단 → forward fix.

## PR #2275 — 공급망 보안

- 모든 CI `pnpm install`에 `--frozen-lockfile` 강제
- zus-publish 의도적 예외 1곳만 보존

## 📝 갱신 이력

- 2026-04-26: 신규. 8개 PR 통합. 인시던트 회복 시간은 git 머지 시각으로 산출.
