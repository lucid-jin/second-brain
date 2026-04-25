---
section: zep-devex
last_updated: 2026-04-26
---

# DevEx & 운영 자동화 플랫폼 (Monorepo / Design Tokens / Observability)

> ZEP > DevEx 담당

## 핵심 작업

- **분산된 프론트 코드베이스를 pnpm workspace 기반 모노레포로 이관** —
  5개 앱과 공통 패키지의 의존성/공통 모듈을 표준화, 변경 전파/릴리즈 플로우 단순화

- **내부 디자인 시스템 패키지(zus) 구축** —
  여러 앱의 UI 컴포넌트/토큰 통합 관리 (DESIGN.md AI-agent 레퍼런스 PR #2297, +802 LOC)

- **Figma 스킬로 디자인 토큰 자동 매칭률 80% 달성** —
  토큰 불일치 기반 DQA를 구조적으로 제거, 디자인-개발 커뮤니케이션 비용 감소

- **Orval 기반 OpenAPI 자동화** —
  타입 안전한 API Hook 자동 생성, 서버와의 불일치 최소화

- **Sentry 연동 공통 패키지 표준화** —
  - 태그/컨텍스트 규약 통일로 멀티앱 에러 필터링/분류/triage 속도 개선
  - API 에러 fingerprint 분류 + 노이즈 필터링 (PR #2179, +75 LOC)
  - Safari-only regex 버그 fix + 노이즈 패턴 23종 추가 (PR #2178, +577/-31)
  - 결과: quiz-app 22일 윈도우 Sentry events **34,864 → 20,472 (-41%)**, 누수 10+ 이슈 차단(QUIZ-APP-38 8,978u 포함)
  - axios 에러 컨텍스트 + HTTP status 필터 (PR #2384, ZEP-394)

- **NX 의존성 제거** (PR #2173, ZEP-346/347) —
  play-app + libs ~1,500 라인 라이브러리 이전, Vercel 워크플로우 8 → 1 통합

## 📝 갱신 이력

- 2026-04-26: 기존 Notion 항목 + Sentry 정량 임팩트 + NX 제거 추가.
