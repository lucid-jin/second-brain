---
section: oss-personal
last_updated: 2026-04-26
---

# Open Source — 개인 레포

## AI / 개발 도구 (자작)

| 레포 | 스택 | ★ | 한 줄 설명 |
|---|---|---|---|
| [**lucid-jin/devflow**](https://github.com/lucid-jin/devflow) | Go | **2** | TUI for Linear cycle-driven development with Claude Code |
| [**lucid-jin/mcpick**](https://github.com/lucid-jin/mcpick) | TypeScript | 0 | Pick and sync MCP server configs across AI tools (Claude Code/Desktop, Cursor, Codex, Gemini CLI, Copilot CLI) |
| [**lucid-jin/clawd-on-desk-tauri**](https://github.com/lucid-jin/clawd-on-desk-tauri) | Rust + JS | 0 | Tauri 포트 — Claude Code/Codex/Cursor 세션용 macOS 데스크톱 펫 |
| [**lucid-jin/notion-mcp-fast**](https://github.com/lucid-jin/notion-mcp-fast) | Python | 0 | Read-only MCP server for Notion — 로컬 SQLite 캐시 직접 읽음 |
| [**lucid-jin/posthog-debugger-extension**](https://github.com/lucid-jin/posthog-debugger-extension) | JavaScript | 0 | PostHog/Zetta 분석 이벤트 실시간 디버깅 Chrome 확장 |
| [**lucid-jin/claude-toolkit**](https://github.com/lucid-jin/claude-toolkit) | Python | 0 | Claude Code 스킬 모음 (개인 연습용) |

## 콘텐츠 / 가시성 인프라

- [**lucid-jin/lucid-jin.github.io**](https://github.com/lucid-jin/lucid-jin.github.io) — Digital Garden (Quartz + Obsidian)
  - ⚠️ **인프라만 구축, 콘텐츠 미발행 상태**. 보강 액션의 1차 도착지 (`meta/backlog.md` §2)

## 시그널 분석

**강점**:
- AI 도구 빌더로서 **폭이 넓다**: TUI(devflow) / 데스크톱 앱(clawd) / MCP 서버(notion-mcp-fast) / MCP 메타툴(mcpick) / 브라우저 확장(posthog-debugger) — 각 형태에서 1개씩 출하
- `devflow` ★2 — 작지만 **외부 검증 첫 시그널** (스타 0이 아님)
- `mcpick`은 멀티-AI 도구 시대의 실수요(MCP 설정이 6개 도구마다 따로 있음) 정조준

**약점**:
- 스타 누적 적음 → 외부 채택률 저조
- README/데모 GIF/사용 예제 부족 (devflow 외)
- Digital Garden은 인프라만 있고 콘텐츠 0 — 가시성 시그널로 미작동

## 🎯 보강 액션 (`meta/backlog.md` 참고)

1. `mcpick` 또는 `devflow` README + 데모 GIF + 한국 커뮤니티 공유 → 별 누적 시작
2. ZEP RTC 디버깅 사례를 `lucid-jin.github.io`에 첫 발행
3. 사내 `/sentry-noise-filter` 또는 `/code-review-lenses`를 OSS로 추출

## 📝 갱신 이력

- 2026-04-26: AI 도구 6개 통합 (devflow ★2, mcpick, clawd-on-desk-tauri, posthog-debugger-extension 추가). Digital Garden 인프라 항목 분리. mac-migration은 제외.
