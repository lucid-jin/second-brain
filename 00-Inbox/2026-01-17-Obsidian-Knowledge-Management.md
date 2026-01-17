---
created: 2026-01-17
related: [[옵시디언]], [[지식관리]], [[PARA]], [[생산성]]
status: complete
---

#public
#productivity #knowledge-management #obsidian #methodology

# 옵시디언 지식 관리 방법론

## 핵심 요약
> 내 옵시디언 공식 관리 방법: PARA 폴더 구조 + 인박스 빠른 캡처 + 주간 정리. 완벽보다 지속 가능성이 우선.

## 주요 내용

### 1. PARA 폴더 구조

```
00-Inbox/      📥 빠른 캡처 → 주간 정리로 비우기
01-Projects/   🎯 명확한 목표/마감일 있는 것
02-Areas/      🔄 지속적 관심사 (프로그래밍, AI 등)
03-Resources/  📚 참고 자료 (튜토리얼, 레퍼런스)
04-Archive/    📦 완료/비활성
private/       🔒 비공개 (GitHub 제외)
```

**분류 기준:**
- Projects: 마감일 있음? 완료 기준 명확?
- Areas: 계속 발전시킬 영역?
- Resources: 나중에 찾아볼 참고 자료?
- Archive: 더 이상 안 씀?

### 2. 인박스 정리 루틴

**주간 정리 (필수) - 일요일 저녁 30분:**
1. 인박스 확인 (5분)
2. `status: complete` 노트만 분류 (20분)
   - draft는 다음 주로
3. 파일 이동 + GitHub 커밋 (5분)

**분류 질문:**
```
완성됨? (status: complete)
  ├─ No → 인박스 유지
  └─ Yes → 폴더 분류
            ├─ 마감일 있음? → Projects
            ├─ 지속 관심사? → Areas
            ├─ 참고 자료? → Resources
            └─ 안 씀? → Archive
```

### 3. 태그 체계

**필수 태그:**
```markdown
#public 또는 #private  ← 첫 줄 필수!
#분야 #유형             ← 두 번째 줄
```

**태그 종류:**
- 공개: `#public` / `#private`
- 분야: `#programming` `#ai` `#career`
- 유형: `#concept` `#howto` `#reference`
- 상태: `#todo` `#learning` `#done`

### 4. 노트 작성 규칙

**파일명:**
- 학습 노트: `YYYY-MM-DD-주제.md`
- 영구 노트: `제목.md`

**템플릿:**
```markdown
---
created: YYYY-MM-DD
related: [[노트1]]
status: draft
---

#public
#분야 #유형

# 제목

## 핵심 요약
> 한 줄 정리

## 주요 내용
- 포인트 1
- [[연결]]

## 나의 생각
- 이해:
- 의문:
- 적용:

## 연결
- 관련: [[노트]]
```

### 5. 핵심 원칙

**완벽주의 피하기:**
- 80% 정확하면 OK
- 나중에 다시 옮길 수 있음
- 중요한 건 **인박스를 비우는 것**

**빠른 캡처 > 완벽한 정리:**
- 일단 인박스에 저장
- 정리는 주말에
- 생각 놓치지 않기가 우선

**연결이 핵심:**
- 고립된 노트 = 죽은 노트
- 최소 2개 이상 백링크
- "연결" 섹션 필수

## 나의 생각

### 이해한 것
- PARA = Projects/Areas/Resources/Archive의 약자
- 핵심은 **인박스를 비우는 것** - 쌓아두면 무용지물
- 주간 정리가 필수 - 한 달 방치하면 정리 불가
- [[Zettelkasten]]과 비슷한 개념 - 연결 중심
- 완벽주의가 가장 큰 적 - 80%면 충분

### 이해한 배경
- PARA는 Tiago Forte의 "Building a Second Brain"에서 나온 개념
- 폴더 구조를 **행동 기반**으로 설계 (주제 기반 X)
- Projects: 단기적, 완료 가능
- Areas: 장기적, 계속 유지
- Resources: 수동적 참조
- Archive: 완료

### 적용 아이디어
- **Claude Code 스킬 활용** - "인박스 정리해줘" 자동화
- 주간 정리를 캘린더에 등록 (일요일 저녁)
- 월간 대청소도 루틴화 (마지막 주)
- 서브폴더 예시:
  - `02-Areas/Programming/Claude/`
  - `02-Areas/AI-Learning/`
  - `01-Projects/사이드프로젝트/`

### 의문점
- 서브폴더를 너무 깊게 만들면?
- 태그와 폴더가 겹치는데 어느 걸 우선?
- 노트가 여러 영역에 걸치면?

### 의문점 답변

**Q: 서브폴더를 너무 깊게 만들면?**
- 최대 2단계 권장 (`02-Areas/Programming/Claude/`)
- 더 깊으면 찾기 어려움
- 태그로 보완

**Q: 태그 vs 폴더 우선순위?**
- **폴더 = 행동 기반** (어떻게 사용할지)
- **태그 = 주제 기반** (무엇에 관한지)
- 예: `02-Areas/Programming/` + `#claude #ai`
- 둘 다 사용하되, 폴더가 1차 분류

**Q: 노트가 여러 영역에 걸치면?**
- 80% 법칙 - 가장 관련 높은 곳에
- 다른 영역에서는 백링크로 참조
- 예: Claude Code 노트 → `02-Areas/Programming/` + `02-Areas/AI-Learning/`에서 백링크

## 연결
- 관련 노트: [[2026-01-17-Claude-Code-Plugins]], [[2026-01-17-Claude-Code-Skills]], [[생산성 시스템]]
- 참고: PARA 방법론 (Tiago Forte), Zettelkasten
- 키워드: #지식관리 #PARA #인박스정리 #생산성

## 실습 계획
- [ ] 폴더 구조 만들기 (Projects, Areas, Resources, Archive)
- [ ] 현재 인박스 노트 3개 분류해보기
- [ ] 주간 정리 캘린더 등록 (일요일 저녁)
- [ ] Claude 스킬로 자동 정리 테스트
