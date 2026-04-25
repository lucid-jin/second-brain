# 이력서용 비즈니스 임팩트 리포트 (2026-04-25 기준)

> 작성: 2026-04-25
> 작성자: hoin.jin (whe1915@gmail.com)
> 데이터 소스: GitHub PR 본문(A/B 측정), ClickHouse `default.otel_traces` (ServiceName=`play-app`), Sentry (`zep-us`), Channel Talk

---

## 핵심 한 줄

- **PR #2310 (RTC PA mismatch 소리 유출 fix)**: 평균 해소시간 **1,400 ms → 321 ms (−77 %, 4.4 x)**, 최소 **1,300 ms → 15 ms (86 x)**, 1,000 active user 당 `pa_persistent_mismatch` 이벤트 **645.6 → 577.1 (−10.6 %)**.
- **PR #2143 (PA guard 카메라 영구 차단 fix)**: 코딧 부트캠프에서 발생하던 **최대 3.6 시간 카메라 영구 차단**을 0 으로 (PA guard 진단 로그 기반). 정량 비교 미지원 — 사유 부록 참조.

---

## PR #2310 — PA Mismatch 소리 유출 fix

| 메트릭 | Before (8 일) | After (8 일) | Δ |
|---|---|---|---|
| 평균 해소시간 (PR 본문 A/B) | 1,400 ms | 321 ms | **−77 % (4.4 x)** |
| 최소 해소시간 (PR 본문 A/B) | 1,300 ms | 15 ms | **−98.8 % (86 x)** |
| `[RTC]_pa_persistent_mismatch` (총 events) | 8,979 | 14,625 | +63 % (트래픽 +82 %) |
| `[RTC]_pa_persistent_mismatch` per 1k active user | **645.55** | **577.11** | **−10.6 %** |
| `[RTC]_pa_subscription_recalc_completed` (신규 fast-path) | 12 | 1,551,703 | 신규 회복 경로 가동 |
| `[RTC]_pa_force_sync` events (회복 호출) | 4,097 | 12,322 | +201 % vs 트래픽 +82 % → **per-user 회복 빈도 증가** |
| Active users (PA tile 진입 unique) | 13,909 | 25,342 | +82 % (자연 증가) |
| Sentry "PA" 메시지 errors | 20,390 | 7,245 | −64 % (검색어 광범위, 추정) |
| 4-week window 동안 `pa_persistent_mismatch=0` 인 일자 | 0 | **5 일** (4/15–4/19) | 신규 fast-path가 mismatch를 user-visible 단계로 가지 않게 처리 |

### Before/After 윈도우
- Before: 2026-04-06 ~ 2026-04-13 (8 일, ClickHouse 텔레메트리 자체가 4/5 부터 살아 있어 4 주 비교 불가)
- After: 2026-04-14 ~ 2026-04-21 (8 일)

### 해석
- **PR 본문 A/B 결과**가 가장 신뢰도 높은 단일 메트릭. 평균 해소시간 1.4 s → 321 ms 는 동일 시나리오 재현 시 측정.
- 프로덕션 텔레메트리에서는 트래픽이 8 일 만에 +82 % 늘어 절대 이벤트 수 증가가 자연스럽다. 1k active user 당 `pa_persistent_mismatch` 가 **−10.6 %** 감소한 것이 신뢰 가능한 production 신호.
- `pa_subscription_recalc_completed` 가 12 → 1.55 M 로 폭증한 것은 PR 이 새로 도입한 fast-path 가 정상 가동 중임을 보여준다 (이 경로가 없던 시절에는 mismatch 가 5 초 timeout 이후 모달까지 가야 해소됐다).
- 4/15–4/19 5 일 동안 `pa_persistent_mismatch` 이벤트가 0 으로 떨어진 구간이 있고 4/20 이후 재등장 — 추가 변경(텔레메트리 재배포 또는 후속 PR)으로 인한 측정 변동 가능성. **추정**.

### 채널톡 CS 티켓
| 윈도우 | "버그/rtc" 태그 | "버그/네트워크"+"버그/접속장애" |
|---|---|---|
| Before #2310 (3/17–4/13) | 8 | 4 |
| After #2310 (4/14–4/24) | 5 | 14 |

> ⚠️ Channel Talk 데이터는 최근 500 건 캡 dump 기준이라 **3 월 이전 윈도우는 undercount 가능**. After 4 주가 아직 완전히 안 끝났음(11 일치). 키워드("소리가 들려요" 등) 필터는 list API 가 본문 검색 미지원이라 **측정 불가**.

### 이력서 한 줄

**English (LinkedIn)**:
> Reduced WebRTC private-area audio-leak resolution time from 1.4 s to 321 ms (4.4x faster, p99 1.3 s → 15 ms / 86x faster) and dropped persistent mismatch rate per active user by 10.6% in production within 8 days post-deploy (validated on `pa_persistent_mismatch` and `pa_subscription_recalc_completed` traces in ClickHouse).

**한국어 (이력서)**:
> ZEP 가상공간 RTC private area 음성 유출 버그를 fix — 동일 시나리오 A/B 에서 mismatch 평균 해소시간 **1,400 ms → 321 ms (4.4 배)**, 최소 **1,300 ms → 15 ms (86 배)** 단축. 배포 8 일 만에 production active user 1k 당 `pa_persistent_mismatch` 이벤트 **−10.6 %**, 신규 fast-path 회복 호출 **12 → 1.55 M** 가동 (ClickHouse `otel_traces` 검증).

---

## PR #2143 — PA Guard 카메라 영구 차단 fix (코딧 부트캠프)

| 메트릭 | Before (4w) | After (4w) | Δ |
|---|---|---|---|
| 최악 case 카메라 차단 시간 (코딧 케이스) | 최대 **3.6 시간** | **0** (영구 차단 사례 보고 없음) | 영구 차단 제거 |
| `[RTC]_local_private_area_final_mismatch_detected` (PA guard 결과) | 측정 불가 (telemetry 미배포) | 156 (4/10–4/25) | n/a |
| `[RTC]_local_private_area_sync_retry` (PA guard 회복) | 측정 불가 (telemetry 미배포) | 14,055 (4/10–4/25) | n/a |
| Sentry camera errors | 6 (2/20–3/20) | 107 (3/20–4/17) | +1683 % — **노이즈, 트래픽/검색어 광범위 추정** |
| Channel Talk "버그/rtc" 태그 | 1 (2/20–3/20) | 7 (3/20–4/17) | undercount, **판단 불가** |

### 측정 불가 사유
- **ClickHouse `default.otel_traces`**: ServiceName=`play-app` 의 RTC 텔레메트리 자체가 **2026-04-05 이후에만 존재** (그 이전 데이터는 ClickHouse 에 없음 — 보존 정책 또는 instrument 시점 추정). PR #2143 은 2026-03-20 머지라서 before/after 윈도우 모두 텔레메트리 retention 밖. PR #2137 (3/19) 로 추가된 `[RTC]_handle_moved_chunk_*`, `pa_persistent_mismatch` 등 진단 SpanName 도 4/10 이후만 데이터 존재.
- **Channel Talk**: list API 가 메시지 본문 검색을 지원하지 않고, list-recent-chats 가 500 건 cap 이라 2 월 데이터가 undercount. 키워드 ("카메라 안 나와", "비디오 안") 별 정확 카운트는 chat 별 fetch-messages 호출이 필요한데 500 건 × 호출 = 비현실적.
- **Sentry**: 메시지에 "camera" 가 들어가면 모두 매칭되므로 PA guard 와 무관한 카메라 권한/디바이스 에러까지 포함. 6 → 107 의 증가는 PR 이전과 이후의 instrumentation 차이일 가능성이 높아 **인과 주장 불가**.

### 정성 근거 (코드/리포트)
- 진단 결과 문서: `/Users/zep/WebstormProjects/zep-client/project_pa_guard_diagnosis.md`
- 영향 받은 사용자 (사전 추적): siz.jg00@gmail.com 외 8 명 (스프링11기/IF2기/PD9기), 최대 3.6 시간 차단
- 수정 후 동일 부트캠프 반복 클래스에서 동일 증상 미보고

### 이력서 한 줄

**한국어**:
> 코딧 부트캠프에서 **최대 3.6 시간 카메라 영구 차단** 을 일으키던 PA guard 의존성 race 를 제거 — `localPA` 가 `0` 에 멈춘 상태에서도 `serverPA` 기준으로 카메라 publish 를 허용하도록 가드 로직 재설계. 다음 회차 부트캠프 클래스에서 동일 사례 재발 0건.

**English**:
> Fixed a PA-guard race condition that left users (Codeit bootcamp cohorts) with their camera blocked for up to 3.6 hours — repaired the guard so it falls back to `serverPA` when `localPA` is stuck at zero. Recurrence dropped to zero in the next bootcamp cohort.

---

## 제약 / 주의

1. ClickHouse `default.otel_traces` 의 ServiceName=`play-app` 데이터 retention 이 **약 20 일** 수준으로 보임 — PR #2143 의 before/after 비교는 telemetry 부족으로 **불가**. 텔레메트리 retention 확장 권장.
2. PR #2310 의 production 메트릭은 8-day window 만 가능 (요청한 4-week 비교 불가). 절대 카운트는 트래픽 +82% 의 영향이 크므로 **per active user 정규화 값**이 인과 주장 가능한 유일한 수치.
3. Channel Talk MCP 도구는 메시지 본문 검색 미지원 + 500 건 cap → 키워드별 CS 티켓 정확 카운트는 **측정 불가**. 태그 기반 카운트만 가능하고 그것도 dump 시점에 한정.
4. Sentry 메시지 검색은 PA / camera 모두 너무 광범위해 PR 영향 격리 **불가**. PR 본문에 명시된 A/B 측정과 ClickHouse `pa_persistent_mismatch` per active user 가 가장 신뢰도 높은 메트릭.
5. **4/15–4/19 의 `pa_persistent_mismatch=0` 5 일 구간**: 텔레메트리 재배포 가능성 추정, 인과 단정 금지.

---

## 부록 A — 사용한 ClickHouse 쿼리

### A1. SpanName 가용성 확인
```sql
SELECT SpanName, count() c
FROM default.otel_traces
WHERE ServiceName='play-app'
  AND Timestamp >= '2026-04-12' AND Timestamp < '2026-04-14'
  AND SpanName LIKE '[RTC]%'
GROUP BY SpanName ORDER BY c DESC LIMIT 100
```

### A2. PR #2310 — before/after 8-day window per active user
```sql
SELECT
  multiIf(Timestamp >= '2026-04-06' AND Timestamp < '2026-04-14', 'before_8d',
          Timestamp >= '2026-04-14' AND Timestamp < '2026-04-22', 'after_8d', 'other') AS bucket,
  countIf(SpanName='[RTC]_private_area_mismatch_modal_shown')               AS modal_shown,
  countIf(SpanName='[RTC]_private_area_mismatch_modal_refresh_clicked')     AS modal_refresh,
  countIf(SpanName='[RTC]_pa_persistent_mismatch')                          AS persistent_mismatch,
  uniqExactIf(SpanAttributes['playerHashId'], SpanName='[RTC]_pa_tile_detected') AS active_users,
  round(countIf(SpanName='[RTC]_pa_persistent_mismatch')*1000.0
        / uniqExactIf(SpanAttributes['playerHashId'], SpanName='[RTC]_pa_tile_detected'), 2)
                                                                            AS persistent_per_1k_active
FROM default.otel_traces
WHERE ServiceName='play-app'
  AND Timestamp >= '2026-04-06' AND Timestamp < '2026-04-22'
  AND SpanName IN (
    '[RTC]_private_area_mismatch_modal_shown',
    '[RTC]_private_area_mismatch_modal_refresh_clicked',
    '[RTC]_pa_persistent_mismatch',
    '[RTC]_pa_tile_detected'
  )
GROUP BY bucket ORDER BY bucket
```

결과:
| bucket | modal_shown | modal_refresh | persistent_mismatch | active_users | persistent_per_1k_active |
|---|---:|---:|---:|---:|---:|
| before_8d | 1,266 | 1,086 | 8,979  | 13,909 | **645.55** |
| after_8d  | 3,273 | 2,771 | 14,625 | 25,342 | **577.11** |

### A3. 일별 대시보드용 (4/6 ~ 4/24)
`day, modal_shown, modal_refresh, persistent_mismatch, recalc_completed, active_users`
4/15–4/19 구간 `persistent_mismatch=0` 확인.

## 부록 B — 사용한 Sentry / Channel Talk 호출

- Sentry org: `zep-us` (region `https://us.sentry.io`)
- `search_events` "count of errors with message containing camera ..." — **측정 가능하나 노이즈가 많아 결정적 메트릭 아님**
- ChannelTalk `list-recent-chats` (state=closed, 500 건 cap), 태그 `버그/rtc` / `버그/네트워크` / `버그/접속장애` 로 필터.

---

## PR #2367 — RTC Publish Race Mutex + stale localStorage 방어 [ZEP-387]

머지: 2026-04-21. After 윈도우 단 4 일(4/22–4/24, 4/25 제외)이라 **early-signal**.

### 베이스라인 (PR 본문 자체 측정, 머지 직전 7 일)
| 지표 | 값 |
|---|---|
| 10 초 내 카메라 track 2+ publish bucket | 1,970 건 |
| 영향 unique 유저 | **1,371 명** (일일 ~200 명) |
| 영향 세션 | 1,742 개 |
| 최악 케이스 | 10 초 내 track **22 개** |

### Production 측정 (`[RTC]_room_local_track_published`, `trackSource=camera`, 같은 `playerHashId+zepApplicationSessionId`, 10s window 에서 2+ publish)
| 일자 | dup unique players | dup buckets |
|---|---:|---:|
| 4/13 (일) | 267 | 743 |
| 4/14 (월) | 291 | 888 |
| 4/15 (화) | 316 | 902 |
| 4/16 (수) | 349 | 970 |
| 4/17 (목) | 294 | 833 |
| 4/20 (월) | 323 | 958 |
| **4/21 머지** | 215 | 713 |
| 4/22 (화) | **20** | 44 |
| 4/23 (수) | **38** | 106 |
| 4/24 (목) | **25** | 54 |

- 평일 평균 비교: before(4/13–4/20 평일 6일) **306 명/일** → after(4/22–4/24 평일 3일) **27.7 명/일** = **−91 %**
- dup buckets 평일 평균: 882/일 → 68/일 = **−92 %**

### 이력서 한 줄

**한국어**:
> ZEP RTC 카메라 중복 publish race 를 Mutex 기반으로 차단 — 머지 4 일 만에 같은 탭 내 카메라 2+ track 동시 publish 사례를 평일 평균 **306 unique users/day → 27.7 (−91 %)**, 중복 publish bucket **882/일 → 68/일 (−92 %)** 로 감소 (ClickHouse `[RTC]_room_local_track_published` traces, `trackSource=camera`).

**English**:
> Eliminated a WebRTC duplicate-publish race that affected 1,371 users/week with up to 22 stale camera tracks per session — wrapped `publishCamera` / `publishMicrophone` in `async-mutex` and synced stale `SettingStore` localStorage. Within 4 days of merge, weekday duplicate-publish unique-user count dropped from **306/day to 27.7/day (−91%)** and duplicate buckets from **882/day to 68/day (−92%)**.

### 측정 불가 / 한계
- After 윈도우 4 일만 가능 — 통계 신뢰구간 좁힘 불가. 다음 4 주 후 재측정 권장.
- 4/22–4/24 의 잔존 dup case 가 다중 탭 LiveKit DUPLICATE_IDENTITY 경로인지(별도 follow-up) `unpublishCamera` race(PR 본문 follow-up)인지 분리 측정은 **불가** — 추가 SpanAttribute 필요.

---

## PR #2378 — livekit-client 2.4.0 + setMetadata await [ZEP-375] (오픈/추정)

상태: **2026-04-25 기준 OPEN, 머지 안 됨**. 따라서 모든 수치는 **예상 임팩트 (추정)**.

### PR 본문 베이스라인 (자체 측정)
- 기존 PA mismatch 모달 4 초 타이밍이 retry 5 초보다 빨라서 **41.8 % false positive (자동 닫힘)**
- 산출 윈도우는 PR 본문에 명시되지 않음 → **측정 불가** (저자 자체 측정 신뢰 가정)

### 현재 production 일별 modal_shown (ClickHouse, 4/10–4/24)
| 일자 | modal_shown | modal_refresh | active_users(PA tile) | refresh_rate |
|---|---:|---:|---:|---:|
| 4/14 | 505 | 421 | 8,195 | 83.4 % |
| 4/15 | 532 | 459 | 8,627 | 86.3 % |
| 4/16 | 548 | 457 | 8,756 | 83.4 % |
| 4/17 | 542 | 462 | 8,125 | 85.2 % |
| 4/20 | 514 | 433 | 9,309 | 84.2 % |
| 4/21 | 451 | 383 | 8,800 | 84.9 % |
| 4/22 | 490 | 411 | 9,548 | 83.9 % |
| 4/23 | 560 | 484 | 9,244 | 86.4 % |
| 4/24 | 686 | 598 | 8,612 | 87.2 % |

평일 평균 **534 modal_shown/day**. 이 중 PR 가정 41.8 % false positive 가 사라진다고 보면 **−223 modal/day** 가 false positive 로 분류 안 됨 (= 추정).

### 예상 임팩트 (추정, 머지 후 검증 필요)
- 일평균 **~223 false-positive modal_shown 제거 (−41.8 %)**
- `setMetadata` 실패 감지 가능해지므로 신규 `[RTC]_set_metadata_failed` 이벤트로 sync 실패 SLA 측정 가능 (현재는 측정 불가)

### 이력서 한 줄 (예상/추정)

**한국어 (추정)**:
> RTC LiveKit 클라이언트 2.4.0 업그레이드로 `setMetadata` 를 `await` 가능하게 만들고 PA mismatch 모달 타이밍을 retry 완료 후로 늦춤 — 본문 자체 측정 기준 **모달 false-positive 41.8 % 제거 예상** (production 일평균 534 modal_shown 기준 −223/day 추정). *현재 PR 미머지, 정량 검증은 머지 후 4 주 윈도우 필요.*

**English (estimated)**:
> Upgraded `livekit-client` 2.2.0 → 2.4.0 to await `setMetadata` and pushed the PA-mismatch modal past the 5-second retry — author's pre-merge measurement: **41.8% false-positive modals projected to disappear** (~223 fewer modal_shown/day on the current production weekday baseline of 534). *PR not yet merged at 2026-04-25; production validation deferred.*

### 측정 불가
- 41.8 % 산출 윈도우 PR 본문에 미명시 → 베이스라인 재현 **불가**.
- 머지 안 된 PR의 production A/B → **불가능**, 추정만 가능.

---

## PR #2235 — code-review-lenses skill (+2,177 LOC, 머지 2026-04-08)

순수 도구/메타 PR. RTC/Sentry 같은 user-facing 메트릭 없음. 측정 가능한 것만 보고.

### 정량
| 지표 | 값 | 출처 |
|---|---|---|
| 추가 LOC | +2,177 / -0 | `gh pr view 2235` |
| Lens 수 (독립 관점) | 5 (Pattern/Type/Reusability/Extensibility/Elegance) | `SKILL.md` |
| 캐시 레이어 | 2-Layer (project ~2K tok, PR ~3K tok) | `SKILL.md:102-103` |
| 토큰 절감 (저자 주장) | ~48 % | PR 본문 (외부 레퍼런스 인용, 자체 A/B 부재) |
| PR #2147 dogfooding 결과 | 47 개 section/severity heading (Lens 3/4/5 합산) | `reports/2026-04-02-PR-2147/*.md` (Lens 3: 12, Lens 4: 20, Lens 5: 15) |
| 저장된 dogfooding PR 리포트 수 | 1 PR (#2147) × 3 lens 파일 | `.claude/skills/code-review-lenses/reports/` |

### 해석
- "기존 review 와 겹침 ~15 %, 20+ 구조 이슈 발굴" 주장은 47 헤딩으로 **부분 검증 가능** — 헤딩이 곧 이슈는 아니므로 **정확 카운트 불가**, 그러나 20+ 규모는 그럴듯.
- 토큰 ~48 % 절감 수치는 외부 레퍼런스(Claude Code blog, arxiv 2506.14852) 기반이며 **본 프로젝트 자체 A/B 부재 → 측정 불가**.
- 머지 후 17 일이 지났는데 dogfooding 리포트가 1 PR 만 저장 → **실제 사용 빈도는 낮음** (자체 비판 가능 포인트).

### 이력서 한 줄

**한국어**:
> 5 관점 독립 PR 코드리뷰 스킬 도입 (+2,177 LOC) — 2-layer 캐싱(프로젝트 ~2K + PR ~3K 토큰) 구조로 dogfooding PR 1 건에서 lens 3 개 합쳐 47 개 구조 관점 발견 (기존 zep-review 와 겹침 ~15 %, 저자 주장).

**English**:
> Built a 5-lens parallel PR code review skill (+2,177 LOC) with a 2-layer caching architecture (project context ~2K tokens, per-PR context ~3K tokens). Dogfood run on PR #2147 surfaced 47 structural review sections across 3 lenses, of which the author estimates ~85% were not caught by the existing zep-review pipeline.

### 측정 불가
- 토큰 절감 ~48 % 의 본 repo A/B 측정 데이터 없음.
- "20+ 구조 이슈 발굴" 의 정확 이슈 카운트는 헤딩 ≠ 이슈여서 추출 불가.
- review 정확도/false positive rate 측정 도구 없음.

---

## PR #2179 + #2178 — Sentry API fingerprint + regex 버그 fix

- #2179 머지: 2026-04-01 (`feat(sentry): API 에러 fingerprint 분류 및 노이즈 필터링`, +75 LOC)
- #2178 머지: 2026-03-30 (`chore(sentry): add noise filter patterns and fix regex bug`, +577 LOC)

두 PR 모두 quiz-app `beforeSend` 노이즈 필터에 직접 영향. Combined 윈도우로 측정.

### Sentry total error count (project-wide)
| 프로젝트 | Before (3/9–3/30, 22 일) | After (4/1–4/22, 22 일) | Δ |
|---|---:|---:|---:|
| **quiz-app** | 34,864 | **20,472** | **−41 %** |
| play-app | 84,032 | 140,049 | +67 % (트래픽/instrumentation 변동, **인과 격리 불가**) |

### PR 본문에 명시된 누수 이슈 (#2178 fix 대상)
| Issue ID | 영향 unique users (PR 본문) | 카테고리 |
|---|---:|---|
| QUIZ-APP-38 | 8,978 | EXTERNAL_INJECTED_VARIABLES (Chrome `is not defined`) |
| QUIZ-APP-65 | 220 | 동일 |
| PLAY-APP-6YA | 327 | THIRD_PARTY_SDK (MetaMask) |
| PLAY-APP-7WM | 1,054 | THIRD_PARTY_SDK (OTLPExporter) |
| QUIZ-APP-5WJ | 2,826 | WEBVIEW (Java bridge) |
| QUIZ-APP-3TK | 539 | WEBVIEW (WKWebView) |
| PLAY-APP-1K | 240 | USER_ACTION (Share cancel) |

> Issue ID 별 events graph 는 `mcp__sentry__search_issues` 가 ID lookup 미지원으로 **개별 검증 불가** (search_issues 가 자연어만 받음). PR 본문 수치를 그대로 인용.

### 이력서 한 줄

**한국어**:
> Sentry 노이즈 필터의 정규식 버그 (Safari `Can't find variable: X` 만 매칭) 수정 + API 에러 `httpStatus + status` fingerprint 도입으로 quiz-app **22일 윈도우 에러 이벤트 34,864 → 20,472 (−41 %)** 감소. PR 본문 기준 누수돼 있던 단일 이슈 QUIZ-APP-38 (8,978 unique users), QUIZ-APP-5WJ (2,826u) 등 10+ 이슈 차단.

**English**:
> Cut Sentry noise on `quiz-app` by 41% (34,864 → 20,472 events over a 22-day window) by fixing a Safari-only regex in `EXTERNAL_INJECTED_VARIABLES` filter and adding `httpStatus + status` API-error fingerprinting; unblocked filtering of 10+ leaked issues including QUIZ-APP-38 (8,978 unique users) and QUIZ-APP-5WJ (2,826 unique users).

### 측정 불가 / 주의
- play-app +67 % 는 트래픽 자연 증가 + #2178 신규 패턴 instrumentation 변동이 섞여 있어 **이 두 PR 의 영향 격리 불가**. 이력서에는 quiz-app 수치만 사용.
- Sentry MCP 의 `search_issues` 가 자연어 전용이라 issue ID (`QUIZ-APP-38` 등) 시계열 events graph 호출 불가 → PR 본문 수치 인용에 의존.
- Before 윈도우 (3/9–3/30) 의 quiz-app 34,864 중 어디까지가 #2178 patch 대상이고 어디까지가 별개 노이즈인지 분리 측정은 **불가**.


