---
created: 2026-02-12
related: []
status: done
---

#public #programming #infra #howto

# Docker 네트워크와 DNS 디버깅

## 핵심 요약
> 같은 서버에 있는 Docker 컨테이너끼리도 네트워크가 분리되어 있으면 직접 통신이 안 되고, 외부 프록시(Cloudflare)를 거쳐야 한다. 이 과정에서 간헐적 연결 문제가 발생할 수 있다.

## 배경 (실제 사례)
Coolify로 배포된 Dashboard와 Supabase가 같은 서버에 있는데, Dashboard에서 Supabase로 요청 시 간헐적 `ETIMEDOUT` 발생.

## Docker 네트워크 격리

### 기본 개념
- Docker는 서비스마다 **별도 네트워크**를 생성
- Coolify에서 **프로젝트가 다르면** Docker 네트워크도 분리됨
- 같은 네트워크에 있어야 컨테이너 이름으로 직접 통신 가능

### 확인 방법
```bash
# 컨테이너 안에서 다른 컨테이너에 접근 시도
wget -qO- --timeout=5 http://[컨테이너이름]:[포트]/
# "bad address" → 다른 네트워크 (통신 불가)
# 응답 있음 → 같은 네트워크 (통신 가능)
```

### 같은 서버인데 외부를 돌아가는 구조
```
Dashboard 컨테이너 → Cloudflare → 같은 서버 → Supabase 컨테이너
(바로 옆인데 밖을 한 바퀴 돌아옴)
```

### 해결 방법
1. 두 서비스를 **같은 Docker 네트워크**에 연결
2. `SUPABASE_URL`을 내부 주소로 변경 (예: `http://supabase-kong:8000`)
3. 또는 Cloudflare DNS를 **DNS-only(회색 구름)**로 변경

## Cloudflare 프록시

### 프록시 ON (주황 구름)
- 실제 서버 IP가 숨겨지고, Cloudflare IP가 반환됨
- DDoS 방어, 캐싱, SSL 자동 관리
- 단점: 같은 서버 내 컨테이너 통신도 Cloudflare를 거침

### 프록시 OFF (회색 구름, DNS-only)
- 실제 서버 IP가 직접 노출됨
- Cloudflare를 거치지 않으므로 지연/불안정 요소 제거

## DNS와 dig 명령어

### dig란?
도메인 이름 → IP 주소 변환 과정을 자세히 보여주는 DNS 조회 명령어.

### 사용법
```bash
dig supabase.zep.works
```

### 결과 읽는 법
```
;; QUESTION SECTION:
;supabase.zep.works.  IN  A          ← "이 도메인의 IPv4 주소를 알려줘"

;; ANSWER SECTION:
supabase.zep.works.  300  IN  A  104.21.65.134    ← Cloudflare IP #1
supabase.zep.works.  300  IN  A  172.67.63.161    ← Cloudflare IP #2

;; Query time: 42 msec                ← DNS 응답 시간
;; SERVER: 210.220.163.82             ← 사용 중인 DNS 서버
```

- **A 레코드**: 도메인 → IPv4 주소 매핑
- **300 (TTL)**: 이 결과를 300초(5분) 동안 캐시해도 됨
- **A 레코드 여러 개** = 로드밸런싱 (클라이언트가 랜덤으로 하나 선택)

### IP 대역으로 서비스 구분
| IP 대역 | 서비스 |
|---------|--------|
| `104.x.x.x`, `172.67.x.x` | Cloudflare |
| `13.225.x.x` | AWS CloudFront |

## Node.js ETIMEDOUT 디버깅

### 에러 패턴
```
TypeError: fetch failed
Caused by: AggregateError: (ETIMEDOUT)
    at internalConnectMultiple (node:net:1122:18)
```

### 의미
- `TypeError: fetch failed`: Node.js `fetch()`가 실패했다는 겉포장
- `ETIMEDOUT`: TCP 연결 자체가 타임아웃 (서버에 도달 못 함)
- `internalConnectMultiple`: Node.js Happy Eyeballs 알고리즘 (IPv4/IPv6 동시 시도)

### 디버깅 순서
1. 컨테이너 안에서 `wget`으로 직접 테스트 → 네트워크 자체 문제인지 확인
2. `dig`로 DNS 확인 → IP가 몇 개인지, 어떤 서비스인지
3. `NODE_OPTIONS=--dns-result-order=ipv4first` → IPv6 문제 배제
4. 간헐적이면 → 프록시/로드밸런싱 의심

## 나의 생각
- 같은 서버에 있다고 같은 네트워크는 아니다 (Docker 격리)
- Coolify에서 프로젝트가 다르면 네트워크도 다르다
- 외부 프록시를 거치는 구조는 내부 통신에 불필요한 불안정 요소
- `dig` 명령어는 네트워크 문제 디버깅 시 가장 먼저 확인할 것
