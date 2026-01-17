---
created: 2026-01-17
related: [[2026-01-17-Claude-Code-Skills]]
status: complete
---

#programming #claude #architecture #howto

# Claude Code 스킬 구조화 베스트 프랙티스

## 핵심 요약
> 스킬은 하나의 거대한 파일보다 **SKILL.md + 참조 파일**로 분리하는 게 좋다. 이유는 컨텍스트 효율성과 필요할 때만 로드하는 Progressive Disclosure 때문.

## 왜 분리해야 하는가?

### 1. 컨텍스트 윈도우 효율성
- Claude는 **컨텍스트 윈도우**(대화 기억 용량)가 한정됨
- 스킬 파일이 크면 → 항상 전체가 로드됨 → 다른 작업에 쓸 공간 감소
- 분리하면 → 핵심만 로드 → 상세는 필요할 때만

```
❌ 큰 파일: 항상 500줄 로드
✅ 분리: 기본 60줄 + 필요시 추가 로드
```

### 2. Progressive Disclosure (점진적 공개)
- 모든 정보를 한번에 주지 않음
- 필요한 정보만 필요한 시점에 제공
- Claude가 참조 링크를 보고 필요하면 해당 파일을 읽음

### 3. 유지보수 용이성
- 읽기 로직 변경 → READ.md만 수정
- 작성 규칙 변경 → WRITE.md만 수정
- 서로 영향 없음

## 구조화 방법

### 권장 구조
```
my-skill/
├── SKILL.md          # 개요 + 핵심 규칙 (500줄 이하)
├── REFERENCE1.md     # 상세 가이드 1
├── REFERENCE2.md     # 상세 가이드 2
└── scripts/          # 유틸리티 스크립트
```

### SKILL.md에서 참조하는 법
```markdown
## 상세 가이드
- 검색 방법: [READ.md](READ.md)
- 작성 규칙: [WRITE.md](WRITE.md)
```

Claude가 이 링크를 보고 필요할 때 해당 파일을 읽음.

### 주의: 참조는 1단계만
```
✅ SKILL.md → READ.md (1단계)
❌ SKILL.md → READ.md → DETAIL.md (2단계 이상)
```
깊은 참조는 Claude가 일부만 읽을 수 있음.

## 분리 vs 통합 판단 기준

| 상황 | 추천 |
|------|------|
| 스킬이 100줄 이하 | 통합 (하나의 SKILL.md) |
| 스킬이 500줄 이상 | 분리 필수 |
| 역할이 명확히 다름 (읽기/쓰기) | 분리 |
| 자주 쓰는 핵심 vs 가끔 쓰는 상세 | 분리 |

## 실습: 옵시디언 스킬 분리

### Before (152줄 단일 파일)
```
obsidian/
└── SKILL.md  ← 검색 + 작성 + 템플릿 전부
```

### After (분리)
```
obsidian/
├── SKILL.md   (60줄)  ← 핵심만
├── READ.md    (80줄)  ← 검색/읽기
└── WRITE.md   (120줄) ← 작성 규칙
```

### 효과
- 기본 로드: 152줄 → 60줄 (60% 감소)
- 검색만 할 때: READ.md만 추가 로드
- 작성할 때: WRITE.md만 추가 로드

## 나의 생각

### 이해한 것
- 분리 = 효율성 + 유지보수성
- "필요할 때만 로드"가 핵심 원리
- [[Lazy Loading]]과 같은 개념

### 적용 아이디어
- 코드 리뷰 스킬 만들 때도 분리 적용
- SKILL.md = 체크리스트, DETAIL.md = 상세 기준

### 의문점
- 참조 파일이 너무 많아지면 관리가 어려워지지 않을까?
- 적정 분리 개수는? (3-5개 정도가 적당해 보임)

## 연결
- 관련 노트: [[2026-01-17-Claude-Code-Skills]], [[Progressive Disclosure]], [[Lazy Loading]]
- 참고: Claude Code 공식 스킬 가이드
