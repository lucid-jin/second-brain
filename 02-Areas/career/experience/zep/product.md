---
section: zep-product
last_updated: 2026-04-26
---

# 프로덕트 개발 (어드민 / 마이그레이션)

> ZEP > B2E 제품 개발

## 핵심 작업

- **레거시 마이그레이션 TF 리드** (WebRTC·Phaser 게임 엔진 코드베이스):
  - play-app + libs **NX 의존성 제거** (~1,500 라인 라이브러리 이전)로 빌드 아키텍처 단순화 및 Vercel 워크플로우 8 → 1 통합 (ZEP-346/347, PR #2173)
  - WebRTC·Phaser 게임 엔진 race·라이프사이클 이슈를 **진단 PR 선행 + 훅 단위 재설계 패턴**으로 정리 (예: `useGameConnection` 훅, ZEP-408)
  - 산재된 스펙 문서화 + 모듈 간 결합도 감소

- **내부 어드민 TF 리드** —
  Spring Boot + Next.js 기반 풀스택 설계·개발

## 📝 갱신 이력

- 2026-04-26: NX 제거(#2173)와 useGameConnection 훅 도입(ZEP-408) 구체 사례 추가. 기존 Notion의 추상적 표현("기술 스택 현대화") 보강.
