---
created: 2026-01-17
related: [[2026-01-17-Claude-Code-Skills]], [[Claude Code]]
status: complete
---

#public
#programming #ai #claude #howto #architecture

# Claude Code 플러그인 (Plugins)

## 핵심 요약
> 플러그인은 슬래시 명령어, 에이전트, 훅, 스킬, MCP 서버를 패키징해서 팀/커뮤니티와 공유하는 방법이다. 개인용(`.claude/`)과 달리 버전 관리와 배포가 가능하다.

## 플러그인이란?

### 정의
- Claude Code 기능을 확장하고 **공유 가능한 형태로 패키징**한 것
- 명령어, 스킬, MCP, 훅 등을 한 번에 묶어서 배포
- Git으로 버전 관리, 마켓플레이스로 배포

### 개인용 vs 플러그인

| 구분 | 개인용 (독립 실행형) | 플러그인 (공유용) |
|------|---------|---------|
| **저장 위치** | `.claude/` 디렉토리 | `.claude-plugin/plugin.json` |
| **명령어 형식** | `/hello` | `/plugin-name:hello` |
| **사용 시기** | 개인 워크플로우 | 팀/커뮤니티 공유, 여러 프로젝트 |
| **버전 관리** | X | O (plugin.json) |
| **배포** | X | 마켓플레이스 가능 |
| **프리픽스** | 불필요 | 필수 (`/플러그인:명령어`) |

## 플러그인 구조

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json          # 플러그인 메타데이터 (필수)
├── commands/                # 슬래시 명령어
│   └── hello.md
├── agents/                  # 사용자 정의 에이전트
├── skills/                  # 에이전트 스킬
│   └── review.md
├── hooks/                   # 이벤트 핸들러
│   └── hooks.json
├── .mcp.json               # MCP 서버 설정
└── .lsp.json               # LSP 서버 설정
```

### plugin.json 예시
```json
{
  "name": "my-plugin",
  "description": "팀용 커스텀 도구",
  "version": "1.0.0",
  "author": {
    "name": "Your Team"
  }
}
```

## 플러그인으로 할 수 있는 일

### ✅ 가능한 것

| 기능 | 설명 | 프리픽스 필요 |
|------|------|------------|
| **슬래시 명령어** | `/plugin:setup`, `/plugin:deploy` | ✓ 필수 |
| **MCP 서버 통합** | Sentry, Linear, GitHub 등 자동 로드 | X (글로벌 사용) |
| **스킬 추가** | 자동 발견되고 Claude가 알아서 사용 | X (글로벌 사용) |
| **에이전트** | 특정 작업 전담 에이전트 | - |
| **훅(Hooks)** | 도구 실행 전/후 자동 실행 | - |
| **LSP 서버** | 언어별 코드 분석 도구 | - |
| **버전 관리** | Git으로 변경 이력 추적 | - |
| **팀 공유** | Git/마켓플레이스로 배포 | - |
| **외부 플러그인 재배포** | 다른 플러그인을 마켓플레이스에 포함 | - |

### ❌ 불가능한 것

| 제약사항 | 대안 |
|---------|------|
| 프리픽스 없는 명령어 | 짧은 이름 사용 (`/ct:setup`) |
| API 키 자동 배포 | 사용자가 환경변수로 직접 설정 |
| 의존성 자동 해결 | README에 수동 설치 명시 |
| 파일 시스템 직접 수정 | Claude에게 요청하거나 훅 사용 |
| Claude Code 설정 변경 | 사용자가 직접 변경 |
| 프로젝트별 자동 활성화 | 수동 설치/활성화 필요 |

## MCP와 스킬의 중요한 특징

### 🎯 핵심: 자동 로드 & 글로벌 사용

플러그인에 포함된 **MCP 서버와 스킬은 프리픽스 없이** 사용 가능!

```
플러그인 설치 후:

# MCP 서버 (자동 로드, 글로벌)
> Sentry에서 최근 에러 확인해줘
✓ 플러그인의 MCP를 자동으로 사용

# 스킬 (자동 로드, 글로벌)
> 이 코드 검수해줘
✓ 플러그인의 review.md 스킬 자동 사용

# 명령어만 프리픽스 필요
/company-tools:setup
```

### 로드 방식 비교

| 구성 요소 | 자동 로드 | 글로벌 사용 | 프리픽스 |
|---------|---------|---------|---------|
| **MCP 서버** | ✓ | ✓ | X 불필요 |
| **스킬** | ✓ | ✓ | X 불필요 |
| **명령어** | ✓ | - | ✓ 필수 |

## 플러그인 vs "즐겨찾기" 방식

회사 컴과 개인 컴만 동기화할 때 고려할 두 가지 방법:

### 방법 1: 플러그인 (권장)

```
company-tools/
├── .claude-plugin/plugin.json
├── .mcp.json              # 팀 공용 MCP
├── skills/                # 팀 공용 스킬
└── commands/              # 팀 명령어
```

**장점:**
- MCP/스킬은 Claude가 알아서 사용 (프리픽스 불필요)
- 명령어만 `/company:setup` 형태 (명시적, 충돌 방지)
- Git으로 버전 관리
- API 키 포함 안 됨 (안전)

**사용:**
```bash
git clone https://company-git.com/company-tools
/plugin marketplace add ./company-tools
/plugin install company-tools
```

### 방법 2: "즐겨찾기" (간단)

```
company-favorites/
├── mcp/shared-mcp.json
├── skills/
└── install.sh
```

**장점:**
- 설정 간단
- 모든 명령어 프리픽스 불필요

**단점:**
- 버전 관리 어려움
- 배포 불가

**사용:**
```bash
git clone [url] ~/favorites
./install.sh  # ~/.claude/로 복사
```

## 플러그인 개발 워크플로우

### 1단계: 기본 구조
```bash
mkdir my-plugin
cd my-plugin
mkdir -p .claude-plugin commands skills
```

### 2단계: plugin.json 작성
```json
{
  "name": "my-plugin",
  "description": "설명",
  "version": "1.0.0",
  "author": { "name": "Your Name" }
}
```

### 3단계: 구성 요소 추가
- `commands/` - 슬래시 명령어
- `skills/` - 스킬 (자동 로드됨)
- `.mcp.json` - MCP 서버 (자동 로드됨)

### 4단계: 테스트
```bash
claude --plugin-dir ./my-plugin
```

### 5단계: Git으로 공유
```bash
git init
git add .
git commit -m "Initial plugin"
git push origin main
```

## 플러그인 마켓플레이스

### 여러 플러그인을 묶어서 배포

```
company-marketplace/
├── .claude-plugin/
│   └── marketplace.json
└── plugins/
    ├── plugin-a/
    └── plugin-b/
```

**marketplace.json:**
```json
{
  "name": "company-tools",
  "owner": { "name": "Your Team" },
  "plugins": [
    {
      "name": "internal-tools",
      "source": "./plugins/plugin-a"
    },
    {
      "name": "external-helper",
      "source": {
        "type": "github",
        "repo": "other/plugin"
      }
    }
  ]
}
```

## 나의 생각

### 이해한 것
- 플러그인 = 스킬/MCP/명령어를 패키징하는 방식
- 명령어는 프리픽스가 필요하지만, **MCP와 스킬은 자동으로 글로벌하게 작동** (이게 핵심!)
- 개인용과 플러그인의 차이는 "공유"와 "버전 관리"
- [[2026-01-17-Claude-Code-Skills|스킬]]은 플러그인의 구성 요소 중 하나

### 적용 아이디어
- 회사 팀용 플러그인 만들기
  - `.mcp.json`: Sentry, Linear, 내부 DB
  - `skills/`: 코드 검수, 배포 절차
  - `commands/`: 프로젝트 초기화
- Git으로 관리하면 회사 컴 ↔ 개인 컴 동기화 가능
- 서드파티 없이 회사 Git만 사용

### 의문점과 답변

**Q: 프리픽스가 불편한데 피할 수 없나?**
- 명령어는 불가능 (플러그인 간 충돌 방지 위해)
- 하지만 MCP와 스킬은 프리픽스 없이 자동 사용됨!
- 짧은 플러그인 이름으로 완화: `/ct:setup`

**Q: API 키는 어떻게 공유?**
- 플러그인에 직접 포함 불가 (보안)
- 환경변수로 사용자가 직접 설정
- `.mcp.json`에서 `${API_KEY}` 형태로 참조

**Q: 플러그인 vs 즐겨찾기, 뭐가 나을까?**
- **플러그인 추천**: MCP/스킬 많이 쓰면
- 즐겨찾기: 간단하게 공유만 하면 될 때

## 연결
- 관련 노트: [[2026-01-17-Claude-Code-Skills]], [[2026-01-17-Claude-Skill-Structure]], [[Claude Code]]
- 참고: https://code.claude.com/docs/ko/plugins
- 키워드: #플러그인 #MCP #스킬공유 #팀협업

## 실습 계획
- [ ] 회사용 플러그인 저장소 만들기
- [ ] `.mcp.json`에 현재 사용 중인 MCP 서버 추가
- [ ] `skills/`에 자주 쓰는 스킬 정리
- [ ] Git으로 회사 서버에 푸시
- [ ] 개인 컴에서 클론해서 테스트
