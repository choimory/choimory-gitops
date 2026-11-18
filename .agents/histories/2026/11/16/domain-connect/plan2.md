# 목차

- [개요](#개요)
- [도메인 작업 계획 재정리](#도메인-작업-계획-재정리)
  - [답변](#답변)
  - [확정 사항](#확정-사항)
  - [작업 순서](#작업-순서)
  - [1. 도메인과 Cloudflare DNS](#1-도메인과-cloudflare-dns)
  - [2. Cloudflare Tunnel과 cloudflared](#2-cloudflare-tunnel과-cloudflared)
  - [3. 앱 도메인 확인](#3-앱-도메인-확인)
  - [4. ArgoCD 도메인, Cloudflare Access, 계정](#4-argocd-도메인-cloudflare-access-계정)
  - [5. DB, Redis Tailscale 전용 접속](#5-db-redis-tailscale-전용-접속)
  - [결정 필요 사항](#결정-필요-사항)
- [.dev 부가서비스 이용 불가 안내](#dev-부가서비스-이용-불가-안내)
  - [답변](#답변-1)
  - [안내의 의미](#안내의-의미)
  - [.dev만 안 되는 이유](#dev만-안-되는-이유)
  - [작업과의 관계](#작업과의-관계)
  - [구매 시 확인할 것](#구매-시-확인할-것)
- [도메인 변경 시 포워딩](#도메인-변경-시-포워딩)
  - [답변](#답변-2)
  - [리다이렉트 방법](#리다이렉트-방법)
  - [도메인 변경 시 챙길 것](#도메인-변경-시-챙길-것)
- [Cloudflare Registrar로 이전](#cloudflare-registrar로-이전)
  - [답변](#답변-3)
  - [이전 조건](#이전-조건)
  - [이전 절차](#이전-절차)
  - [비용과 기간](#비용과-기간)
  - [참고](#참고)

---

# 개요

- plan1에서 스택, 접속 방식, 이름 규칙을 정했고 task1에서 Gateway 이름을 `gateway`로 변경함
- 이름 정리로 미뤄 둔 원래 도메인 작업(Tunnel, 앱 도메인, 인프라 도구 도메인, DB 접속)을 다시 정리함
- 참고 문서: [plan1.md](./plan1.md), [task1.md](./task1.md), `.agents/histories/2026/11/06/k8s-setup/plan12.md`

---

# 도메인 작업 계획 재정리

> 이제 원래하려 했던 도메인 작업 등을 plan2에 다시 적어줘

## 답변

- plan1에서 확정한 내용을 기준으로 남은 도메인 작업을 5단계로 다시 정리
- 단계별로 할 일, gitops 변경, 확인 방법, 결정이 필요한 항목을 표시

## 확정 사항

| 항목 | 결정 | 출처 |
|---|---|---|
| 도메인 | 가비아 구매, DNS는 Cloudflare | plan1 추가1 |
| 외부 접속 | Cloudflare Tunnel | plan1 추가1 |
| 인증서 | Cloudflare Universal SSL (Let's Encrypt 불필요) | plan1 추가1 |
| 라우팅 | Gateway API (Envoy Gateway, Gateway `gateway`, HTTPRoute). Ingress 미사용 | plan1, task1 |
| 인프라 도구 접근 제한 | Cloudflare Access | k8s-setup plan12 |
| DB, Redis | Tailscale Operator, tailnet 전용 | plan1 추가1 |
| 이름 | 기본 이름은 유지, 새로 만드는 공용 자원은 `choimory-` 접두어 | plan1 추가2 |

| 대상 | 접속 방식 |
|---|---|
| front, member-api (stg, prod) | Cloudflare Tunnel + 도메인 (공개) |
| ArgoCD 등 인프라 도구 | Cloudflare Tunnel + 도메인 + Cloudflare Access |
| PostgreSQL, Redis | Tailscale 전용 (도메인, 터널 없음) |

## 작업 순서

| 단계 | 작업 | gitops 변경 | 선행 조건 |
|---|---|---|---|
| 1 | 도메인 구매와 Cloudflare DNS 연결 | 없음 | - |
| 2 | Cloudflare Tunnel 생성, cloudflared 배포 | 있음 | 1 |
| 3 | 앱 도메인 4개 접속 확인 | 필요 시 | 2 |
| 4 | ArgoCD 도메인, Cloudflare Access, 계정 설정 | 있음 | 2 |
| 5 | Tailscale Operator로 DB, Redis 접속 | 있음 | 없음 (1~4와 독립) |

- 5단계는 도메인과 무관해 순서를 바꿔 먼저 진행해도 됨

## 1. 도메인과 Cloudflare DNS

1. 가비아에서 `choimory.dev` 구매 (이미 보유 중인지 확인 필요)
2. Cloudflare에 사이트 추가 (무료 플랜) → 네임서버 2개 발급
3. 가비아 도메인 관리에서 네임서버를 Cloudflare 것으로 변경 (DNSSEC가 켜져 있으면 먼저 끄기)
4. Cloudflare에서 Active 확인
5. SSL/TLS 설정 확인: Universal SSL 발급 상태, 암호화 모드

- `.dev`는 HTTPS만 허용되지만 Cloudflare가 처리하므로 문제 없음
- Universal SSL은 `choimory.dev`, `*.choimory.dev`까지만 적용되므로 서브도메인은 한 단계만 사용 (`staging-member-api`)

## 2. Cloudflare Tunnel과 cloudflared

### Tunnel 관리 방식

| 방식 | 라우팅 설정 위치 | 의견 |
|---|---|---|
| 대시보드 관리 (remote-managed, 토큰 방식) | Cloudflare 대시보드 | 추천. cloudflared는 토큰만 있으면 되고, 와일드카드 하나로 Gateway에 넘기면 대시보드 설정이 거의 바뀌지 않음 |
| 설정 파일 관리 (locally-managed) | gitops의 ConfigMap | 라우팅이 Git에 남지만 인증서 파일(credentials) 관리가 추가됨 |

### 연결 구조

```
브라우저 → Cloudflare (Universal SSL) → Tunnel → cloudflared Pod (choimory-infra)
  → Envoy 프록시 Service (gateway) → HTTPRoute 호스트 매칭 → 앱 Service
```

- Tunnel의 Public Hostname은 `choimory.dev`와 `*.choimory.dev` 두 개를 모두 Envoy 프록시 Service로 연결
  - `*.choimory.dev`는 `choimory.dev`에 매칭되지 않아 둘 다 필요 (Gateway listener에 hostname을 두지 않은 것과 같은 이유)
  - 와일드카드는 DNS 레코드(`*` CNAME → Tunnel)를 직접 추가해야 할 수 있음. 작업 시 확인
- 도메인을 추가해도 HTTPRoute만 만들면 되고 Cloudflare 설정은 바뀌지 않음
- cloudflared는 원래 Host 헤더를 그대로 전달하므로 HTTPRoute의 hostnames 매칭이 그대로 동작

### Envoy 프록시 Service 이름

- 자동 생성 이름은 `envoy-choimory-infra-gateway-<해시>`라 Tunnel 대상 주소에 해시가 들어감
- EnvoyProxy의 `envoyService`에 고정 이름을 지정할 수 있는지 확인 후, 가능하면 고정 이름으로 지정해 Tunnel 대상 주소를 안정적으로 유지
- 고정 이름을 쓰지 않으면 Gateway를 다시 만들 때마다 Tunnel 대상 주소도 바꿔야 함

### gitops 변경 (예정)

| 파일 | 내용 |
|---|---|
| `infrastructure/apps/cloudflared.yaml` | Application `choimory-cloudflared` (sync wave는 gateway 다음) |
| `infrastructure/cloudflared/deployment.yaml` | Deployment `choimory-cloudflared`, `cloudflared tunnel run --token` |
| `infrastructure/cloudflared/choimory-cloudflared-token.yaml` | Tunnel 토큰 SealedSecret |
| `infrastructure/gateway/gateway.yaml` | (확인 후) EnvoyProxy에 Service 고정 이름 |
| `README.md` | 구성 표, 디렉토리 구조, 도메인 항목 |

- Tunnel 이름은 대시보드에서 `choimory-tunnel`
- 레플리카 1개, 메모리는 요청 32Mi 수준에서 시작
- 토큰은 서버에서 `kubeseal --controller-namespace choimory-infra`로 암호화하고 원문은 저장소, 문서, 채팅에 남기지 않음

## 3. 앱 도메인 확인

| 앱 | stg | prod |
|---|---|---|
| front | `staging.choimory.dev` | `choimory.dev` |
| member-api | `staging-member-api.choimory.dev` | `member-api.choimory.dev` |

- 브라우저에서 front 화면, 로그인과 회원가입 흐름 확인
- member-api 직접 호출 확인 (예: `/v1/member/...`)
- k8s-setup 1차 확인에서 미뤄 둔 브라우저 흐름과 CORS를 여기서 확인
- 문제 시 확인 위치: Cloudflare Tunnel 상태, cloudflared 로그, HTTPRoute `Accepted`, Envoy 프록시 로그

## 4. ArgoCD 도메인, Cloudflare Access, 계정

### 도메인과 HTTPRoute

- 도메인 예: `argocd.choimory.dev` (결정 필요)
- `infrastructure/argocd/`에 HTTPRoute 추가: `argocd-server` Service의 HTTP 포트로 전달
- 확인할 점
  - Gateway는 `gateway-access: "true"` 라벨이 있는 네임스페이스의 HTTPRoute만 허용. `choimory-infra` 네임스페이스(`infrastructure/apps/namespace.yaml`)에 라벨 추가 필요
  - ArgoCD 서버는 기본으로 자체 TLS를 사용하므로 values에 `configs.params."server.insecure": true` 추가 (TLS는 Cloudflare가 처리)
  - ArgoCD UI의 외부 주소 설정(`configs.cm.url`)도 도메인으로 지정

### Cloudflare Access

- Zero Trust에서 Self-hosted Application으로 인프라 도구 도메인 등록
- 인프라 도구 공통 정책 하나를 만들고, 이후 Grafana 등은 도메인만 추가
- 로그인 수단: 이메일 OTP 또는 GitHub 로그인 (결정 필요)
- 무료 플랜 50명 한도 내

### 계정 설정 (k8s-setup plan10에서 정한 순서)

1. Cloudflare Access 적용 확인 (Access 없이 ArgoCD 로그인 화면이 열리면 안 됨)
2. `argocd-initial-admin-secret`의 초기 비밀번호로 로그인
3. 비밀번호 변경
4. `argocd-initial-admin-secret` 삭제

## 5. DB, Redis Tailscale 전용 접속

### 구성

- Tailscale Operator를 Helm 차트로 설치 (Application 이름은 차트 기본 이름, 네임스페이스는 `choimory-infra`)
- Tailscale 관리 화면에서 OAuth 클라이언트 발급 → SealedSecret으로 저장
- DB, Redis Service에 Tailscale 노출 설정을 붙이면 DB마다 tailnet 기기가 생김

| 대상 | tailnet 이름 (예) | 포트 |
|---|---|---|
| stg PostgreSQL | `choimory-dev-stg-postgres` | 5432 |
| prod PostgreSQL | `choimory-dev-prod-postgres` | 5432 |
| stg Redis | `choimory-dev-stg-redis` | 6379 |
| prod Redis | `choimory-dev-prod-redis` | 6379 |

### 확인할 점

- PostgreSQL Service는 CloudNativePG가 만들므로, Cluster의 추가 Service 설정을 쓸지 별도 Service를 만들지 결정
- Tailscale 프록시 Pod는 `choimory-infra`에서 뜨므로 프로젝트 NetworkPolicy(`choimory-infra` 허용)에 막히지 않음
- tailnet ACL로 접근 기기를 제한할지 결정
- prod는 DB 도구에서 Read-only 연결 추천
- 적용 후 task9에서 쓰던 port-forward, SSH 터널 방식은 불필요

## 결정 필요 사항

1. 도메인 `choimory.dev`를 이미 보유 중인지, 지금 구매해야 하는지
2. Tunnel 관리 방식: 대시보드 관리(추천) / 설정 파일 관리
3. ArgoCD 도메인 이름 (예: `argocd.choimory.dev`)
4. Cloudflare Access 로그인 수단: 이메일 OTP / GitHub
5. 진행 순서: 1~4단계(도메인) 먼저 / 5단계(DB 접속) 먼저

---

# .dev 부가서비스 이용 불가 안내

> dev 도메인 구매중인데 
>
> * 해당 도메인들은 부가서비스(포워딩, 파킹, 도메인 연결) 이용이 불가합니다.
>
> 뭐지

## 답변

- 구매해도 됨. 작업에 영향 없음

## 안내의 의미

| 부가서비스 | 하는 일 |
|---|---|
| 포워딩 | `choimory.dev`로 들어오면 다른 주소로 넘겨 줌 |
| 파킹 | 안 쓰는 도메인에 가비아 안내 페이지를 띄움 |
| 도메인 연결 | 가비아 웹호스팅, 블로그 등에 클릭으로 연결 |

- 세 가지 모두 가비아 서버가 대신 응답하는 방식

## .dev만 안 되는 이유

- `.dev`는 브라우저가 HTTPS로만 접속하도록 강제되는 도메인 (HSTS preload)
- 가비아의 포워딩, 파킹 서버에는 해당 도메인 인증서가 없어 브라우저가 접속을 막음. 그래서 처음부터 막아 둠
- `.app`, `.page` 등 Google 계열 도메인도 같은 이유

## 작업과의 관계

- 가비아 기능은 쓰지 않고 네임서버만 Cloudflare로 변경
- 네임서버 변경은 부가서비스가 아닌 기본 기능이라 `.dev`도 가능
- DNS, HTTPS(Universal SSL), 접속(Tunnel)은 모두 Cloudflare가 처리

## 구매 시 확인할 것

- 도메인 정보 보호(WHOIS 개인정보 비공개): 무료면 켜기 추천
- 자동 연장: 만료 시 사이트가 끊기므로 켜기 추천. 갱신 가격 확인
- DNSSEC: 켜져 있으면 네임서버 변경 전에 끄기

---

# 도메인 변경 시 포워딩

> 내가 나중에 도메인 변경할때 포워딩이 필요하지 않을까

## 답변

- 필요해질 수 있지만 가비아 포워딩이 아니라 Cloudflare에서 처리. `.dev`도 문제없음
- 네임서버를 Cloudflare로 옮기면 가비아 포워딩은 어차피 동작하지 않음 (가비아 네임서버를 쓸 때만 되는 기능)
- Cloudflare는 `choimory.dev` 인증서(Universal SSL)가 있어 HTTPS 요청을 받아 새 도메인으로 넘길 수 있음. 가비아가 막은 원인(인증서 없음)이 없음

## 리다이렉트 방법

| 방법 | 위치 | 특징 |
|---|---|---|
| Cloudflare Redirect Rules (추천) | Cloudflare 대시보드 → Rules | 무료. 클러스터까지 가지 않고 Cloudflare에서 바로 301. 경로, 쿼리 유지 가능 (`choimory.dev/abc` → `새도메인/abc`) |
| HTTPRoute RequestRedirect | gitops (Gateway API) | 리다이렉트도 Git으로 관리. 요청이 Tunnel, Envoy까지 들어옴 |

## 도메인 변경 시 챙길 것

- 옛 도메인 유지: 리다이렉트는 옛 도메인이 살아 있어야 동작. 북마크, 검색 결과가 정리될 때까지 최소 1년 정도 연장 권장
- 301 사용: 영구 이동이어야 검색엔진이 새 도메인으로 옮겨 감
- 서브도메인: `staging.choimory.dev`, `member-api.choimory.dev` 등도 같은 규칙에 포함 가능
- 앱 설정: API 주소(`NEXT_PUBLIC_MEMBER_API_BASE_URL`), CORS, 쿠키 도메인은 리다이렉트로 해결되지 않아 새 도메인으로 재빌드, 재배포 필요

---

# Cloudflare Registrar로 이전

> 가비아에서 1년쓰다가 cloudflare registar로 갈아타는거도 가능?

## 답변

- 가능. DNS를 이미 Cloudflare로 옮겨 둔 상태라 이전 중에도 접속이 끊기지 않음

## 이전 조건

| 조건 | 내용 |
|---|---|
| 등록 후 60일 경과 | 새로 등록한 도메인은 60일간 기관 이전이 잠김 (ICANN 규정). 1년 사용 후 이전이면 상관없음 |
| Cloudflare 네임서버 사용 중 | Cloudflare에서 Active 상태인 도메인만 이전 가능. 1단계에서 맞춰 둠 |
| 만료 직전이 아닐 것 | 만료일이 너무 가까우면 거부되거나 꼬일 수 있음. 만료 1~2달 전 이전 추천 |
| 이전 잠금 해제 | 가비아에서 도메인 이전 잠금(Transfer Lock) 해제 |
| 인증코드(EPP/Auth Code) | 가비아에서 발급받아 Cloudflare에 입력 |

## 이전 절차

1. 가비아: 이전 잠금 해제, 인증코드 발급
2. Cloudflare 대시보드 → Domain Registration → Transfer Domains에서 도메인 선택, 인증코드 입력, 결제
3. 등록자 이메일로 오는 승인 메일 확인 (가비아 쪽 승인 절차가 따로 있을 수 있음)
4. 며칠 안에 완료. 그동안 DNS는 계속 Cloudflare라 사이트 정상 동작

## 비용과 기간

- 이전 시 1년 연장 비용을 Cloudflare 원가로 결제
- 남은 기간은 유지되고 1년이 더해짐 (예: 만료 2달 전 이전 → 원래 만료일에서 1년 뒤로 연장)
- 이전 직후 다시 60일간 이전 잠금

## 참고

- `.dev`는 Cloudflare Registrar 지원 확장자
- 처음부터 Cloudflare에서 사면 이전 절차가 필요 없음. 가비아를 고른 이유가 첫해 가격이나 결제 편의라면 두 곳의 첫해, 갱신 가격 비교 후 결정
- 가비아에서 사서 1년 뒤 이전하는 방식도 문제없음. 만료 1~2달 전 이전하도록 메모
