---
title: Career / Resume Hub
status: active
last_updated: 2026-04-26
---

# 🎯 Career & Resume

**섹션·도메인 분리 기반 누적 관리** 이력서 시스템. 마크다운 + git.

## 📁 구조

```
career/
├── README.md                       # 이 파일
├── meta/                           # 메타·평가·갱신
│   ├── evaluation.md               # 강점/약점 + Codex 평가
│   ├── voice.md                    # 본인 인터뷰 (1차 자료)
│   ├── backlog.md                  # 보강 액션 (블로그/OSS/스모크)
│   └── changelog.md                # 갱신 이력
├── metrics/                        # 📊 정량 지표 전용
│   ├── README.md
│   ├── rtc.md                      # PA, race condition
│   ├── observability.md            # Sentry
│   ├── infra.md                    # CI/CD, self-hosted, 인시던트
│   ├── product.md                  # SEO, web vitals
│   └── unverified.md               # 측정 불가/약한 주장
├── profile/                        # 정적 프로필
│   ├── contact.md
│   ├── intro.md                    # intro 후보 + 채택 버전
│   ├── skills.md
│   ├── education.md
│   └── other.md                    # 멘토링 등
├── experience/                     # 경력
│   ├── zep/                        # 메인 (도메인별)
│   │   ├── rtc.md
│   │   ├── ai-ops.md
│   │   ├── cicd.md
│   │   ├── devex.md
│   │   ├── quiz.md
│   │   └── product.md
│   ├── suprema.md
│   └── cloudstone.md
└── oss/
    ├── company.md                  # zep-us 조직 레포
    └── personal.md                 # lucid-jin 개인 레포
```

## 🔄 갱신 룰

- **분기별 1회 전체 점검** (3/6/9/12월 마지막 주)
- **PR 머지 후 즉시** `metrics/<domain>.md`에 정량 추가 → 1주 후 production 검증값 업데이트
- 섹션 수정 시 `meta/changelog.md`에 1줄 + frontmatter `last_updated` 갱신
- intro 변경 시 `profile/intro.md`에 새 후보 추가 (이전 버전 archive 보존)

## 🎯 정체성 한 줄

> 고치기 전에 측정하고, 같은 문제가 두 번 반복되지 않도록 구조를 바꾸는 7년차 프론트엔드 개발자.
> (인터뷰 후 v4 후보: "라이브에 먼저 측정값을 띄우고, 데이터로 가설을 좁힌 뒤에 코드를 만진다.")

## 📌 우선 보강 액션 (TOP 3, `meta/backlog.md` 참고)

1. **RTC E2E 스모크 테스트 PoC #2385 마무리 머지** — 회귀 방어 인프라 사례
2. **블로그 1편 (PR #2310) + `/sentry-noise-filter` 또는 `mcpick` OSS README/데모 보강** — 외부 가시성 0 → 1
3. **CS 티켓 정량화 자동 리포트** — 비즈니스 메트릭 빈자리

## 🔗 출처

1차 분석 산출물 (이 폴더 `_raw/`로 이관):
- `_raw/resume-handoff-2026-04-26.md` — 핸드오프 단일 문서 (TL;DR/시그니처/임팩트/intro/체크리스트)
- `_raw/impact-metrics-2026-04.md` — before/after 정량 측정 + ClickHouse 쿼리 부록
- `_raw/resume-strengths-2026-04.md` — 영역별 강점 + STAR 불릿 12개

## 📝 갱신 이력

상세는 `meta/changelog.md` 참고.

- 2026-04-26: 신규 작성 + 구조 정리(meta/metrics/profile/experience/oss) + 인터뷰 캡처 + 개인 OSS 6개 추가
