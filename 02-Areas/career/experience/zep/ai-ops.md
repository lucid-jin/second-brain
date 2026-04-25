---
section: zep-ai-ops
last_updated: 2026-04-26
---

# AI 워크플로우 / Claude Code Skills / MCP

> ZEP > DevEx 겸직 / AI 도구 자체 설계·운영

## 핵심 작업

- **반복 운영·디버깅 작업을 5~6개 Claude Code Skill로 추상화해 팀 워크플로우 표준화**:
    - `/sentry-noise-filter` — Sentry URL → 룰 중복 체크 → 패턴 등록 → PR 자동 생성 (PR #2384)
    - `/deploy-dev` — dev-N 배포·branch-sync·PR preview 재사용을 단일 워크플로우로 통합 (PR #2403, +226/-96)
    - `/rtc` — RTC 통합 진단 (품질 리포트, PA 코드 진단, ClickHouse 추적)
    - `/translate` — i18n 번역 시트 자동화 (PR #2198, #2240)
    - `/code-review-lenses` — 5개 관점(consistency·type-safety·reusability·extensibility·elegance) 병렬 + prompt caching 코드리뷰 (PR #2235, +2,177 LOC)
    - `/branch-sync` — 다중 브랜치 동기화

- **MCP 서버 오픈소스 공개**:
    - [zep-us/posthog-mcp](https://github.com/zep-us/posthog-mcp)
    - [zep-us/mcp-hub](https://github.com/zep-us/mcp-hub)
    - [lucid-jin/notion-mcp-fast](https://github.com/lucid-jin/notion-mcp-fast)

- **DeepL/Lingo.dev + BullMQ 기반 19개 언어 번역 자동화 파이프라인** —
  3,613개 한영 용어집 + i18n placeholder 보호 시스템으로 번역 품질 보장
  개발자 중심 선(先) 로컬라이징 → 후(後) 검수 프로세스로 빠른 다국어 릴리즈

## 작업 방식

- 같은 카테고리 작업이 두 번 보이면 Skill로 추상화 (반복 → 도구화 패턴)
- Skill 자체에 prompt caching, 5-lens 병렬 같은 **워크플로우 설계** 적용 — 단순 매크로 아님

## 관련 개인 OSS (cross-reference)

사내 Skill과 별개로, 개인 레포에 AI 도구 빌더 작업 분산 — `oss/personal.md` 참고:
- `devflow` (Go TUI) — Linear cycle + Claude Code, ★2
- `mcpick` (TS) — 6개 AI 도구(Claude Code/Desktop, Cursor, Codex, Gemini CLI, Copilot CLI) MCP 설정 sync
- `clawd-on-desk-tauri` (Rust+JS) — Claude Code 세션용 macOS 데스크톱 펫
- `notion-mcp-fast` (Python) — 로컬 SQLite 캐시 직접 읽는 read-only MCP
- `posthog-debugger-extension` (Chrome ext) — PostHog/Zetta 이벤트 실시간 디버깅

이 6개가 **AI 도구 빌더로서 형태별 폭(TUI/데스크톱/MCP/브라우저 확장)**을 보여주는 cross-evidence.

## ⚠️ 약점 (보강 필요)

- **외부 검증 시그널 약함**: 사내 Skill + 개인 OSS 합쳐도 별 누적 적음 (devflow ★2가 최고)
- **포지셔닝 함정**: "AI-Enhanced"가 2026년엔 baseline이라 차별점 약화
- **콘텐츠 0**: digital garden(`lucid-jin.github.io`) 인프라만 있고 글 미발행
- 보강 액션은 `meta/backlog.md` 참고 (블로그 1편, OSS README/데모 보강, CS 메트릭 자동화)

## 📝 갱신 이력

- 2026-04-26: Skill 카운트 정직 카운트(5~6개)로 수정. "30+ 자작" 표현은 사실과 다름(세션 로드 수). MCP 항목 정리.
