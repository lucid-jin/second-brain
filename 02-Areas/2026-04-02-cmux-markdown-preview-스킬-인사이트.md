---
tags:
  - claude-code
  - skill
  - cmux
  - insight
created: 2026-04-02
---

# cmux-markdown-preview 스킬을 만들면서 배운 것

## 배경

cmux 환경에서 마크다운 파일을 편집하면서 실시간 프리뷰를 보고 싶었다. Claude Code 한 세션에서 스킬 설계 → 구현 → 테스트 → 버그픽스 → 3번의 메이저 개선 → Zeude 배포 → GitHub 푸시 → 슬랙 공유까지 전체 라이프사이클을 돌렸다.

## 핵심 인사이트

### 1. 대화형 점진적 개발 > PRD 선행 방식

PRD를 먼저 작성하고 구현하는 것보다, 대화하면서 점진적으로 기능을 추가하는 게 훨씬 빠르다. 요구사항이 대화 중에 구체화되기 때문이다.

- v1: 기본 프리뷰 (meta refresh, 고정 포트)
- v2: 인터랙티브 Edit 버튼, 파일별 포트, 세션 제한
- v3: fetch 기반 무깜빡임 갱신, 이벤트 위임, 라인 번호 프롬프트

각 버전이 이전 버전의 실제 사용 피드백에서 나왔다.

### 2. GUI ↔ CLI 브릿지 패턴

cmux의 브라우저 패널이 단순 뷰어가 아니라 **입력 인터페이스**가 될 수 있다.

```
[프리뷰 GUI] → Edit 클릭 → 피드백 입력
     ↓
[클립보드] → 파일:라인 프롬프트 복사
     ↓
[Claude Code CLI] → Cmd+V → 해당 섹션 즉시 개선
```

**"이 부분"을 마우스로 가리킬 수 있는 것**이 말로 설명하는 것보다 정확하다. 이건 Figma의 Dev Mode와 같은 패턴이다.

### 3. fetch 기반 리프레시의 중요성

`<meta http-equiv="refresh">`는 전체 리로드 → 깜빡임 + 스크롤 초기화 + UI 상태 소멸.

`fetch` + DOM diff는:
- 변경분만 교체 → 깜빡임 없음
- 스크롤 위치 유지
- 열려있던 입력창, 타이핑 중인 내용 보존

**인터랙티브 요소가 있는 프리뷰는 반드시 fetch 방식**이어야 한다.

### 4. 기능 통합의 원칙

처음에 Improve 버튼 + Comment 버튼 2개를 만들었다가, Comment가 Improve의 상위호환임을 깨닫고 Edit 하나로 통합했다.

> 하나가 다른 하나의 상위호환이면 합쳐야 한다.

빈칸 Enter = "개선해줘" (= Improve), 입력 후 Enter = 구체적 피드백 (= Comment).

### 5. 스킬이 스킬을 배포하는 메타 워크플로우

```
cmux-markdown-preview (이번에 만든 스킬)
     ↓ push-skill-zeude
Zeude Dashboard
     ↓ gh repo push  
GitHub (claude-toolkit)
     ↓ slack-agent
슬랙 공유 (6_ai_잘써보기)
```

배포 파이프라인 자체가 스킬 조합으로 동작한다. "업로드해줘" 한마디면 3곳에 동시 배포 가능.

## 기술적 결정들

| 결정 | 이유 |
|------|------|
| 파일별 고유 포트 (md5 해시) | 세션 간 충돌 방지, 가장 단순한 격리 |
| 최대 3개 동시 세션 | 메모리 보호, 처음부터 안전장치 |
| render_html.py 분리 | serve-markdown.sh 안에 inline Python은 린터/권한 문제 발생 |
| 이벤트 위임 | DOM 교체 후에도 버튼 동작 유지 |

## 관련 링크

- Zeude: https://cc.zep.works/admin/skills
- GitHub: https://github.com/lucid-jin/claude-toolkit/tree/main/skills/cmux-markdown-preview
