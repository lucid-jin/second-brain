---
section: zep-cicd
last_updated: 2026-04-26
status: NEW
---

# CI/CD & Build Infrastructure

> ZEP > 빌드/배포 파이프라인 오너십 (2026 Q1~Q2)

## 핵심 작업

- **GitHub Actions self-hosted 러너 일괄 전환** (PR #2234) —
  13개 워크플로우 19곳을 사내 Docker 러너 8개 풀로 전환,
  **연 ~$6,000 절감 예상** (월 ~$550, PR 본문 명시)
  - 빌드 시간 단축/네트워크 격리/한도 회피는 PR 본문 미명시

- **dev 환경 통합 배포 파이프라인** (PR #2304 + #2403) —
  main 머지 → 6개 앱 자동 배포 + concurrency group,
  core 앱 publish/dev → publish/dev-1 매트릭스 일관화,
  `/deploy-dev` 스킬이 branch-sync를 흡수해 단일 워크플로우로 통합 (+226/-96)

- **dev-N alias 인시던트 closed-loop 대응** (PR #2393/2394/2395, 2026-04-23) —
  - #2393 머지 06:57Z → #2394 revert 07:15Z (**18분**)
  - #2394 → #2395 forward fix 08:06Z (**51분**)
  - 전체 인시던트 회복: **1시간 8분**
  - 가설 검증 실패 시 즉시 revert + 진짜 원인(`.next/cache` stale Tailwind JIT) 재진단 → forward fix

- **공급망 보안** (PR #2275) —
  모든 CI `pnpm install`에 `--frozen-lockfile` 강제,
  zus-publish의 의도적 예외 1곳만 보존

## 부수 작업

- PR 머지 슬랙 알림 OAuth 이슈를 OpenRouter로 우회 (#2333)
- generated 타입 파일을 linguist-generated 마킹으로 PR 리뷰 친화성 개선 (#2236)
- 레거시 dev-7 자동배포 트리거 제거 (#2217)

## 작업 방식 (시그니처)

- **인프라 변경 + closed-loop 인시던트 회복력**: 변경이 사고를 유발하면 18분 내 revert, 51분 내 forward fix. 진짜 원인 재진단 후 닫기.
- 단발성 워크플로우 수정이 아니라 **(env, app) 매트릭스 일관화** 같은 구조적 정리

## 관련 PR

| PR | 머지 | 내용 | 임팩트 |
|----|------|------|--------|
| #2234 | 2026-04-15 | self-hosted 러너 전환 | ~$6,000/년 절감 (예상) |
| #2304 | 2026-04-14 | main 머지 시 dev 자동 배포 | 단계 통합 |
| #2403 | 2026-04-24 | /deploy-dev 통합 스킬 | +226/-96 |
| #2393 | 2026-04-23 | dev-N alias 갱신 | (인시던트 트리거) |
| #2394 | 2026-04-23 | revert #2393 | 18분 회복 |
| #2395 | 2026-04-23 | .next/cache 클리어 forward fix | 1h 8m 클로즈 |
| #2275 | 2026-04-04 | --frozen-lockfile 강제 | 공급망 보안 |
| #2333 | 2026-04-14 | 슬랙 알림 OpenRouter 전환 | — |

## 📝 갱신 이력

- 2026-04-26: 신규 작성. 8개 PR 통합. self-hosted 러너 비용 절감 추정값은 PR 본문 명시.
