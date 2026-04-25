# 이력서 강점 분석 — 2026-01-25 ~ 2026-04-25 (최근 3개월)

분석 대상: 머지된 PR 100개(`gh pr list --author=@me --state=merged --search "merged:>=2026-01-25"`) + 약 60개 Claude Code 세션 + commit history.

---

## 한눈에 보는 강점 요약

- **3개월간 100개 PR 머지**, RTC/게임/Quiz/Auth/모노레포 빌드 시스템/Observability/DX 7개 도메인을 동시에 추진. 수치 임팩트가 명시된 PR이 다수(예: PA mismatch 모달 41.8% false positive → 한 자릿수, 카메라 중복 publish 1,371명/주 영향 → 0).
- **시스템 단위 리팩토링 주도**: dashboard-app·web-app의 NX → 독립 pnpm 빌드 전환(ZEP-305/306, +789/-1094 라인), Lambda → Next.js API/서버 마이그레이션, Logstash 로깅 인프라 제거 등 모노레포 빌드/인프라 레이어를 직접 재설계.
- **데이터 드리븐 RTC 디버깅**: ClickHouse `otel_traces`로 가설 검증 후 코드를 손대는 워크플로우. publisher/subscriber ICE 비대칭, qualityLimitationReason(bandwidth/cpu/none) 분리 등 "어떤 진단 필드가 필요한가"를 직접 설계해 후속 fix(ZEP-386/388)의 트리거 지표를 정의.
- **closed-loop 디버깅 사이클**: 진단 로그 PR → ClickHouse 분석 → 정량화 → 픽스 → 회귀 방어를 PR 단위로 일관되게 반복 (예: PA guard #2137 진단 → #2143 픽스, RTC race #2367/#2400/#2391).
- **도구 빌더 마인드셋**: 30+개 사내 Claude Code Skill을 자작·유지보수. `/code-review-lenses`(2,177 LOC, 5-lens 병렬 리뷰 + 2-Layer prompt caching으로 토큰 ~48% 절감)는 기존 리뷰가 놓치던 구조적 이슈 20+건을 추가로 탐지.
- **CI 자동화 / Performance Guard 구축**: PR 자동 리뷰 워크플로우(#2075, +6,484 LOC)와 vercel-react-best-practices 룰 도입으로 성능 안티패턴 사전 차단.

---

## 영역별 강점

### 1. RTC / LiveKit 실시간 통신 — 진단 인프라부터 직접 설계

- **publish race condition 제거(ZEP-387, PR #2367)**: 채널톡 CS "마이크 켜면 카메라도 켜진다"를 ClickHouse 분석으로 **최근 7일 1,371명 영향, 1,742개 세션, 최악 케이스 10초 내 track 22개**로 정량화 → `publishCamera/Microphone` 코어에 Mutex 추가 + stale localStorage 방어. SAME_TAB race + auto-activate 두 경로 동시 차단.
- **PA mismatch 41.8% false positive 제거(ZEP-375, PR #2378)**: livekit-client 2.2.0 → 2.4.0 업그레이드, `await setMetadata()` 적용, 모달 타이밍 4s → 6s 재설계. setMetadata가 void 반환이라 서버 확인이 불가능했던 구조적 한계를 SDK 레벨에서 해결.
- **ICE 진단 인프라 설계(ZEP-365, PR #2358/#2296/#2357)**: publisher/subscriber 양쪽 PC ICE info, remoteCandidateType, currentRttMs, disconnect 직전 스냅샷을 단일 이벤트로 수집해 "TURN relay vs UDP 차단 vs LiveKit 클러스터 경로"를 한 줄 SQL로 구분 가능하게 만듦. 세션당 추가 이벤트 +2로 노이즈 최소화.
- **병목 원인 분리 진단(ZEP-389, PR #2369)**: Codex adversarial review 지적("ZEP-388은 RTT 문제를 bandwidth 문제처럼 다룸")을 받아들여 `qualityLimitationReason`(bandwidth/cpu/other/none), nack/pli/freeze delta, fpsMin을 60s 윈도우로 집계. 후속 자동 품질 조정의 트리거를 RTT가 아닌 실제 비디오 병목 지표로 재설계.
- **StatsReporter 데드코드 활성화(PR #2285)**: 이미 구현되어 있으나 미사용이던 `useRtcStatsReporter`를 production-safe하게 정비(`isCollectingRef` overlap guard, `usePreservedCallback`, warm-up 첫 샘플 버림, disconnect 시 buffer flush) 후 PlayRtc에 연결.
- **PA guard 카메라 미표시 closed loop(#2137 → #2143)**: 코딧 부트캠프 CS에서 "최대 3.6시간 영구 차단"을 ClickHouse로 확인 → 4개 진단 로그(PA_TILE_DETECTED, PA_PERSISTENT_MISMATCH, HANDLE_MOVED_CHUNK_*) 배포 → localPA 의존 제거로 근본 fix.

**STAR 불릿**
- 카메라 중복 publish race를 ClickHouse trace 분석으로 **1,371명/주 영향, 최악 22개 track 동시 publish**로 정량화하고 publish 코어 함수의 Mutex 도입과 SettingStore localStorage 부활 경로 차단으로 동일 카테고리 인시던트 0건.
- ICE 진단 필드와 qualityLimitationReason 기반 윈도우 집계를 직접 설계해, 단순 "RTT 높음"이었던 RTC 품질 알람을 bandwidth/cpu/none/relay 차원으로 분리해 후속 자동 품질 조정 기능의 트리거 지표를 정의.

### 2. 모노레포 빌드 시스템 리아키텍처 — NX 의존 제거

- **dashboard-app NX 제거(ZEP-305, PR #1994, +577/-966)**: `withNx`/`composePlugins`/`project.json` 제거 후 `pnpm --filter dashboard-app build` 독립 구조로 전환. `libs/common/dashboard/*`를 앱 내부로 이동하고 외부 소비자(admin-app, layouts)에 인라인 처리해 라이브러리 경계 재정의.
- **web-app NX 제거(ZEP-306, PR #2151)**: dashboard 패턴을 web-app에 확산. `eslint.config.mjs` flat config 전환, `tailwind.config.js` 경로 수동화, jest preset 교체까지 일괄 처리.
- **Lambda → Next.js API route(PR #2127)**: Vimeo 썸네일 fetch를 외부 Lambda에서 play-app 내 API route로 이전, Sentry 에러 태깅까지 같이 추가.
- **Logstash/es.zep.us 인프라 제거(PR #2043, +6/-464)**: 사용 안 하는 `libs/common/logstash/` 라이브러리 자체 삭제, core-logger·auth-app·quiz-app에서 LogstashLogger 제거. Zetta/HyperDX 단일 경로로 통합.
- **Vercel 워크플로우 통합(PR #2078, +528/-1125)**: 8개 deploy 워크플로우 + 4개 composite action을 `deploy-vercel.yml` 1개로 통합, quiz-app maintenance mode 추가, CLI preview 시 vercel alias 자동 연결.

**STAR 불릿**
- dashboard-app/web-app의 NX 빌드 시스템을 제거하고 `pnpm --filter` 독립 빌드 구조로 전환해(ZEP-305/306) 빌드 캐시 의존을 끊고 앱별 라이브러리 경계를 재정의. 1,500+ 라인의 라이브러리 이전과 인라인 처리를 회귀 0으로 완료.
- 8개 Vercel 배포 워크플로우와 4개 composite action을 단일 워크플로우로 통합하고 quiz-app maintenance mode를 도입해, dev/stage/LIVE 배포의 운영 표면적을 1/3 이하로 축소.

### 3. 게임 엔진(Phaser 3) Race Condition 안정화

- **GameConnection 싱글턴 race 차단(hoinjin/zep-408 브랜치, 5 commits)**: connect/destroy/reconnect 사이의 listener 누수 및 emit 순서 어긋남을 `useGameConnection` React 훅 도입으로 라이프사이클 소유권을 뷰 레이어에 명시 위임. `connect()` 실패 경로 안정화 + destroy 계약 위반 감지 추가. **셀프 ultra-review로 race 3건 추가 식별** 후 동시 수정.
- **PlayRtc mount/unmount 카운터 진단(#2400/#2391)**: cleanup이 의도치 않게 발동되는 경로를 카운터 로그로 가시화해 race를 행동→결과 단위로 추적 가능하게 만듦.

**STAR 불릿**
- Phaser GameConnection 싱글턴 race를 `useGameConnection` 훅 + destroy 계약 검사로 구조적 차단, 셀프 adversarial review로 추가 race 3건 발견 후 일괄 수정해 동일 카테고리 Sentry 이슈 재발 0.

### 4. Observability — Sentry 노이즈 자동화 + GPU/플랫폼 메타데이터

- **`/sentry-noise-filter` Skill 작성·운영**: Sentry URL 입력 → `@zep/sentry` 룰 중복 체크 → 필터 바이패스 진단 → 패턴 추가 → PR 생성을 단일 워크플로우로 자동화. 결과 리포트는 일자별 누적.
- **API 에러 fingerprint 시스템(PR #2179)**: `httpStatus + status` 기반 그룹핑 + 401/403/404/422/429 노이즈 drop을 `@zep/sentry`에 추가. 이전에 네트워크 에러 노이즈 때문에 통째로 끄면서 생긴 모니터링 사각지대(500 에러가 다른 에러와 섞임) 해소.
- **Regex/DOMException 버그 픽스(PR #2178)**: `EXTERNAL_INJECTED_VARIABLES` 정규식이 Safari만 매칭해 QUIZ-APP-38(8,978u)/65(220u) 등 10+ 이슈가 누수되던 버그 발견·수정. DOMException 매칭을 위해 `getErrorMessage()`가 `name`을 `message`에 결합하도록 개선.
- **axios 컨텍스트 + HTTP status 필터(PR #2384, ZEP-394)**: axios 에러에서 status가 빠져 있어 노이즈 분류가 불가능했던 구조적 한계를 자체 글로벌 인터셉터로 해소.
- **GPU 렌더러/플랫폼 정보 글로벌 로그(PR #2169)**: WebGL `UNMASKED_RENDERER`, `webgl_available`, `userAgentData` 기반 `cpu_architecture`/`platform_version`을 모든 ClickHouse 로그에 첨부. Chrome UA Reduction으로 OS 버전 손실된 환경에서도 인텔맥/M시리즈/GPU 가속 OFF 유저를 SQL 1줄로 식별 가능.
- **Sentry instrumentation.ts 마이그레이션(PR #2195)**: 4개 앱(play/web/auth/quiz)을 deprecated `sentry.server.config.ts` → Next.js 표준 `instrumentation.ts` 패턴으로 일괄 이전.

**STAR 불릿**
- API 에러 fingerprint + 노이즈 status drop을 `@zep/sentry` 패키지 레벨에 통합해, "노이즈가 심해서 통째로 껐다가 모니터링 사각지대 발생" 패턴을 종결.
- WebGL/userAgentData 기반 디바이스 메타데이터를 글로벌 로거에 주입해 GPU·CPU·플랫폼 단위 CS 분류를 SQL 1쿼리로 수행 가능하게 만듦.

### 5. CI 자동 리뷰 / Code Review 자동화 (DX)

- **`/code-review-lenses` Skill(PR #2235, +2,177 LOC)**: PR을 5개 독립 렌즈(Pattern Consistency / Type Safety / Reusability / Extensibility / Elegance)로 병렬 분석. **2-Layer prompt caching**(Project Context 주 1회 + PR Context PR당 1회) 구조로 토큰 ~48% 절감. PR #2147(DM 기능)에 적용해 기존 claude-review가 놓친 구조 이슈 20+건(God Context 18 props, 80줄 단일 함수, 1:1 DM 가정 하드코딩 등) 추가 발굴. Agent-Agnostic 구조로 Claude/Codex/CI 호환.
- **CI Performance Guard(PR #2075, +6,484 LOC)**: `claude-review.yml` 워크플로우로 PR 열림/업데이트 시 vercel-react-best-practices 기반 성능 안티패턴 자동 차단. 40+ 룰 / 8 카테고리. `@claude` 멘션 대응 워크플로우 별도 분리.
- **agentation 도입(PR #2245)**: dev 환경 visual feedback 도구를 react-grab(서버 의존, Claude Code 전용)에서 agentation(zero-dep, 컴포넌트 only, AI agent 범용)으로 6개 앱 전체 교체. 비개발자(QA/디자이너) 사용까지 커버.
- **quiz-app ESLint 강화(PR #2282, +832/-217)**: `no-floating-promises`, `no-misused-promises`, `max-lines`/`max-lines-per-function`/`max-depth`/`max-params`/`complexity`, type-aware lint, t() 하드코딩 폴백 금지 등 코드 품질 룰을 prod 코드베이스에 일괄 적용.

**STAR 불릿**
- 5-lens 병렬 분석 + 2-Layer prompt caching 구조의 PR 리뷰 Skill을 자작해 토큰 비용 ~48% 절감하면서 기존 리뷰가 놓치던 구조적 이슈 20+건을 단일 PR에서 추가 발굴.
- CI 자동 PR 리뷰 워크플로우를 vercel-react-best-practices 룰셋 기반으로 구축해 성능 안티패턴을 머지 전 단계에서 차단.

### 6. 도구 자동화 · 운영 워크플로우

- **자체 제작/유지하는 Skill 30+개**: `/sentry-noise-filter`, `/deploy-dev`(branch-sync 흡수, PR #2403), `/clickhouse`, `/rtc`, `/debug-game`, `/channeltalk-debug`, `/code-review-lenses`, `/translate`(i18n 시트 자동화, PR #2198), `/onboarding`, `/builder-onboarding`, `/zep-japan-blog` 등.
- **main 머지 → dev 자동 배포 + core 앱 publish/dev-1 통합(PR #2304)**: `push: branches: [main]` 트리거 추가, `concurrency: cancel-in-progress`로 연속 머지 시 최신 main 재배포, 6개 앱 분기 통일.
- **CODEOWNERS 자동 assign(PR #2044)**: 코드리뷰 자동 assign을 CODEOWNERS 단일 진실 소스로 전환해 PR 별 수동 멘션 비용 제거.
- **번역 인프라 Lambda → fe-devx(PR #2153)**: quiz-app `translate-auto`/`locale:translation`을 외부 Lambda에서 사내 fe-devx API로 이전, 동기/비동기 두 경로 분리.

**STAR 불릿**
- dev-N 배포·branch-sync·PR preview 재사용을 `/deploy-dev` 단일 Skill로 통합(PR #2403)하고 main 머지 → dev 자동 배포 트리거를 추가(PR #2304)해, 6개 앱 dev 배포의 수동 단계를 제거.

### 7. 보안 / 인시던트 대응 (간략 — 운영 영역)

> 임팩트는 작지만 closed-loop 사례로 1줄씩.

- Vercel 4월 보안 인시던트 24시간 내 대응(미사용 시크릿 정리 #2355, `VERCEL_TOKEN` 로테이션 #2353).
- axios CVE-2026-40175(CVSS 10.0) 공개 당일 1.15.0 업그레이드(#2324) + 인접 백엔드 영향도 검토.
- dev-N alias 버그 hotfix 3개(#2393/#2394/#2395)로 인시던트 동안 배포 파이프라인 가용성 유지.

---

## 이력서에 바로 쓸 수 있는 영문 + 한글 불릿 (12개)

1. **Eliminated a privacy-critical RTC race affecting 1,371 users/week** where mic activation duplicated camera tracks (worst case: 22 tracks in 10s) — quantified via ClickHouse `otel_traces`, fixed with publish-side Mutex and SettingStore localStorage stale-state defense (PR #2367, ZEP-387).
   - 마이크 켜면 카메라가 중복 publish되는 race를 ClickHouse로 1,371명/주·최악 22 track으로 정량화 후 publish Mutex + localStorage 방어로 차단 (PR #2367).

2. **Reduced PA-mismatch modal false positives from 41.8% to single digits** by upgrading livekit-client 2.2.0 → 2.4.0, awaiting `setMetadata()`, and recalibrating retry/buffer timing 4s → 6s (PR #2378, ZEP-375).
   - LiveKit 2.4.0 + setMetadata 비동기화 + 모달 타이밍 재설계로 PA mismatch 오탐률 41.8%→한 자릿수 (PR #2378).

3. **Designed RTC diagnostic infrastructure** that distinguishes RTT vs bandwidth vs CPU vs UDP-blocked vs TURN-relay bottlenecks via a single ClickHouse query — publisher/subscriber dual ICE collection, `qualityLimitationReason` window aggregation, disconnect snapshots (PRs #2358/#2296/#2369).
   - publisher/subscriber 이중 ICE 수집 + qualityLimitationReason 윈도우 집계 + disconnect 스냅샷으로 RTC 병목 원인을 SQL 1쿼리로 분리 (#2358/#2296/#2369).

4. **Re-architected the dashboard-app and web-app build systems** by removing NX dependency and migrating to standalone `pnpm --filter` builds (ZEP-305/306). Relocated 1,500+ lines of internal libraries and inlined external consumers without regression.
   - dashboard/web-app의 NX 의존을 제거하고 pnpm 독립 빌드 구조로 전환, 1,500+ 라인 라이브러리 이전을 회귀 0으로 완료 (ZEP-305/306).

5. **Built `/code-review-lenses`** — a 5-lens parallel PR review skill with 2-Layer prompt caching (project + PR context) achieving ~48% token reduction, surfacing 20+ structural issues missed by single-pass review on a real DM feature PR (#2235, +2,177 LOC).
   - 5-lens 병렬 + 2-Layer prompt caching으로 토큰 ~48% 절감, 기존 리뷰가 놓친 구조 이슈 20+건 발굴하는 사내 리뷰 Skill 자작 (#2235).

6. **Productionized a CI Performance Guard** by integrating Claude Code review into GitHub Actions with the vercel-react-best-practices ruleset (40+ rules, 8 categories), automatically blocking React/Next.js performance anti-patterns at PR time (#2075).
   - CI에 Claude 자동 리뷰 + vercel-react-best-practices 40+ 룰을 결합해 성능 안티패턴을 머지 전 차단 (#2075).

7. **Closed an RTC observability gap** by activating dead-code `useRtcStatsReporter` with production-safe guards (overlap prevention, warm-up sample drop, disconnect buffer flush) and adding direction-aware quality logging (#2285).
   - 데드코드였던 useRtcStatsReporter를 production-safe하게 정비·활성화하고 connection_quality_changed에 direction 필드 추가 (#2285).

8. **Resolved a PA-guard "permanent camera blackout"** affecting bootcamp users (up to 3.6h block) via a closed-loop debug cycle: 4 diagnostic events → ClickHouse pattern analysis → root-cause fix (removed localPA dependency from PA guard) (#2137 → #2143).
   - 부트캠프 카메라 영구 차단(최대 3.6h)을 진단 로그 → ClickHouse 분석 → 근본 원인(localPA 의존) 제거의 closed loop으로 종결 (#2137→#2143).

9. **Fixed a Sentry filter regex bug** that leaked 10+ noisy issues (QUIZ-APP-38: 8,978 users, QUIZ-APP-65: 220 users) by extending Safari-only `EXTERNAL_INJECTED_VARIABLES` patterns to Chrome/Firefox and combining DOMException name into message-matching path (#2178).
   - Safari-only로 작성된 노이즈 정규식이 Chrome/Firefox 에러 10+종(8,978u + 220u 포함)을 누수시키던 버그 발견·수정 (#2178).

10. **Authored 30+ internal Claude Code skills** (`/code-review-lenses`, `/clickhouse`, `/rtc`, `/deploy-dev`, `/sentry-noise-filter`, `/translate`, `/debug-game` …) codifying the team's debug, deploy, review, and i18n workflows.
    - RTC/리뷰/배포/번역/디버깅용 사내 Skill 30+개 자작·유지보수해 팀 워크플로우 표준화.

11. **Hardened a Phaser 3 GameConnection singleton against race conditions** by introducing a `useGameConnection` React hook that owns the connect/destroy contract — self-driven adversarial review surfaced 3 additional races during the same branch (ZEP-408, 5 commits).
    - Phaser GameConnection 싱글턴 race를 useGameConnection 훅으로 라이프사이클 명시화, 셀프 ultra-review로 race 3건 추가 발견·수정.

12. **Unified Vercel deployment pipeline** by collapsing 8 workflows + 4 composite actions into a single `deploy-vercel.yml`, adding maintenance mode for quiz-app, and triggering dev auto-deploy on main merge (#2078, #2304).
    - Vercel 배포 워크플로우 8개 + composite action 4개를 단일 워크플로우로 통합, quiz-app maintenance mode 도입, main 머지 시 dev 자동 배포 트리거 추가 (#2078, #2304).

---

## 자주 보이는 메타-스킬

- **가설 → 데이터 검증 → 코드 변경 순서 강제**: 슬랙/채널톡/Sentry URL을 받으면 즉시 ClickHouse 조회부터. 추측으로 코드 만지지 않음. 본인 MEMORY.md에도 "URL slug ≠ spaceHashId" 같은 룰을 명시.
- **진단 우선, 픽스는 그 다음**: livekit, PA mismatch, RTC mount/unmount 모두 픽스 전에 로깅 PR을 별도로 머지(예: #2400, #2391, #2369, #2137). 결과적으로 픽스 PR이 작고 리뷰 친화적.
- **Adversarial review를 코드에 짜넣기**: ZEP-408 브랜치에서 본인이 ultra-review를 돌려 race 3건을 추가로 잡아낸 커밋(`ea450415d`). ZEP-389 PR은 Codex adversarial review의 지적을 받아들여 트리거 지표를 RTT → qualityLimitationReason으로 재설계.
- **도구 자동화 사고**: 같은 분류의 작업이 두 번 보이면 곧장 Skill로 추상화. 결과적으로 일을 "코드"가 아니라 "워크플로우" 단위로 출하. 30+개 Skill 중 다수가 다른 PR/세션에서 재사용됨.
- **시스템 단위 리팩토링 두려워하지 않음**: NX 빌드 시스템 제거, Logstash 인프라 삭제, Lambda → Next.js API 이전 같은 큰 수술을 회귀 0으로 처리하면서 모노레포 라이브러리 경계를 의식적으로 재정의.
- **문서화 위계 인식**: Linear 이슈 vs 코드 주석 vs ADR vs MEMORY를 룰로 구분(MEMORY: "리니어 이슈보다 코드 주석 선호"). 의사결정 비용을 본인 차원에서 미리 줄여둠.
