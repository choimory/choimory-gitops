# 목차

- [개요](#개요)
- [4단계 (E2) 작업 계획](#4단계-e2-작업-계획)
  - [답변](#답변)
  - [사전 확인](#사전-확인)
  - [제안 값](#제안-값)
  - [작업 단계](#작업-단계)
    - [A. 사전 준비](#a-사전-준비)
    - [B. gitops 1차 작성](#b-gitops-1차-작성)
    - [C. 확인](#c-확인)
    - [D. DNS](#d-dns)
    - [E. gitops 2차 작성](#e-gitops-2차-작성)
    - [F. 확인과 계정 정리](#f-확인과-계정-정리)
  - [1차, 2차로 나누는 이유](#1차-2차로-나누는-이유)
  - [주의할 점](#주의할-점)
- [내부 도메인, 내부 Gateway, 인증서, DB 단계 확인](#내부-도메인-내부-gateway-인증서-db-단계-확인)
  - [답변](#답변-1)
  - [1. 내부 도메인 사용 주체](#1-내부-도메인-사용-주체)
  - [2. 내부 Gateway의 역할](#2-내부-gateway의-역할)
    - [따로 두는 이유](#따로-두는-이유)
    - [이름](#이름)
  - [3. 인증서](#3-인증서)
  - [4. DB 진행 시점](#4-db-진행-시점)
  - [남은 결정 사항](#남은-결정-사항)
- [Universal SSL 대신 와일드카드 인증서](#universal-ssl-대신-와일드카드-인증서)
  - [답변](#답변-2)
  - [Universal SSL을 쓸 수 없는 이유](#universal-ssl을-쓸-수-없는-이유)
  - [Let's Encrypt 와일드카드](#lets-encrypt-와일드카드)
  - [Cloudflare Origin 인증서](#cloudflare-origin-인증서)
- [Cloudflare에 Let's Encrypt 적용](#cloudflare에-lets-encrypt-적용)
  - [답변](#답변-3)
  - [1. 엣지 인증서 CA](#1-엣지-인증서-ca)
  - [2. 직접 발급 인증서 업로드](#2-직접-발급-인증서-업로드)
  - [3. origin 구간 Let's Encrypt](#3-origin-구간-lets-encrypt)
  - [정리](#정리)
- [추가1](#추가1)
- [추가1 분석](#추가1-분석)
  - [답변](#답변-4)
  - [1. 게이트웨이 이름](#1-게이트웨이-이름)
  - [2. 도메인 규칙](#2-도메인-규칙)
    - [DNS](#dns)
    - [인증서](#인증서)
    - [도구 추가 시 할 일](#도구-추가-시-할-일)
  - [3. 확인할 점](#3-확인할-점)
  - [정리 (추가1)](#정리-추가1)
- [추가2](#추가2)
- [추가2 분석](#추가2-분석)
  - [답변](#답변-5)
  - [1. 도메인](#1-도메인)
  - [2. 프로필 표기](#2-프로필-표기)
  - [3. 공용 도구와 프로젝트별 도구](#3-공용-도구와-프로젝트별-도구)
  - [확정 제안 값](#확정-제안-값)
- [4단계 최종 작업계획](#4단계-최종-작업계획)
  - [답변](#답변-6)
  - [확정 값](#확정-값)
  - [작업 단계 (최종)](#작업-단계-최종)
    - [A. 사전 준비 (최종)](#a-사전-준비-최종)
    - [B. gitops 1차 (최종)](#b-gitops-1차-최종)
    - [C. 확인 (최종)](#c-확인-최종)
    - [D. DNS (최종)](#d-dns-최종)
    - [E. gitops 2차 (최종)](#e-gitops-2차-최종)
    - [F. 확인과 계정 정리 (최종)](#f-확인과-계정-정리-최종)
  - [1차, 2차 push 구분](#1차-2차-push-구분)
  - [주의할 점과 문제 생길 때 볼 곳](#주의할-점과-문제-생길-때-볼-곳)
  - [기록](#기록)

---

# 개요

- plan4에서 인프라 도구 접근 방식을 E2(Tailscale 전용 내부 Gateway + 내 도메인)로 결정
- 이 문서는 4단계(ArgoCD 내부 접속, 계정 정리)를 E2 기준으로 구체화한 작업 계획
- 참고 문서: [plan4.md](./plan4.md)(E2 결정과 4단계 계획, E2에서 인증서가 필요한 이유)

---

# 4단계 (E2) 작업 계획

> 이제 작업계획 이야기해봐

## 답변

- 결정이 필요한 항목은 제안 값을 넣어 정리. 그대로 갈지, 바꿀지만 정하면 됨

## 사전 확인

- Envoy Gateway v1.10 EnvoyProxy가 `envoyService.loadBalancerClass`, `annotations`, `name` 지원
- ArgoCD 차트 `10.17.0`이 `server.httproute` 지원 → ArgoCD HTTPRoute는 차트 values로 생성

## 제안 값

| 항목 | 제안 |
|---|---|
| 내부 도메인 | `internal.choimory.dev` (ArgoCD: `argocd.internal.choimory.dev`) |
| 내부 Gateway | Gateway `internal-gateway`, GatewayClass `envoy-internal`, EnvoyProxy `envoy-proxy-internal`, 프록시 Deployment, Service `internal-gateway-proxy` |
| tailnet 기기 이름 | 내부 Gateway `choimory-internal-gateway`, Operator `choimory-k8s-operator` |
| 허용 범위 라벨 | `internal-gateway-access: "true"` (`choimory-infra`에 우선 적용) |
| 인증서 | ClusterIssuer `letsencrypt`, Certificate `internal-choimory-dev` (Secret `internal-choimory-dev-tls`) |
| DB 노출 | 이번에는 Operator 설치까지, DB는 바로 이어서 5단계 |

## 작업 단계

### A. 사전 준비

| 순서 | 위치 | 할 일 |
|---|---|---|
| A-1 | Cloudflare → My Profile → API Tokens | 토큰 생성. 권한 Zone·DNS·Edit, Zone·Zone·Read, 대상 영역 `choimory.dev`만 |
| A-2 | Tailscale 관리 화면 → Access controls | `tagOwners`에 `tag:k8s-operator`, `tag:k8s` 추가 |
| A-3 | Tailscale 관리 화면 → Settings → OAuth clients | OAuth 클라이언트 발급 (Devices Core, Auth Keys 쓰기, 태그 `tag:k8s-operator`) |
| A-4 | 서버 | SealedSecret 2개 생성: `cloudflare-api-token`, `operator-oauth` |

- 사용자 진행, 화면 안내와 명령은 Claude 제공

### B. gitops 1차 작성

| 파일 | 내용 |
|---|---|
| `apps/cert-manager.yaml` + `cert-manager/values.yaml` | cert-manager 차트, CRD 포함 설치, wave 5 |
| `apps/tailscale-operator.yaml` + `tailscale-operator/` | Operator 차트, values, OAuth SealedSecret, wave 5 |
| `apps/internal-gateway.yaml` + `internal-gateway/` | EnvoyProxy(LoadBalancer, `loadBalancerClass: tailscale`, 고정 이름), GatewayClass, Gateway(HTTPS 443, `*.internal.choimory.dev`, 인증서 참조), ClusterIssuer(Cloudflare DNS-01), Certificate, 토큰 SealedSecret, wave 6 |
| `apps/namespace.yaml` | `choimory-infra`에 `internal-gateway-access: "true"` 라벨 |
| `README.md` | 구성, 디렉토리, sync wave 표, 인프라 도구 접속 방식 |

- Claude 작성, 사용자 push

### C. 확인

- cert-manager, Operator Pod Running
- Certificate `Ready True` (보통 1~2분)
- `internal-gateway-proxy` Service에 `100.x` IP, Tailscale 관리 화면에 기기 생성

### D. DNS

- Cloudflare DNS에 `*.internal` A 레코드 → `100.x`. 반드시 회색 구름(DNS only)

### E. gitops 2차 작성

| 파일 | 내용 |
|---|---|
| `argocd/values.yaml` | `configs.params."server.insecure": true`, `configs.cm.url: https://argocd.internal.choimory.dev`, `server.httproute`(parentRef `internal-gateway`, 호스트 `argocd.internal.choimory.dev`) |

### F. 확인과 계정 정리

1. Tailscale 켠 기기에서 `https://argocd.internal.choimory.dev` 접속 → 인증서 자물쇠, ArgoCD 로그인 화면 확인
2. Tailscale 끄고 접속 안 되는지 확인
3. 초기 비밀번호 로그인 → 비밀번호 변경 → `argocd-initial-admin-secret` 삭제

## 1차, 2차로 나누는 이유

- 인증서와 tailnet IP가 준비되기 전에 ArgoCD 설정(`server.insecure`)을 바꾸면 접속 경로가 꼬일 수 있음
- 내부 Gateway를 먼저 완성해 확인한 뒤 ArgoCD 연결
- 지금 ArgoCD 접속 방식(서버 port-forward)은 2차 push 전까지 그대로 사용 가능

## 주의할 점

- Let's Encrypt는 처음부터 실제 발급(prod). 발급 실패 시 반복 재시도하지 말고 원인 확인 (발급 한도)
- Cloudflare API 토큰, Tailscale OAuth secret 원문은 채팅, 문서에 남기지 않고 서버에서 바로 암호화
- calendar는 TASK 유지, 작업 기록은 task4
- 결정 필요: 제안 값 확정, A-1부터 시작 여부

---

# 내부 도메인, 내부 Gateway, 인증서, DB 단계 확인

> 1. 내부도메인은 누가 가지는거임? argocd?
> 2. 내부 gateway는 내부용으로 하나의 gateway를 또 두는거야? tailscale용? 아니면 infra용?
> 3. 인증서는 letsencrypt 할거임
> 4. operator 설치랑 argocd 적용후에 db는 5단계에서 한다는건가?

## 답변

- 내부 도메인은 ArgoCD 전용이 아니라 내부 도구들이 함께 쓰는 영역
- 내부 Gateway는 관리 도구 전용 입구로 하나 더 두며, 들어오는 길이 Tailscale
- 인증서는 Let's Encrypt 확정
- DB는 4단계에서 설치한 Operator를 재사용해 5단계에서 진행

## 1. 내부 도메인 사용 주체

- `internal.choimory.dev`는 내부 도구 공용 영역. 도구마다 하위 도메인을 하나씩 붙임
  - 지금: `argocd.internal.choimory.dev`
  - 나중: `grafana.internal.choimory.dev` 등 HTTPRoute만 추가
- 와일드카드 인증서 `*.internal.choimory.dev` 하나, DNS 레코드 하나로 모든 내부 도구 처리 → 도구 추가 시 인증서, DNS 작업 없음
- 도메인을 받는 곳은 내부 Gateway (리스너 호스트 `*.internal.choimory.dev`)
- DB, Redis는 HTTP가 아닌 TCP라 이 도메인을 쓰지 않음 (5단계 방식으로 접속)

## 2. 내부 Gateway의 역할

- Gateway를 하나 더 둠. 용도는 인프라 관리 도구 입구, 들어오는 길은 Tailscale

| | 공개 Gateway (`gateway`) | 내부 Gateway (`internal-gateway`) |
|---|---|---|
| 용도 | 서비스 앱 (front, member-api) | 관리 도구 (ArgoCD, 나중에 Grafana 등) |
| 들어오는 길 | 인터넷 → Cloudflare → Tunnel → `gateway-proxy` | 내 기기(Tailscale) → `internal-gateway-proxy`(tailnet IP) |
| 도메인 | `*.choimory.dev` | `*.internal.choimory.dev` |
| 인증서 | Cloudflare가 처리 | Let's Encrypt (cert-manager) |
| 위치 | `choimory-infra` | `choimory-infra` (공용 인프라) |

### 따로 두는 이유

- ArgoCD를 공개 Gateway에 붙이면 Tunnel을 통해 인터넷에 그대로 노출됨
- Gateway를 나누면 어느 Gateway에 붙였는지만으로 공개 여부가 정해져 실수 여지가 줄어듦

### 이름

- 용도를 드러내려면 `infra-gateway`도 가능
- 도메인(`internal.`)과 이름을 맞추는 쪽이 덜 헷갈려 `internal-gateway` 추천

## 3. 인증서

- Let's Encrypt 확정. ClusterIssuer 이름 `letsencrypt`
- 내부 도메인은 인터넷에서 접근할 수 없어 HTTP-01 불가. 와일드카드도 DNS-01 필수
- cert-manager가 Cloudflare API 토큰으로 DNS에 TXT 레코드를 만드는 DNS-01 방식으로 발급 (A-1 토큰이 필요한 이유)

## 4. DB 진행 시점

1. 4단계: cert-manager → Tailscale Operator → 내부 Gateway → 인증서, DNS 확인 → ArgoCD 연결 → 계정 정리
   - Operator는 내부 Gateway를 tailnet에 노출하는 데 필요해 4단계에서 설치
2. 5단계: 설치된 Operator로 PostgreSQL, Redis를 tailnet에 노출
   - DB는 내부 Gateway를 거치지 않음
   - 각 DB에 Tailscale 전용 Service를 붙여 바로 접속 (예: `choimory-dev-stg-postgres:5432`)
   - 새로 설치할 것이 없어 4단계 직후 바로 이어서 가능
- 4단계 확인 후 넘어가려고 나눈 것이며, 합쳐서 진행해도 됨

## 남은 결정 사항

1. 내부 Gateway 이름: `internal-gateway`(추천) / `infra-gateway`
2. 5단계: 4단계 확인 후 바로 이어서 / 4단계와 합쳐서

---

# Universal SSL 대신 와일드카드 인증서

> 아 universal ssl이 아니라 와일드카드 인증서가 가능한거야?

## 답변

- 가능함. 내부 도메인은 Universal SSL로 대신할 수 없어 Let's Encrypt 와일드카드 인증서를 따로 받음

## Universal SSL을 쓸 수 없는 이유

1. Universal SSL은 Cloudflare를 거치는 요청에만 적용
   - Cloudflare 엣지가 브라우저에 보여주는 인증서. 주황 구름(Proxied) 레코드만 해당
   - 내부 도메인은 회색 구름(DNS only). Cloudflare는 `100.x` 주소만 알려주고, 실제 연결은 내 기기 → Tailscale → `internal-gateway`로 바로 감
   - Cloudflare를 지나가지 않으므로 `internal-gateway`가 직접 인증서를 내밀어야 함
2. Universal SSL은 한 단계 하위 도메인까지만 덮음
   - 범위: `choimory.dev`, `*.choimory.dev`
   - `argocd.internal.choimory.dev`(두 단계)는 범위 밖. 덮으려면 유료 기능(Advanced Certificate Manager) 필요

| | 공개 앱 (`staging.choimory.dev`) | 내부 도구 (`argocd.internal.choimory.dev`) |
|---|---|---|
| 요청 경로 | 브라우저 → Cloudflare → Tunnel → 클러스터 | 내 기기 → Tailscale → `internal-gateway` |
| 인증서를 내미는 곳 | Cloudflare 엣지 | `internal-gateway` |
| 인증서 | Universal SSL (자동, 무료) | Let's Encrypt 와일드카드 (cert-manager) |

## Let's Encrypt 와일드카드

- 무료, 브라우저가 신뢰함
- 와일드카드는 DNS-01 인증으로만 발급 → cert-manager에 Cloudflare API 토큰 제공
- 유효기간 90일, cert-manager가 만료 전 자동 갱신
- `*.internal.choimory.dev`는 한 단계만 덮음
  - `argocd.internal.choimory.dev`, `grafana.internal.choimory.dev` → 가능
  - `a.b.internal.choimory.dev` → 불가 (쓸 일 없음)

## Cloudflare Origin 인증서

- 무료지만 Cloudflare만 신뢰하는 인증서
- 브라우저가 직접 접속하면 경고가 떠서 내부 도메인에는 사용 불가

---

# Cloudflare에 Let's Encrypt 적용

> cloudflare에도 lets encrypt를 쓰는건 안됨?

## 답변

- 세 가지로 해석할 수 있으며, 결론은 지금 구성 유지 추천

## 1. 엣지 인증서 CA

- Universal SSL은 Cloudflare가 Let's Encrypt, Google Trust Services 같은 CA에서 대신 받아오는 인증서
- 지금 엣지 인증서가 이미 Let's Encrypt일 수 있음. 대시보드 SSL/TLS → Edge Certificates에서 발급 CA 확인 가능
- 무료 플랜에서 CA를 고를 수 있는지는 확인하지 않음. 바꾸려면 대시보드에서 먼저 확인 필요
- 어느 CA든 무료, 자동 갱신, 브라우저 신뢰는 같아 바꿀 이유가 거의 없음

## 2. 직접 발급 인증서 업로드

- 무료 플랜 불가. 직접 받은 인증서 업로드(Custom Certificate)는 Business 플랜 이상
- cert-manager로 받은 인증서 하나를 공개, 내부 양쪽에 같이 쓰는 방식은 불가

## 3. origin 구간 Let's Encrypt

- 기술적으로 가능 (`gateway`에 HTTPS 리스너와 인증서, cloudflared가 HTTPS로 연결)
- Tunnel 구조에서는 불필요

| 구간 | 지금 상태 |
|---|---|
| 브라우저 → Cloudflare | Universal SSL (HTTPS) |
| Cloudflare → cloudflared | Tunnel 자체가 암호화 |
| cloudflared → `gateway-proxy` | 클러스터 내부 HTTP |

- 마지막 구간만 평문이지만 같은 서버 안 클러스터 내부 통신이라 토이 프로젝트에서는 충분
- 정상 동작하는 2단계를 다시 건드리는 위험만 생김

## 정리

- 공개 쪽: Cloudflare가 처리 (내부적으로 Let's Encrypt 같은 CA 사용). 그대로 유지
- 내부 쪽: Cloudflare를 거치지 않아 cert-manager로 Let's Encrypt 와일드카드 직접 발급
- 둘 다 무료, 자동 갱신

---

# 추가1 

- 도메인은 {profile}.{infra-name}.choimory.dev
  - 예: argocd.choimory.dev, stg.prometheus.choimory.dev
  - 운영은 profile 없이 도구명만, 개발과 스테이징은 프로필까지 
- 게이트웨이는 infra-gateway
  - 내부망을 인프라도구들에 입힐것이기 때문에, infra-gateway

---

# 추가1 분석

> - 도메인은 {profile}.{infra-name}.choimory.dev
>   - 예: argocd.choimory.dev, stg.prometheus.choimory.dev
>   - 운영은 profile 없이 도구명만, 개발과 스테이징은 프로필까지 
> - 게이트웨이는 infra-gateway
>   - 내부망을 인프라도구들에 입힐것이기 때문에, infra-gateway

## 답변

- 둘 다 가능
- 단, 도메인 규칙을 바꾸면 E2를 고를 때 기대한 "도구 추가는 HTTPRoute 하나로 끝"이 성립하지 않음

## 1. 게이트웨이 이름

- `infra-gateway` 문제없음. 관련 이름은 아래처럼 맞춤

| 항목 | 이름 |
|---|---|
| Gateway | `infra-gateway` |
| 프록시 Service, Deployment | `infra-gateway-proxy` (공개 쪽 `gateway-proxy`와 같은 규칙) |
| GatewayClass, EnvoyProxy | `envoy-infra`, `envoy-proxy-infra` |
| 허용 라벨 | `infra-gateway-access: "true"` |
| tailnet 기기 이름 | `choimory-infra-gateway` |

## 2. 도메인 규칙

- `{profile}.{infra-name}.choimory.dev`도 동작함
- 공개 앱과 같은 층(`*.choimory.dev`)을 함께 쓰게 되어 몇 가지가 달라짐

### DNS

- 지금 `*.choimory.dev`는 와일드카드로 Tunnel에 연결됨
- `argocd.choimory.dev`처럼 이름을 명시한 레코드가 와일드카드보다 우선하므로 도구마다 레코드를 직접 만들면 됨
- 와일드카드 레코드 하나로 끝나지 않고 도구를 추가할 때마다 DNS 레코드가 생김
  - `prometheus.choimory.dev` → `100.x`
  - `*.prometheus.choimory.dev` → `100.x` (stg, dev용)
- 레코드를 빠뜨려도 공개 Gateway에는 해당 라우트가 없어 404만 남. 노출 위험 없음

### 인증서

- `*.internal.choimory.dev` 하나로 처리 불가. 도구마다 인증서 이름 추가 필요
  - `argocd.choimory.dev`
  - `prometheus.choimory.dev`, `*.prometheus.choimory.dev`
- `*.choimory.dev` 와일드카드 하나로 처리하는 방법은 비추천. 공개 도메인 전체에 쓸 수 있는 키가 클러스터에 저장됨
- 필요한 이름만 나열하는 방식 추천

### 도구 추가 시 할 일

| | 기존안 (`*.internal.choimory.dev`) | 추가1 (`{profile}.{tool}.choimory.dev`) |
|---|---|---|
| HTTPRoute | 1개 | 1개 |
| DNS 레코드 | 없음 | 1~2개 |
| 인증서 | 없음 | 이름 추가 (재발급은 자동) |

## 3. 확인할 점

1. 이름 표기 통일
   - 공개 앱은 `staging.choimory.dev`, `staging-member-api.choimory.dev`처럼 `staging`을 접두어로 쓰고 하이픈으로 붙임
   - 추가1 예시는 `stg.`에 점 구분
   - 인프라와 앱의 규칙이 달라도 되는지, 맞출지 결정 필요
2. 프로젝트 구분
   - 앞선 고민은 "다른 프로젝트의 grafana가 추가되면 두 개가 됨"이었음
   - `{profile}.{tool}` 규칙에는 프로젝트 자리가 없음
   - 도구를 프로젝트마다 둘지, 서버 전체에 하나만 둘지에 따라 규칙이 달라짐
   - 예: `stg.grafana.choimory.dev`가 어느 프로젝트 것인지 이름만으로 알 수 없음

## 정리 (추가1)

- 짧은 주소를 원하면 추가1 방식도 충분히 운영 가능. 대신 도구 추가 시 DNS와 인증서 이름도 같이 추가
- 관리 비용을 최소로 하려면 기존안의 와일드카드 방식(`*.internal.choimory.dev`)이 나음
  - 예: `argocd.internal.choimory.dev`, `stg-prometheus.internal.choimory.dev`
  - 하이픈 규칙도 앱과 통일됨
- 결정 필요: 도메인 방식, `stg`와 `staging` 표기, 프로젝트 구분

---

# 추가2

- staging이 맞음 내가 대충쓴거
- infra.choimory.dev로 가자
  - argocd.choimory.dev, staging-argocd.choimory.dev..
- 인프라도구는 전체에 놓는것도 있고 프로젝트별로 두게 될수도 있음

---

# 추가2 분석

> - staging이 맞음 내가 대충쓴거
> - infra.choimory.dev로 가자
>   - argocd.choimory.dev, staging-argocd.choimory.dev..
> - 인프라도구는 전체에 놓는것도 있고 프로젝트별로 두게 될수도 있음

## 답변

- 도메인 `*.infra.choimory.dev`, 프로필 `staging-` 접두어로 정리
- 와일드카드 방식이라 공용, 프로젝트별 도구 규칙은 나중에 정해도 됨

## 1. 도메인

- 예시의 `argocd.choimory.dev`, `staging-argocd.choimory.dev`는 `infra.`가 빠진 것으로 해석
  - 의도: `argocd.infra.choimory.dev`, `staging-xxx.infra.choimory.dev` (해석 확인 필요)
- 기존 와일드카드 방식과 같아 관리 부담 없음
  - 인증서: `*.infra.choimory.dev` 하나
  - DNS: `*.infra.choimory.dev` → `100.x` 레코드 하나
  - 도구 추가: HTTPRoute 하나
- `infra-gateway`와 `infra.choimory.dev`로 이름도 맞춰짐

## 2. 프로필 표기

- `staging-` 접두어. 공개 앱(`staging-member-api.choimory.dev`)과 같은 규칙
- 하이픈으로 붙여 한 단계 이름이 유지되므로 와일드카드 인증서 하나로 덮임
- 서버 전체에 하나만 있는 도구에는 프로필이 붙지 않음
  - ArgoCD는 클러스터당 하나라 `staging-argocd`는 생기지 않고 `argocd.infra.choimory.dev`만 존재
  - 프로필은 환경별로 따로 띄우는 도구에만 붙음

## 3. 공용 도구와 프로젝트별 도구

- 지금 정하지 않아도 됨. `*.infra.choimory.dev` 아래 한 단계 이름이면 인프라 변경 없이 추가 가능
- 이번 작업에서 실제로 만드는 것은 공용 도구인 ArgoCD 하나
- 나중을 위한 규칙 예시

| 구분 | 규칙 | 예 |
|---|---|---|
| 공용 (서버 전체) | `{tool}.infra.choimory.dev` | `argocd`, `grafana` |
| 프로젝트별 | `[staging-]{project}-{tool}.infra.choimory.dev` | `choimory-dev-grafana`, `staging-choimory-dev-grafana` |

- 다른 프로젝트가 자기 도메인을 따로 가지면 그 도메인 아래 `infra.`를 두는 방법도 있음. 그때 결정

## 확정 제안 값

- plan5 [제안 값](#제안-값) 갱신

| 항목 | 값 |
|---|---|
| 내부 도메인 | `infra.choimory.dev` (ArgoCD: `argocd.infra.choimory.dev`) |
| 인증서 | ClusterIssuer `letsencrypt`, Certificate `infra-choimory-dev` (`*.infra.choimory.dev`) |
| DNS | `*.infra.choimory.dev` A → `100.x` (DNS only) |
| Gateway | `infra-gateway` / 프록시 `infra-gateway-proxy` / GatewayClass `envoy-infra` / EnvoyProxy `envoy-proxy-infra` |
| 허용 라벨 | `infra-gateway-access: "true"` |
| tailnet 기기 | Gateway `choimory-infra-gateway`, Operator `choimory-k8s-operator` |

- 결정 필요: 도메인 해석 확인, 5단계(DB)를 4단계 확인 후 이어서 할지 합칠지, A-1 시작 여부

---

# 4단계 최종 작업계획

> 도메인해석 맞고 5단계는 4단계 한 다음에 한번 끊고 갈거임 이제 4단계 작업 시작전 최종 작업계획 얘기해줘

## 답변

- 도메인 해석(`argocd.infra.choimory.dev`) 확정, 5단계(DB)는 4단계 완료 후 끊고 따로 진행
- 확정 값과 저장소 현재 구성(EnvoyProxy, Application 형식, sync wave, ArgoCD 차트 `10.17.0`)을 반영한 최종 계획

## 확정 값

| 항목 | 값 |
|---|---|
| 내부 도메인 | `*.infra.choimory.dev` (ArgoCD: `argocd.infra.choimory.dev`) |
| Gateway | `infra-gateway` / 프록시 `infra-gateway-proxy` / GatewayClass `envoy-infra` / EnvoyProxy `envoy-proxy-infra` |
| 허용 라벨 | `infra-gateway-access: "true"` (`choimory-infra`에 적용) |
| 인증서 | ClusterIssuer `letsencrypt`, Certificate `infra-choimory-dev` → Secret `infra-choimory-dev-tls` |
| tailnet 기기 | Gateway `choimory-infra-gateway`, Operator `choimory-k8s-operator` |
| SealedSecret | `cloudflare-api-token`, `operator-oauth` |
| 설치 위치 | 전부 `choimory-infra` 네임스페이스 |
| 범위 | 4단계까지. 5단계(DB)는 끊고 따로 진행 |

## 작업 단계 (최종)

### A. 사전 준비 (최종)

- 사용자 진행, 화면과 명령은 Claude 안내

| 순서 | 위치 | 할 일 |
|---|---|---|
| A-1 | Cloudflare → My Profile → API Tokens | 토큰 생성. 권한 Zone·DNS·Edit, Zone·Zone·Read. 대상 `choimory.dev`만 |
| A-2 | Tailscale → Access controls | `tagOwners`에 `tag:k8s-operator`, `tag:k8s`(소유자 `tag:k8s-operator`) 추가 |
| A-3 | Tailscale → Settings → Trust credentials (OAuth) | OAuth 클라이언트 발급. 권한과 태그(`tag:k8s-operator`)는 작업 시점 Tailscale 문서 기준으로 안내 |
| A-4 | 서버 | 값을 `kubeseal`로 바로 암호화해 SealedSecret 2개 생성. 원문은 채팅, 문서에 남기지 않음 |

### B. gitops 1차 (최종)

- Claude 작성, 사용자 push

| 파일 | 내용 | wave |
|---|---|---|
| `infrastructure/apps/cert-manager.yaml`, `infrastructure/cert-manager/values.yaml` | cert-manager 차트, CRD 포함 설치, 리소스 최소화 | 5 |
| `infrastructure/apps/tailscale-operator.yaml`, `infrastructure/tailscale-operator/` | Operator 차트, values(기기 이름), `operator-oauth` SealedSecret | 5 |
| `infrastructure/apps/infra-gateway.yaml`, `infrastructure/infra-gateway/` | 아래 표의 리소스 | 6 |
| `infrastructure/apps/namespace.yaml` | `choimory-infra`에 `infra-gateway-access: "true"` 라벨 | -1 |
| `README.md` | 구성, 디렉토리, sync wave 표, 인프라 도구 접속 방식 | |

- `infrastructure/infra-gateway/` 구성

| 파일 | 리소스 |
|---|---|
| `gateway.yaml` | EnvoyProxy(`LoadBalancer` + `loadBalancerClass: tailscale`, 고정 이름 `infra-gateway-proxy`, tailnet 기기 이름 annotation), GatewayClass, Gateway(HTTPS 443, `*.infra.choimory.dev`, 인증서 참조, 라벨 Selector) |
| `issuer.yaml` | ClusterIssuer(Let's Encrypt prod, Cloudflare DNS-01) |
| `certificate.yaml` | `*.infra.choimory.dev` |
| `cloudflare-api-token.yaml` | SealedSecret |
| `kustomization.yaml` | 위 파일 묶음 |

- 차트 버전은 작성 시점 최신 안정 버전 확인 후 고정
- 작성 후 `kubectl kustomize`로 빌드 검증

### C. 확인 (최종)

- 서버에서 확인, 명령은 Claude 제공

1. cert-manager, Operator Pod Running
2. Certificate `Ready True` (보통 1~2분)
3. `infra-gateway-proxy` Service에 `100.x` IP, Tailscale 관리 화면에 `choimory-infra-gateway` 기기 생성

### D. DNS (최종)

- Cloudflare에서 사용자 진행
- `*.infra.choimory.dev` A 레코드 → `100.x`
- 반드시 회색 구름(DNS only)
- 이름이 더 구체적인 레코드라 기존 `*.choimory.dev` Tunnel 레코드보다 우선

### E. gitops 2차 (최종)

- Claude 작성, 사용자 push

| 파일 | 내용 |
|---|---|
| `infrastructure/argocd/values.yaml` | `configs.params."server.insecure": true`, `configs.cm.url: https://argocd.infra.choimory.dev`, `server.httproute`(parentRef `infra-gateway`, 호스트 `argocd.infra.choimory.dev`) |

### F. 확인과 계정 정리 (최종)

- 사용자 진행

1. Tailscale 켠 기기에서 `https://argocd.infra.choimory.dev` 접속 → 인증서 자물쇠, 로그인 화면 확인
2. Tailscale 끄고 접속 안 되는지 확인
3. 초기 비밀번호 로그인 → 비밀번호 변경 → `argocd-initial-admin-secret` 삭제

## 1차, 2차 push 구분

- 인증서와 tailnet IP가 준비되기 전에 ArgoCD를 `server.insecure`로 바꾸면 접속 경로가 꼬일 수 있음
- 2차 push 전까지는 지금 방식(port-forward)으로 ArgoCD 계속 사용 가능

## 주의할 점과 문제 생길 때 볼 곳

| 항목 | 내용 |
|---|---|
| Let's Encrypt | 처음부터 실제 발급(prod). 실패 시 반복 재시도하지 말고 Challenge 상태부터 확인 (발급 한도) |
| DNS-01 전파 확인 실패 | cert-manager가 클러스터 DNS로 TXT 레코드를 확인하지 못하면 재귀 DNS를 `1.1.1.1`로 지정하는 옵션 추가 |
| tailnet 프록시가 Gateway에 연결 안 됨 | Cilium 설정(kube-proxy 대체 여부) 확인. 지금은 kube-proxy를 써서 문제없을 것으로 봄 |
| `100.x` 주소가 조회 안 됨 | 쓰는 DNS(공유기, tailnet DNS)가 사설·CGNAT 주소 응답을 막는지 확인 (DNS rebinding 보호) |
| 메모리 | cert-manager Pod 3개, Operator, tailnet 프록시, 내부 Envoy 추가. requests를 작게 잡음 |

## 기록

- 작업 기록은 `task4.md`, calendar는 `TASK` 유지
- 커밋, push는 사용자 진행. 1차, 2차 두 번
- 결정 필요: A-1(Cloudflare API 토큰 발급) 시작 여부