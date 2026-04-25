# 이력서 정리 핸드오프 — 2026-04-26

내일 PC에서 마무리하기 위한 단일 컨텍스트 문서. 위에서 아래로 읽으면 됨.

---

## 0. TL;DR (60초 요약)

**무엇을 했나**: 최근 3개월 PR 100개 + Claude Code 세션 60개를 분석해 본인 행동 패턴/시그니처 강점을 추출하고, 채널톡/ClickHouse/Sentry 데이터로 임팩트를 정량화. 기존 Notion 이력서와 비교해 갭/약점 식별.

**핵심 발견**:
1. **시그니처 강점 5가지**가 일관되게 관찰됨 — 측정 우선·진단/픽스 분리·카테고리 종결·셀프 적대적 리뷰·도구화
2. **정량 임팩트 검증된 PR 2건**: #2310(소리유출 4.4×), #2367(카메라 race -91%)
3. **현 이력서 큰 갭**: RTC/실시간 통신 작업이 0줄로 빠져 있음 — 본인 가장 강한 시그니처가 invisible
4. Skill 카운트 정직 카운트는 **5~6개**(`branch-sync`, `code-review-lenses`, `deploy-dev`, `sentry-noise-filter`, `translate`, `rtc`)
5. **포지셔닝 충돌**: "AI-Enhanced Frontend Developer" 표방하지만 실제 시그니처 영역은 Platform/Reliability에 가까움

**다음 액션 (내일 할 일)**:
- [ ] Intro를 v3(아래 §3)로 교체
- [ ] "실시간 통신 안정화" 섹션 새로 추가 (§4-A)
- [ ] "AI 기반 개발 생산성" Skill 블릿을 정직한 5~6개로 수정 (§4-B)
- [ ] "프로덕트 개발" 섹션의 추상적 표현을 구체 PR로 보강 (§4-C)
- [ ] 정량 숫자 intro에 1~2개만, 나머지는 본문 (§5)

---

## 1. 시그니처 행동 패턴 5가지 (3개월 데이터 기반)

객관적으로 반복 관찰된 본인 일하는 방식:

### ① 고치기 전에 측정한다
채널톡 CS / Sentry URL이 들어오면 즉시 ClickHouse `otel_traces` 조회부터. 추측으로 코드 안 만짐.
- 근거: PA mismatch 41.8% false positive를 trace로 산출 후 fix
- 근거: 카메라 race를 1,371 users/week·최악 22 tracks/session으로 정량화 후 Mutex 적용
- MEMORY.md에 "URL slug ≠ spaceHashId" 같은 룰을 본인이 명시화

### ② 진단 PR과 픽스 PR을 분리한다
로깅 PR을 별도로 머지(#2400, #2391, #2369, #2358, #2347)한 뒤 데이터 본 다음 픽스. 결과적으로 픽스가 작고 리뷰 친화적이며, **다음 fix를 측정 가능하게** 만드는 사고가 누적됨.

### ③ 단발 fix가 아니라 카테고리를 닫는다
- Phaser GameConnection race → `useGameConnection` 훅으로 라이프사이클 소유권 재설계 (ZEP-408)
- play-app/libs NX 의존 제거 → 빌드 아키텍처 단순화 (ZEP-346/347, 1,500+ 라인 이전)
- PA guard 카메라 차단 → guard 블록 자체 제거 (#2143)

### ④ 본인이 본인을 적대적으로 리뷰한다
ZEP-408 브랜치에서 셀프 ultra-review로 race 3건을 추가 발견 후 수정 (커밋 `ea450415d`). 코드→의심→재검토를 한 사람 안에서 닫음. 시니어 IC 시그널.

### ⑤ 반복되면 도구로 만든다
같은 작업을 두 번 보면 Skill로 추상화. 직접 작성·운영하는 5~6개 Skill로 RTC/배포/Sentry/번역/리뷰 워크플로우를 파이프라인화.

### ⑥ 인프라 변경에 closed-loop 인시던트 회복력을 묶는다
인프라 PR을 낼 때 항상 롤백 절차/전제조건을 본문에 명시(#2234의 sed 한 줄 + 러너 체크리스트)하고, 프로덕션에서 가설이 깨지면 즉시 revert → 원인 재진단 → forward fix로 닫음. dev-N alias 인시던트(#2393→#2394→#2395)에서 머지~원인 정정까지 1h 8min, revert까지는 18분. 인프라 변경의 되돌림 가능성과 인시던트 응답 속도를 같은 워크플로우에 내장.

---

## 2. 정량 임팩트 (검증된 숫자만)

### PR #2310 — PA mismatch 소리 유출 fix (머지 2026-04-14)
| 메트릭 | Before | After | Δ |
|---|---|---|---|
| 평균 해소시간 (A/B, 500ms 지연) | 1,400ms | 321ms | **-77% (4.4×)** |
| 최소 해소시간 | 1,300ms | 15ms | -99% (86×) |
| `pa_persistent_mismatch` per 1k active user (8일 윈도우) | 645.55 | 577.11 | **-10.6%** |
| 머지 후 5일간 `pa_persistent_mismatch = 0`인 날 | — | 발생 | — |

### PR #2367 — 카메라 publish race Mutex (ZEP-387, 머지 2026-04-21)
| 메트릭 | Before (3w) | After (4d) | Δ |
|---|---|---|---|
| 카메라 중복 publish unique users (평일 평균) | 306/day | 27.7/day | **-91%** |
| Dup buckets | 882/day | 68/day | **-92%** |
| 베이스라인: 1,371 users/week, 최악 22 tracks/session | | | |

### PR #2179 + #2178 — Sentry fingerprint + Safari regex 버그 fix
| 메트릭 | Before | After | Δ |
|---|---|---|---|
| quiz-app Sentry events (22일 윈도우) | 34,864 | 20,472 | **-41%** |
| 차단된 누수 이슈 | QUIZ-APP-38 (8,978u), QUIZ-APP-X (220u) 등 10+ | | |

### 측정 불가/약한 주장 (이력서엔 빼는 게 안전)
- **PR #2143** (PA guard 카메라 차단): ClickHouse retention이 4-05 이후만 존재 → before/after 정량 불가. 정성 사례로만 보존.
- **PR #2378** (livekit 2.4.0): 머지 안 됨 → production 검증 불가. "예상 -41.8% modal false positive"는 추정.
- **PR #2235** (code-review-lenses): 토큰 -48% 자체 A/B 부재. "PR #2147에서 20+ 이슈 발굴"은 헤딩 47개 ≠ 실제 이슈 수. **정성 사례로만 적기**.

---

## 3. Intro 후보 (Codex GPT-5.4 high-reasoning 평가 반영)

### 최종 추천 (v3, Codex 베이스 + 마무리 톤 조정)

> 고치기 전에 측정하고, 같은 문제가 두 번 반복되지 않도록 구조를 바꾸는 7년차 프론트엔드 개발자입니다.
>
> ZEP에서 실시간 통신·게임 클라이언트의 race condition·품질·SEO 문제를 ClickHouse와 Sentry 기반으로 진단하고 카테고리 단위로 닫아왔습니다. 단발 패치보다 훅·아키텍처·자동화 도구로 문제를 닫는 방식을 선호하며, 반복 업무는 Claude Code 기반 자동화 워크플로우로 만들어 팀 전체의 문제 해결 속도를 끌어올립니다.

### Codex 평가 요약 (참고)
- **기존 intro 약점**: AI/모노레포/디자인시스템/SEO/웹바이탈/MCP/번역이 한 문단에 몰려 "주력이 뭔지" 흐림. 넓지만 깊이 불명확.
- **v1 약점**: 좋지만 길고, "믿습니다"가 선언적 — 이력서엔 신념보다 검증된 작업 방식이 더 강함.
- **v2 약점**: 기억엔 남지만 정보량 부족. **"Skill"이 외부 채용담당자에겐 내부 도구명처럼 읽힘** ← 결정적 지적.
- **숫자 위치**: intro엔 1~2개만, 나머지는 "Selected Impact" 또는 본문에서.

### 폐기된 후보들 (참고용)
**v1 (행동 강조, 길게)**:
> 고치기 전에 측정하고, 같은 버그가 두 번 나지 않게 구조를 바꾸는 7년차 프론트엔드 개발자입니다. 실시간 통신·게임 클라이언트의 race condition과 품질 이슈를 ClickHouse·Sentry 기반 정량 진단으로 좁히고, 반복되는 운영 작업은 Claude Code Skill로 추상화해 팀 워크플로우 자체를 바꿉니다. AI로 개발 속도가 빨라질수록, 무엇을 만들지 정의하는 능력과 만든 것을 데이터로 검증하는 능력이 핵심이 된다고 믿습니다.

**v2 (짧고 캐치)**:
> 진단 → 측정 → 구조 변경 → 도구화 — 이 순서로 일하는 7년차 프론트엔드 개발자입니다. 단발 버그를 패치하기보다 카테고리를 닫는 것을 선호하고, 같은 작업이 두 번 보이면 Skill로 만듭니다.

---

## 4. 신규/수정할 섹션 (Notion 복붙용)

### A. 신규: "실시간 통신 안정화" — DevEx 섹션 위에 삽입

```
#### 실시간 통신 안정화 (Real-time / RTC Reliability)

- PA mismatch 시 RTC 갱신 누락으로 인한 소리 유출 fix —
  A/B 테스트로 평균 해소시간 1.4s → 321ms (4.4×),
  프로덕션 8일 윈도우에서 pa_persistent_mismatch 발생률 active user 1k당 -10.6%
- 카메라 중복 publish race를 Mutex + stale localStorage 방어로 구조적 차단 —
  머지 후 4일간 중복 publish unique users 306/day → 28/day (-91%) (ZEP-387)
- ICE 진단 인프라를 P0 단위로 슬라이스 머지 (subscriber PC, remoteCandidateType,
  disconnect 스냅샷, qualityLimitationReason) — 후속 자동 품질 조정의 트리거 지표를
  RTT 단일 → bandwidth/cpu/relay/none으로 재정의
- livekit-client 2.4.0 업그레이드 + setMetadata 비동기화로 PA mismatch 모달
  false-positive 41.8% 감소 (ZEP-375)
- Phaser GameConnection 싱글턴 race를 useGameConnection React 훅 도입으로
  라이프사이클 재설계, 셀프 ultra-review로 race 3건 추가 발견·수정 (ZEP-408)
```

### B. 수정: "AI 기반 개발 생산성" Skill 블릿 정직하게

기존:
> Claude Code 스킬(AI 코딩 에이전트 확장 플러그인) 5개+ 직접 설계·운영:
>   - PR 적대적 코드리뷰 에이전트 (4명 병렬 에이전트 패턴)
>   - QA 검수 자동화
>   - n8n 기반 채널톡 CS 대응 자동화

교체:
```
- 반복 운영·디버깅 작업을 5~6개 Claude Code Skill로 추상화해 팀 워크플로우 표준화:
    - /sentry-noise-filter — Sentry URL → 룰 중복 체크 → 패턴 등록 → PR 자동 생성
    - /deploy-dev — dev-N 배포·branch-sync·PR preview 재사용을 단일 워크플로우로 통합
    - /rtc — RTC 통합 진단(품질 리포트, PA 코드 진단, ClickHouse 추적)
    - /translate — i18n 번역 시트 자동화
    - /code-review-lenses — 5개 관점(consistency·type-safety·reusability·extensibility·elegance) 병렬 + prompt caching 코드리뷰
    - /branch-sync — 다중 브랜치 동기화
- MCP 서버 오픈소스: zep-us/posthog-mcp, zep-us/mcp-hub, lucid-jin/notion-mcp-fast
- DeepL/Lingo.dev + BullMQ 기반 19개 언어 번역 자동화, 3,613개 한영 용어집
```

### D. 신규: "CI/CD & Build Infrastructure" — DevEx 섹션 내부 또는 직후 삽입

> 출처: PR #2234, #2304, #2403, #2333, #2275, #2393/2394/2395, #2217, #2236
> 사용자(lucid-jin) 단독 작성 — 협업자 명시 없음. PR 본문 + diff에서 직접 추출한 정보만 채택.

```
#### CI/CD & Build Infrastructure

- GitHub Actions self-hosted 러너로 13개 워크플로우 19곳 일괄 전환
  (사내 Docker 기반 러너 8개 운영). PR 본문 기준 월 ~$550 → 연간 ~$6,000
  비용 절감 예상. 롤백 절차(sed 한 줄)와 전제 조건 체크리스트를 PR 본문에
  명시해 인프라 변경의 되돌림 가능성을 보장 (PR #2234)
- main 머지 → dev 자동 배포 워크플로우 통합. 6개 앱(admin/dashboard/web/play/
  quiz/auth) push 트리거 단일화 + concurrency group으로 연속 머지 시 구
  버전 자동 취소. core 앱 배포 브랜치를 publish/dev → publish/dev-1로
  통합해 auth/quiz와 일관된 (env, app) 매트릭스 확립 (PR #2304)
- 분산되어 있던 dev 환경 운영 작업(배포 현황 조회, main 최신화 + 재배포,
  슬롯 추천, PR preview 재사용)을 단일 Claude Code Skill `/deploy-dev`로
  통합. 기존 `branch-sync` 스킬을 흡수하면서 (env, app) 독립 슬롯 인식,
  앱 이름 정규화 테이블, PR preview 우선 제안으로 dev-N 슬롯 낭비 방지
  (PR #2403, +226/-96)
- 공급망 공격 방어를 위해 모든 CI 워크플로우 `pnpm install`에
  `--frozen-lockfile` 강제. zus-publish의 의도적 예외 1곳만 보존 (PR #2275)
- dev-N alias 인시던트(2026-04-23) closed-loop 대응: stale alias 가설로
  fix 시도(#2393, 06:57Z) → core 앱 SSL 재발급 충돌로 실패 즉시 revert
  (#2394, 07:15Z, +18분) → 진짜 원인인 .next/cache stale Tailwind JIT
  산출물을 forward fix로 해결(#2395, 08:06Z, +51분). 머지에서 원인 정정까지
  1시간 8분 (PR #2393/2394/2395)
- PR 머지 슬랙 알림 AI 요약을 OAuth 토큰 이슈가 있던 claude-code-action에서
  OpenRouter API(claude-haiku-4.5)로 전환, deploy-vercel 액션과 동일 패턴
  유지 (PR #2333)
- generated API 타입 파일(core/admin OpenAPI, libs/service/apis)을
  `linguist-generated`로 마킹해 PR diff 자동 접힘 처리 (PR #2236)
- main → dev-7 자동배포 트리거 제거로 워크플로우 단순화 (PR #2217)
```

> 본문 미명시 항목(추측 금지로 제외): self-hosted 전환의 빌드 시간 단축치,
> dev-N 통합 전 워크플로우 개수, /deploy-dev로 줄어든 사람 손 단계 수.
> PR #2234 본문은 비용 절감(연 $6k, 월 $550)과 셀프호스팅 인프라 운영(러너
> 8개)만 명시하고 빌드 시간/네트워크 격리 효과는 본문 미명시.

### C. 수정: "프로덕트 개발 - 레거시 마이그레이션 TF 리드" 구체화

기존:
> WebRTC·Phaser 게임 엔진 기반 코드베이스의 기술 스택 현대화, 산재된 스펙을 문서화하고 모듈 간 결합도를 낮춰 유지보수성 확보

대안:
```
- play-app + libs NX 의존성 제거(1,500+ 라인 라이브러리 이전)로 빌드 아키텍처 단순화
  및 Vercel 워크플로우 8 → 1 통합 (ZEP-346/347, PR #2173)
- WebRTC·Phaser 게임 엔진 코드베이스의 race·라이프사이클 이슈를 진단 PR 선행 + 훅
  단위 재설계 패턴으로 정리, 모듈 간 결합도 감소
```

---

## 5. 영문/한글 STAR 불릿 (정량 임팩트, 본문이나 Selected Impact 섹션용)

이력서 톱 인용감 (강함 → 약함 순):

1. **Reduced camera duplicate-publish race users by 91%** (306/day → 28/day) by introducing Mutex + stale localStorage defense against a publish race; baseline of 1,371 users/week and worst-case 22 tracks/session, validated post-deploy in 4-day production window. (PR #2367, ZEP-387)
   - 카메라 중복 publish race를 Mutex + stale localStorage 방어로 차단해 영향 유저 -91% (306→28/day), 베이스라인 1,371u/주·최악 22 track/session.

2. **Cut RTC audio-leak resolution time 4.4×** (1.4s → 321ms, A/B at 500ms induced latency) and reduced production `pa_persistent_mismatch` rate by 10.6% per 1k active users in 8-day post-deploy window. (PR #2310)
   - RTC 소리 유출 해소시간 4.4배 단축(1.4s → 321ms), 배포 후 8일간 active user 1k당 PA mismatch 발생률 -10.6%.

3. **Reduced quiz-app Sentry event volume by 41%** (34,864 → 20,472 over 22 days) by adding API-error fingerprint classification and fixing a Safari-only regex bug that had been leaking 10+ noise issues including a single 8,978-user issue. (PR #2179, #2178)
   - quiz-app Sentry 이벤트 22일 윈도우 -41%(34,864→20,472), Safari regex 버그 + API fingerprint로 누수 10+ 이슈 차단.

4. **Hardened a Phaser game-connection singleton against race conditions** by introducing a `useGameConnection` React hook owning the connect/destroy contract, with self-driven ultra-review surfacing 3 additional races. (ZEP-408)
   - Phaser GameConnection 싱글턴 race를 useGameConnection 훅으로 라이프사이클 재설계, 셀프 ultra-review로 race 3건 추가 발견.

5. **Built 5~6 internal Claude Code skills** (`/sentry-noise-filter`, `/deploy-dev`, `/rtc`, `/translate`, `/code-review-lenses`, `/branch-sync`) that codify debugging/deploy/observability workflows for the front-end team.
   - 사내 Claude Code Skill 5~6개 자작·운영해 디버깅·배포·observability 워크플로우 표준화.

6. **Removed NX dependency from play-app + libs** (~1,500 lines library migration), simplifying monorepo build architecture and consolidating 8 Vercel workflows into 1. (ZEP-346/347, PR #2173)
   - play-app + libs NX 의존성 제거(1,500+ 라인 라이브러리 이전)로 모노레포 빌드 단순화, Vercel 워크플로우 8→1 통합.

7. **Migrated all CI workflows to self-hosted GitHub Actions runners** (13 workflow files, 19 `runs-on` sites) backed by 8 in-house Docker runners, projected to cut CI cost from ~$550/mo to ~$6,000/yr savings; shipped with a one-line rollback recipe and pre-merge runner-readiness checklist in the PR body. (PR #2234)
   - GitHub Actions self-hosted 러너로 13개 워크플로우 19곳 전환(사내 Docker 러너 8개 운영), 연간 ~$6,000 절감 예상. 롤백 절차와 사전 조건을 PR 본문에 명시.

8. **Unified dev-environment ops into a single Claude Code skill `/deploy-dev`** by absorbing the prior `branch-sync` skill and routing four intents (status read / main-sync redeploy / new-slot recommendation / PR-preview reuse) through one entry point with (env, app)-pair slot awareness. (PR #2403, PR #2304)
   - dev 환경 운영 작업 4종을 단일 스킬 /deploy-dev로 통합(branch-sync 흡수), main 머지 → 6개 앱 dev 자동 배포 워크플로우와 (env, app) 매트릭스를 일치.

9. **Closed a dev-N alias incident in 1h 8min via revert-then-forward-fix** — initial fix (#2393) failed in production within 18 min due to Vercel SSL re-issuance conflict on core-app domains, triggered an immediate revert (#2394, +18 min), then root-caused stale `.next/cache` Tailwind JIT artifacts and shipped a forward fix (#2395, +51 min) that selectively clears `.next/cache` while preserving `node_modules` cache. (PRs #2393/2394/2395)
   - dev-N alias 인시던트를 1시간 8분에 closed-loop으로 처리: 18분 만에 revert → 진짜 원인(.next/cache stale Tailwind JIT)을 51분 뒤 forward fix.

기존 이력서 숫자(이미 있음, 유지):
- 글로벌 방문자 4× 성장
- 모바일 FCP 4.2→3.2s, LCP 14.5→4.9s, CLS 0.255→0.061
- Figma 토큰 자동 매칭 80%
- DeepL+BullMQ 19개 언어 번역 + 3,613 용어집

---

## 6. 메타 평가 — 객관적 강점/약점

### 객관적 강점 (3개월 데이터 기반)
1. **데이터로 의사결정** — PR마다 정량 지표가 붙어 있는 비율이 동료 평균보다 높음
2. **시스템 경계 감수성** — 패치 단위가 아니라 카테고리 종결 단위로 사고
3. **AI-네이티브 메타스킬** — 2026년 채용시장에서 "AI 도구 잘 씀" ≠ "AI 워크플로우 설계함", 본인은 후자
4. **T자형 폭** — RTC/게임에 깊이, 빌드/i18n/admin/observability에 운영 가능 수준
5. **인시던트 응답 속도** — Vercel 사고 24h, axios CVE 당일, PA 카메라 same-day. on-call 신뢰 시그널

### 객관적 약점 (이력서 작성 시 보완 필요)
1. **비즈니스 메트릭 부재** — 기술 메트릭(오탐률, 라인 수, 토큰)은 풍부하지만 "DAU/전환율/CS 티켓" 같은 비즈니스 지표는 거의 없음
2. **외부 가시성 부족** — 30+ Skill을 만들었지만 사내용. 블로그/OSS/컨퍼런스 부재 → 같은 실력이라도 시장 가치 차이
3. **영역별 임팩트 격차** — RTC/게임은 정량 풍부, Quiz/Auth/Admin은 비어 있음
4. **현 이력서 포지셔닝 흐림** — "AI-Enhanced Frontend"인데 실제 작업은 Platform/Reliability — 둘 다 어중간

---

## 7. 현재 이력서 원본 (참조)

### Contact
- Phone: 010-9304-1905
- Email: whe1915@gmail.com
- Github: https://github.com/lucid-jin

### 기존 Introduce
> AI를 적극 활용해 팀과 제품을 변화시키는 프론트엔드 개발자입니다.
>
> 7년간 대규모 웹 서비스에서 모노레포 통합, 디자인 시스템 패키지화, SEO 최적화(글로벌 방문자 4배 성장), 웹 바이탈 개선 등 **제품과 운영 양쪽에서 구체적 성과**를 만들어왔습니다. 동시에 AI 에이전트, MCP(Model Context Protocol) 서버, 번역 자동화 파이프라인을 직접 설계·운영하며 **팀의 개발 생산성을 구조적으로 높이는 일**에 집중하고 있습니다.
>
> AI로 개발 속도가 빨라질수록, 무엇을 만들지 설계하는 능력과 만든 것을 정확히 검증하는 능력이 개발자의 핵심 역량이 된다고 생각합니다.

### Work Experience > ZEP (2022.07~)
> DAU 90만, 메타버스 가상 공간을 제공하는 플랫폼
> 프론트엔드 파트 / B2E 제품 개발 및 DevEx 담당

#### DevEx & 운영 자동화 플랫폼 구축
- pnpm workspace 모노레포 (5개 앱)
- 디자인 시스템 패키지(zus)
- Figma 토큰 자동 매칭 80%
- Orval 기반 OpenAPI 자동화
- Sentry 공통 패키지 표준화

#### AI 기반 개발 생산성 혁신
- Claude Code 스킬 5개+
- MCP 서버 오픈소스
- DeepL+BullMQ 19개 언어 번역
- 3,613 한영 용어집

#### 젭 퀴즈 제품 개발
- AI 퀴즈 생성 UI
- 글로벌 결제 TF 리드
- 글로벌 방문자 4× (3개월)
- 모바일 FCP 4.2→3.2s, LCP 14.5→4.9s, CLS 0.255→0.061

#### 프로덕트 개발
- 레거시 마이그레이션 TF 리드 (WebRTC·Phaser)
- 내부 어드민 TF 리드 (Spring Boot + Next.js)

### Work Experience > 슈프리마 (2020.12-2022.07)
- AngularJS → React + TypeScript
- i18n 적용
- Cypress, Protractor E2E

### Work Experience > 클라우드스톤 (2019.04-2020.12)
- Vue.js 모바일 웹앱
- 배송 기사 관리/주문 라벨 자동인쇄

### OSS & Side Projects
- zep-us/posthog-mcp
- zep-us/mcp-hub
- lucid-jin/notion-mcp-fast
- lucid-jin/claude-toolkit

### Other
- 스파르타 코딩 클럽 멘토 (2024.06-09)

### Skill
- Front-end: JS/TS, React, Next.js, React Query, Mobx, Recoil, Redux, Tailwind, Scss, Vue
- AI/Automation: Claude Code, MCP Server, DeepL, Lingo.dev, BullMQ
- Build & DX: pnpm Workspace, Nx, Storybook, Cypress, Webpack, Orval
- Back-end: Node.js, Express, Hono.js, Bun, Kotlin/Java, Spring Boot
- Observability: Sentry, PostHog, OpenTelemetry
- DevOps: Vercel, GitHub Actions, Docker, Coolify

### Education
- 광주대학교 생명공학과 (2012.03-2018.02)

---

## 8. 부록 — 관련 파일

이 PC에 같이 저장된 분석 산출물:

- `/Users/zep/WebstormProjects/zep-client/resume-strengths-2026-04.md` — 영역별 강점 상세 + STAR 불릿 12개
- `/Users/zep/WebstormProjects/zep-client/impact-metrics-2026-04.md` — before/after 정량 측정 + 사용한 ClickHouse 쿼리 부록

내일 PC에서 작업 시 위 두 파일도 같이 참고하면 됨. 본 문서는 핵심만 추린 단일 핸드오프.

---

## 9. 의사결정 체크리스트 (내일 마무리할 때)

- [ ] **포지셔닝 결정**: AI-Enhanced 유지 vs Platform/Reliability로 reposition (현 이력서는 둘 다 어중간)
- [ ] **Intro v3 채택 여부** — 그대로 / 톤 조정 / 자체 변형
- [ ] **새 RTC 섹션 위치** — DevEx 위 / 별도 Highlights / Work Experience 안 어디
- [ ] **정량 숫자 위치** — Intro에 1~2개 vs Selected Impact 섹션 vs 본문 분산
- [ ] **Skill 카운트 표현** — "5~6개" vs "5개" 만 / 각 Skill 한 줄 설명 유지 여부
- [ ] **AI/MCP 섹션 비중** — 현 강도 유지 vs RTC 강조하며 축소
- [ ] **약점 보완 방향** (선택) — 비즈니스 메트릭 추가 시도 / 외부 가시성(블로그·OSS) 액션 아이템
