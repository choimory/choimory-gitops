# 목차

- [개요](#개요)
- [전체 단계와 2단계 계획](#전체-단계와-2단계-계획)
  - [답변](#답변)
  - [전체 단계](#전체-단계)
  - [단계별 과정](#단계별-과정)
    - [1단계. 도메인과 Cloudflare DNS (완료)](#1단계-도메인과-cloudflare-dns-완료)
    - [2단계. Cloudflare Tunnel과 cloudflared](#2단계-cloudflare-tunnel과-cloudflared)
    - [3단계. 앱 도메인 확인](#3단계-앱-도메인-확인)
    - [4단계. ArgoCD 도메인, Cloudflare Access, 계정](#4단계-argocd-도메인-cloudflare-access-계정)
    - [5단계. DB, Redis Tailscale 전용 접속](#5단계-db-redis-tailscale-전용-접속)
    - [6단계. 마무리](#6단계-마무리)
  - [2단계 상세](#2단계-상세)
    - [목표](#목표)
    - [요청 흐름](#요청-흐름)
    - [진행 순서](#진행-순서)
    - [Envoy 프록시 Service 이름 고정](#envoy-프록시-service-이름-고정)
    - [Tunnel 생성과 토큰](#tunnel-생성과-토큰)
    - [토큰 SealedSecret 생성](#토큰-sealedsecret-생성)
    - [gitops 변경](#gitops-변경)
    - [Public Hostname과 DNS](#public-hostname과-dns)
    - [확인 방법](#확인-방법)
    - [주의할 점](#주의할-점)
  - [결정 필요 사항](#결정-필요-사항)
- [다음 작업 위치](#다음-작업-위치)
  - [답변](#답변-1)
  - [2단계 작업 위치](#2단계-작업-위치)
  - [1번 전에 확인할 것](#1번-전에-확인할-것)
- [추가1](#추가1)
- [추가1 분석](#추가1-분석)
  - [답변](#답변-2)
  - [1. Tunnel gitops 관리](#1-tunnel-gitops-관리)
  - [2, 4. Gateway 이름 변경 적용 확인](#2-4-gateway-이름-변경-적용-확인)
  - [3. 프록시 이름](#3-프록시-이름)
  - [결정 필요 사항 (추가1)](#결정-필요-사항-추가1)
- [추가2](#추가2)
- [추가2 분석](#추가2-분석)
  - [답변](#답변-3)
  - [2단계 진행 순서 (gitops 방식)](#2단계-진행-순서-gitops-방식)
  - [대시보드 방식과 달라진 점](#대시보드-방식과-달라진-점)
  - [1번 작업 내용](#1번-작업-내용)
- [cloudflared CLI 실행 위치](#cloudflared-cli-실행-위치)
  - [답변](#답변-4)
  - [홈서버가 나은 이유](#홈서버가-나은-이유)
  - [서버 설치 시 주의](#서버-설치-시-주의)
  - [서버 진행 순서](#서버-진행-순서)
  - [완료 후 정리](#완료-후-정리)
- [인증 파일 암호화와 gitops 작성 계획](#인증-파일-암호화와-gitops-작성-계획)
  - [답변](#답변-5)
  - [4번 인증 파일 암호화 (서버)](#4번-인증-파일-암호화-서버)
  - [5번 gitops 작성](#5번-gitops-작성)
  - [5번 전에 확인할 것](#5번-전에-확인할-것)
- [5번 gitops 작성 계획](#5번-gitops-작성-계획)
  - [답변](#답변-6)
  - [파일 구성](#파일-구성)
  - [파일별 내용](#파일별-내용)
  - [작성 후 검증](#작성-후-검증)
  - [push 후 확인 (5번)](#push-후-확인-5번)
  - [참고 (5번)](#참고-5번)
  - [결정 필요 사항 (5번)](#결정-필요-사항-5번)
- [sync wave 설명](#sync-wave-설명)
  - [답변](#답변-7)
  - [sync wave란](#sync-wave란)
  - [필요한 이유](#필요한-이유)
  - [이 저장소의 구성](#이-저장소의-구성)
  - [cloudflared를 7로 하는 이유](#cloudflared를-7로-하는-이유)
- [5번 이후 할 일](#5번-이후-할-일)
  - [답변](#답변-8)
  - [순서](#순서)
  - [1. 커밋, push](#1-커밋-push)
  - [2. 서버 확인](#2-서버-확인)
  - [3. DNS 레코드 생성](#3-dns-레코드-생성)
  - [4. 접속 확인](#4-접속-확인)
  - [5. 인증 파일 원본 삭제](#5-인증-파일-원본-삭제)

---

# 개요

- 1단계(도메인 구매, Cloudflare 연결)가 끝나 `choimory.dev`가 Cloudflare에서 Active 상태
- 전체 작업이 몇 단계이고 어떤 과정을 거치는지 다시 정리하고, 다음인 2단계(Cloudflare Tunnel, cloudflared)의 구체적인 계획을 세움
- 참고 문서: [plan2.md](./plan2.md)(전체 계획), [task1.md](./task1.md)(Gateway 이름 변경), [task2.md](./task2.md)(1단계 진행)

---

# 전체 단계와 2단계 계획

> 전체 몇단계가 있고 어떤 과정을 거치는지, 2단계에선 무엇을 할건지 plan3에 작성

## 답변

- 전체 6단계 (plan2의 5단계 + 마무리)
- 1단계 완료. 2단계는 Cloudflare 대시보드에서 Tunnel을 만들고, cloudflared를 클러스터에 배포해 Cloudflare와 Gateway를 연결하는 작업

## 전체 단계

| 단계 | 작업 | 주 작업 위치 | gitops 변경 | 상태 |
|---|---|---|---|---|
| 0 | Gateway 이름 변경 (`shared-gateway` → `gateway`) | gitops | 있음 | 완료 (task1, push 후 확인 필요) |
| 1 | 도메인 구매, Cloudflare DNS 연결 | 가비아, Cloudflare | 없음 | 완료 (SSL, 봇 설정 확인만 남음) |
| 2 | Cloudflare Tunnel 생성, cloudflared 배포 | Cloudflare, 서버, gitops | 있음 | 다음 |
| 3 | 앱 도메인 4개 접속 확인 | 브라우저 | 필요 시 | |
| 4 | ArgoCD 도메인, Cloudflare Access, 계정 설정 | gitops, Cloudflare | 있음 | |
| 5 | Tailscale Operator로 DB, Redis 접속 | Tailscale, 서버, gitops | 있음 | |
| 6 | 마무리 (done 문서, calendar DONE, README) | 문서 | README | |

- 5단계는 도메인과 무관해 순서를 바꿀 수 있음

## 단계별 과정

### 1단계. 도메인과 Cloudflare DNS (완료)

1. 가비아에서 `choimory.dev` 구매
2. Cloudflare에 사이트 추가 (Free), 레코드 없음 확인
3. 가비아 네임서버를 `jule.ns.cloudflare.com`, `micah.ns.cloudflare.com`으로 변경
4. Cloudflare Active 확인
5. 남은 것: Universal SSL Active 확인, Always Use HTTPS 켜기, AI Crawl Control 전부 Block

### 2단계. Cloudflare Tunnel과 cloudflared

1. Envoy 프록시 Service 이름 고정 (gitops)
2. Cloudflare에서 Tunnel `choimory-tunnel` 생성, 토큰 발급
3. 서버에서 토큰을 SealedSecret으로 암호화
4. cloudflared를 클러스터에 배포 (gitops, push)
5. Tunnel에 `choimory.dev`, `*.choimory.dev` 연결, DNS 레코드 확인
- 상세는 아래 [2단계 상세](#2단계-상세)

### 3단계. 앱 도메인 확인

| 앱 | stg | prod |
|---|---|---|
| front | `staging.choimory.dev` | `choimory.dev` |
| member-api | `staging-member-api.choimory.dev` | `member-api.choimory.dev` |

1. 브라우저에서 front 화면, 회원가입, 로그인 흐름 확인
2. member-api 직접 호출 확인
3. CORS, 쿠키 동작 확인 (k8s-setup에서 미뤄 둔 브라우저 흐름)
4. 문제 시 Tunnel 상태, cloudflared 로그, HTTPRoute 상태, Envoy 로그 순서로 확인

### 4단계. ArgoCD 도메인, Cloudflare Access, 계정

1. `choimory-infra` 네임스페이스에 `gateway-access: "true"` 라벨 추가
2. ArgoCD values에 `server.insecure: true`, `configs.cm.url` 추가
3. ArgoCD HTTPRoute 추가 (도메인 결정 필요, 예: `argocd.choimory.dev`)
4. Cloudflare Zero Trust에서 Access Application과 인프라 도구 공통 정책 생성 (로그인 수단 결정 필요)
5. Access 적용 확인 후 초기 비밀번호로 로그인 → 비밀번호 변경 → `argocd-initial-admin-secret` 삭제

- 도메인은 와일드카드로 Tunnel에 이미 연결되므로 Cloudflare Tunnel 설정 변경 없음

### 5단계. DB, Redis Tailscale 전용 접속

1. Tailscale 관리 화면에서 OAuth 클라이언트 발급 → SealedSecret
2. Tailscale Operator Helm 차트를 ArgoCD로 설치 (`choimory-infra`)
3. stg, prod의 PostgreSQL, Redis를 tailnet에 노출 (CNPG Service 처리 방식 결정 필요)
4. Mac에서 tailnet 이름으로 DataGrip 접속 확인 (prod는 Read-only)

### 6단계. 마무리

1. `done1.md`에 전체 작업 요약
2. calendar를 `DONE:도메인 연동 작업(domain-connect)`으로 변경
3. README에 도메인, Tunnel, Access, Tailscale 구성 반영

## 2단계 상세

### 목표

- 외부에서 `https://*.choimory.dev`로 들어온 요청이 Cloudflare Tunnel을 거쳐 클러스터의 Gateway까지 도달
- 이후 도메인을 추가할 때 HTTPRoute만 만들면 되고 Cloudflare 설정은 바꾸지 않아도 되는 구조

### 요청 흐름

```
브라우저
  → Cloudflare (Universal SSL로 HTTPS 처리, AI 봇 차단)
  → Tunnel (choimory-tunnel)
  → cloudflared Pod (choimory-infra, Cloudflare로 나가는 연결만 사용)
  → Envoy 프록시 Service (choimory-infra, HTTP 80)
  → HTTPRoute 호스트 매칭
  → 앱 Service (choimory-dev-stg / choimory-dev-prod)
```

- 서버로 들어오는 포트는 열지 않음. cloudflared가 Cloudflare로 연결을 먼저 맺고 그 연결로 요청을 받음
- cloudflared는 원래 Host 헤더를 그대로 전달하므로 HTTPRoute의 hostnames 매칭이 그대로 동작

### 진행 순서

| 순서 | 위치 | 작업 | 담당 |
|---|---|---|---|
| 1 | gitops | EnvoyProxy에 Service 고정 이름 지정 | Claude 작성, 사용자 push |
| 2 | 서버 | Envoy Service가 고정 이름으로 바뀌었는지 확인 | 사용자 |
| 3 | Cloudflare | Tunnel 생성, 토큰 복사 | 사용자 |
| 4 | 서버 | 토큰을 SealedSecret으로 암호화, 결과 파일 전달 | 사용자 |
| 5 | gitops | cloudflared Application, Deployment, SealedSecret 추가 | Claude 작성, 사용자 push |
| 6 | Cloudflare | Tunnel 상태 Healthy 확인, Public Hostname 연결 | 사용자 |
| 7 | 브라우저, 터미널 | 접속 확인 (3단계로 이어짐) | 사용자, Claude |

- 1과 5는 한 번에 push해도 되지만, Service 이름 변경을 먼저 확인하면 문제 위치를 나누기 쉬워 분리 추천

### Envoy 프록시 Service 이름 고정

- 현재 Envoy Gateway가 자동으로 만드는 이름은 `envoy-choimory-infra-gateway-<해시>`
- Tunnel 대상 주소에 해시가 들어가면 Gateway를 다시 만들 때마다 Cloudflare 설정도 바꿔야 함
- Envoy Gateway v1.10 API에 `envoyService.name`(Service 이름), `envoyDeployment.name`(Deployment 이름) 필드가 있음을 확인
- 변경안 (`infrastructure/gateway/gateway.yaml`의 EnvoyProxy)

```yaml
spec:
  provider:
    type: Kubernetes
    kubernetes:
      envoyService:
        type: ClusterIP
        name: gateway-envoy          # 이름은 결정 필요
      envoyDeployment:
        name: gateway-envoy          # 이름은 결정 필요
        replicas: 1
```

- Tunnel 대상: `http://gateway-envoy.choimory-infra.svc.cluster.local:80`
- 주의: EnvoyProxy는 GatewayClass 단위 설정이라, 같은 GatewayClass로 Gateway를 두 개 이상 만들면 이름이 겹침. 지금은 Gateway가 하나라 문제없고, 주석으로 남겨 둠
- 이름 변경 시 Envoy Pod와 Service가 새로 만들어짐. 아직 외부 트래픽이 없어 영향 없음

### Tunnel 생성과 토큰

1. Cloudflare 대시보드 → Zero Trust → Networks → Tunnels → Create a tunnel
2. 종류는 Cloudflared, 이름은 `choimory-tunnel`
3. 설치 안내 화면의 명령어에서 `--token` 뒤의 토큰 값만 복사
   - 화면의 설치 명령(서버에 직접 설치)은 실행하지 않음. cloudflared는 클러스터에 배포
4. Public Hostname 설정 화면은 cloudflared 배포 후(Healthy 확인 후) 진행해도 됨

- Tunnel 관리 방식은 대시보드 관리(remote-managed, 토큰 방식) 기준 (결정 필요)
- 토큰은 Tunnel에 접속할 수 있는 비밀값이므로 문서, 채팅, 저장소에 원문을 남기지 않음

### 토큰 SealedSecret 생성

- 서버에서 실행. 토큰이 셸 히스토리에 남지 않도록 `read -s`로 입력

```bash
read -s TUNNEL_TOKEN   # 토큰 붙여넣고 Enter
kubectl create secret generic choimory-cloudflared-token \
  -n choimory-infra \
  --from-literal=token="$TUNNEL_TOKEN" \
  --dry-run=client -o yaml \
| kubeseal --controller-namespace choimory-infra -o yaml \
> choimory-cloudflared-token.yaml
unset TUNNEL_TOKEN
```

- 결과 파일은 암호화되어 있어 저장소에 커밋 가능. `infrastructure/cloudflared/`에 넣음

### gitops 변경

| 파일 | 내용 |
|---|---|
| `infrastructure/gateway/gateway.yaml` | EnvoyProxy에 Service, Deployment 고정 이름 |
| `infrastructure/apps/cloudflared.yaml` | Application `choimory-cloudflared` (sync wave 6, gateway와 같은 단계) |
| `infrastructure/cloudflared/kustomization.yaml` | 네임스페이스 `choimory-infra`, 리소스 목록 |
| `infrastructure/cloudflared/deployment.yaml` | Deployment `choimory-cloudflared` |
| `infrastructure/cloudflared/choimory-cloudflared-token.yaml` | 토큰 SealedSecret (서버에서 생성한 파일) |
| `README.md` | 구성 표, 디렉토리 구조, sync wave 표, 도메인 항목 |

- Deployment 구성
  - 이미지: `cloudflare/cloudflared` 공식 이미지. 작업 시 최신 버전을 확인해 태그 고정 (`latest` 미사용)
  - 실행: `tunnel --no-autoupdate --metrics 0.0.0.0:2000 run`, 토큰은 환경변수 `TUNNEL_TOKEN`으로 Secret에서 주입
  - 상태 확인: metrics 포트의 `/ready`로 liveness, readiness probe
  - 레플리카 1개 (노드 1대)
  - 메모리 요청 32Mi, 상한 128Mi 수준에서 시작
- 네트워크: cloudflared와 Envoy 모두 `choimory-infra`에 있고 이 네임스페이스에는 NetworkPolicy가 없어 추가 설정 불필요

### Public Hostname과 DNS

| Public Hostname | 대상 (Service) |
|---|---|
| `choimory.dev` | `http://gateway-envoy.choimory-infra.svc.cluster.local:80` |
| `*.choimory.dev` | `http://gateway-envoy.choimory-infra.svc.cluster.local:80` |

- 둘 다 필요: `*.choimory.dev`는 `choimory.dev`에 매칭되지 않음 (Gateway listener에 hostname을 두지 않은 것과 같은 이유)
- DNS 레코드
  - `choimory.dev`: Public Hostname 저장 시 CNAME(→ `<TUNNEL_ID>.cfargotunnel.com`, Proxied)이 자동 생성
  - `*.choimory.dev`: 자동 생성되지 않을 수 있음. DNS → Records에서 확인하고 없으면 `*` CNAME을 같은 대상으로 직접 추가 (Proxied)
- Universal SSL이 `choimory.dev`, `*.choimory.dev`를 덮으므로 인증서 추가 작업 없음
- HTTPRoute에 없는 호스트(예: `www.choimory.dev`)는 Envoy가 404 응답

### 확인 방법

```bash
# 서버: Envoy Service 고정 이름
kubectl -n choimory-infra get svc gateway-envoy

# 서버: cloudflared 상태와 로그
kubectl -n choimory-infra get pod -l app=choimory-cloudflared
kubectl -n choimory-infra logs deploy/choimory-cloudflared | grep -i "registered tunnel connection"

# Mac: DNS와 HTTP 응답
dig +short choimory.dev
dig +short staging.choimory.dev
curl -I https://choimory.dev
curl -I https://staging-member-api.choimory.dev
```

- 기대 결과
  - Cloudflare 대시보드의 Tunnel 상태 Healthy
  - cloudflared 로그에 Tunnel 연결 등록 메시지
  - `dig`가 Cloudflare IP를 반환
  - `curl`이 앱 응답 (front 200, member-api는 경로에 따라 404/401 등 앱 응답)
- `curl`이 Cloudflare 오류(502, 530 등)면 Tunnel 대상 주소나 cloudflared 상태 확인

### 주의할 점

- Tunnel 토큰, Tunnel ID는 문서에 마스킹
- 앞서 AI Crawl Control을 전부 Block으로 두면 `curl` 기본 User-Agent는 영향 없지만, 일부 확인 도구가 봇으로 분류될 수 있음
- task1의 Gateway 이름 변경이 아직 push 전이라면 같이 반영됨. push 순서: Gateway 이름 변경 → Envoy 고정 이름 → cloudflared

## 결정 필요 사항

1. Tunnel 관리 방식: 대시보드 관리(추천) / 설정 파일 관리
2. Envoy 프록시 Service, Deployment 고정 이름 (예: `gateway-envoy`)
3. push 분리 여부: Envoy 고정 이름 먼저 확인 후 cloudflared (추천) / 한 번에

---

# 다음 작업 위치

> 다음작업은 gitops에서 하는건가?

## 답변

- 추천 순서대로면 다음 작업은 gitops (Envoy 고정 이름)
- 2단계는 gitops, Cloudflare 대시보드, 서버를 번갈아 진행

## 2단계 작업 위치

| 순서 | 위치 | 작업 | 담당 |
|---|---|---|---|
| 1 | gitops | EnvoyProxy에 Service, Deployment 고정 이름 지정 | Claude 수정, 사용자 push |
| 2 | 서버 | `kubectl -n choimory-infra get svc`로 고정 이름 확인 | 사용자 |
| 3 | Cloudflare | Tunnel `choimory-tunnel` 생성, 토큰 복사 | 사용자 |
| 4 | 서버 | 토큰을 SealedSecret 파일로 암호화 | 사용자 |
| 5 | gitops | cloudflared Application, Deployment, SealedSecret 추가 | Claude 작성, 사용자 push |
| 6 | Cloudflare | Tunnel Healthy 확인, Public Hostname 연결 | 사용자 |
| 7 | 브라우저, 터미널 | 접속 확인 | 사용자, Claude |

- 3번(Tunnel 생성)은 gitops와 무관해 1번과 동시에 진행 가능

## 1번 전에 확인할 것

- task1(Gateway 이름 변경) push 여부
  - push 전이면 1번 변경과 같이 push해도 됨. 둘 다 Envoy Pod, Service를 다시 만드는 변경이라 한 번에 반영하는 편이 깔끔함
  - 이미 push했다면 서버에서 `kubectl -n choimory-infra get gateway`로 `gateway`가 `PROGRAMMED True`인지 먼저 확인
- 고정 이름
  - `gateway-envoy` 또는 `choimory-gateway-envoy`
  - 새 공용 자원에 `choimory-`를 붙이는 규칙이 있지만, Gateway 이름을 `gateway`로 단순하게 정한 흐름에는 `gateway-envoy`가 더 맞는다고 봄

---

# 추가1

1. Tunnel 관리 방식: gitops로 하고 싶은데 별론가?
2. gateway 이름 변경 적용 되어있음
3. gateway-envoy -> 이미 envoy-gateway가 있는데 그럼 envoy-gateway, gateway-envoy가 모두 있는건가? 헷갈릴거 같은데
4. gateway programmed는 true임

---

# 추가1 분석

> 1. Tunnel 관리 방식: gitops로 하고 싶은데 별론가?
> 2. gateway 이름 변경 적용 되어있음
> 3. gateway-envoy -> 이미 envoy-gateway가 있는데 그럼 envoy-gateway, gateway-envoy가 모두 있는건가? 헷갈릴거 같은데
> 4. gateway programmed는 true임

## 답변

- Tunnel gitops 관리는 별로가 아님. 저장소 전체가 gitops라 오히려 일관됨. 대시보드 관리 추천은 간단해서였을 뿐
- task1 반영 확인됨
- 프록시 이름은 `envoy-gateway`와 헷갈리지 않게 `gateway-proxy` 추천

## 1. Tunnel gitops 관리

| | 대시보드 관리 (토큰) | gitops 관리 (설정 파일) |
|---|---|---|
| 라우팅 규칙 위치 | Cloudflare 대시보드 | gitops ConfigMap (Git 이력) |
| 비밀값 | 토큰 1개 | 인증 파일(credentials JSON) 1개 |
| Tunnel 생성 | 대시보드 클릭 | Mac에서 `cloudflared` CLI (1회) |
| DNS 레코드 | Public Hostname 저장 시 자동 생성 | CLI(`cloudflared tunnel route dns`)나 대시보드에서 추가 (1회) |
| 대시보드에서 수정 | 가능 | 불가 (읽기 전용 표시) |

- Git에 남는 라우팅 규칙 (와일드카드라 거의 바뀌지 않음)

```yaml
ingress:
  - hostname: choimory.dev
    service: http://<Envoy Service>.choimory-infra.svc.cluster.local:80
  - hostname: "*.choimory.dev"
    service: http://<Envoy Service>.choimory-infra.svc.cluster.local:80
  - service: http_status:404
```

- 처음 한 번 하는 작업
  1. Mac에 `brew install cloudflared`
  2. `cloudflared tunnel login`으로 브라우저 인증
  3. `cloudflared tunnel create choimory-tunnel` → 인증 파일 생성
  4. 인증 파일을 SealedSecret으로 암호화
- gitops 밖에 남는 것: DNS 레코드 2개(`choimory.dev`, `*.choimory.dev` → Tunnel). 코드 관리는 Terraform이 필요하므로 처음 한 번 수동 생성 추천

## 2, 4. Gateway 이름 변경 적용 확인

- task1 반영 확인 (Gateway `gateway` `PROGRAMMED True`)
- HTTPRoute 연결 확인 명령 (4개 모두 `True`면 정상)

```bash
kubectl get httproute -A -o custom-columns='NS:.metadata.namespace,NAME:.metadata.name,ACCEPTED:.status.parents[0].conditions[?(@.type=="Accepted")].status'
```

## 3. 프록시 이름

| 이름 | 정체 |
|---|---|
| `envoy-gateway` | 컨트롤러 (Gateway를 보고 프록시를 만드는 관리자). 차트 기본 이름이라 유지 |
| `gateway-envoy` (기존 제안) | 실제로 요청을 받는 프록시 |

- 단어 순서만 바뀐 이름이 같은 네임스페이스에 있으면 헷갈림. 역할이 드러나는 이름으로 변경

| 후보 | 의견 |
|---|---|
| `gateway-proxy` | 추천. "`gateway`의 프록시"라는 뜻이 보이고 `envoy-gateway`와 단어가 겹치지 않음 |
| `envoy-proxy` | 비추천. EnvoyProxy 설정 리소스 이름 `envoy-proxy`와 같아짐 |
| `choimory-gateway-proxy` | 가능. 다만 `gateway`를 단순하게 정한 흐름과 결이 다름 |

```
envoy-gateway   → 컨트롤러
gateway         → Gateway (입구 정의)
gateway-proxy   → 실제 요청을 받는 Envoy Pod, Service
```

- Tunnel 대상 주소: `http://gateway-proxy.choimory-infra.svc.cluster.local:80`

## 결정 필요 사항 (추가1)

1. Tunnel 관리 방식을 gitops로 확정할지
2. 프록시 이름을 `gateway-proxy`로 할지

---

# 추가2

- tunnel은 gitops로 관리하자
- 프록시 이름은 gateway-proxy로 하자

---

# 추가2 분석

> - tunnel은 gitops로 관리하자
> - 프록시 이름은 gateway-proxy로 하자

## 답변

| 항목 | 결정 |
|---|---|
| Tunnel 관리 방식 | gitops (설정 파일 방식) |
| Envoy 프록시 이름 | `gateway-proxy` |

- gitops 방식이라 2단계 순서가 대시보드 방식 기준으로 적은 [2단계 상세](#2단계-상세)와 일부 달라짐

## 2단계 진행 순서 (gitops 방식)

| 순서 | 위치 | 작업 | 담당 |
|---|---|---|---|
| 1 | gitops | EnvoyProxy의 Service, Deployment 이름을 `gateway-proxy`로 고정 | Claude 수정, 사용자 push |
| 2 | 서버 | `kubectl -n choimory-infra get svc gateway-proxy` 확인 | 사용자 |
| 3 | Mac | `brew install cloudflared` → `cloudflared tunnel login` → `cloudflared tunnel create choimory-tunnel` → `~/.cloudflared/<UUID>.json` 생성 | 사용자 |
| 4 | Mac 또는 서버 | 인증 파일을 SealedSecret `choimory-cloudflared-credentials`로 암호화 | 사용자 |
| 5 | gitops | Application `choimory-cloudflared`, ConfigMap(라우팅 규칙), Deployment, SealedSecret 추가 | Claude 작성, 사용자 push |
| 6 | Mac | `cloudflared tunnel route dns`로 `choimory.dev`, `*.choimory.dev` DNS 레코드 생성 (1회) | 사용자 |
| 7 | 브라우저, 터미널 | Tunnel Healthy, `dig`, `curl` 확인 | 사용자, Claude |

- 3번은 1번과 별개라 먼저 진행 가능

## 대시보드 방식과 달라진 점

- 비밀값: 토큰 대신 인증 파일(JSON). SealedSecret 이름 `choimory-cloudflared-token` → `choimory-cloudflared-credentials`
- 라우팅 규칙: ConfigMap에 `choimory.dev`, `*.choimory.dev` → `gateway-proxy`, 나머지 404
- Tunnel UUID: ConfigMap에 들어감. UUID만으로는 Tunnel에 접속할 수 없어 비밀값은 아니지만 histories 문서에는 마스킹
- 암호화 위치: Mac에 `kubectl`, `kubeseal`이 있으면 Mac에서 바로 암호화 추천 (인증 파일을 서버로 옮기지 않음). 없으면 서버로 옮겨 암호화 후 원본 삭제
- `~/.cloudflared/cert.pem`(로그인 인증서): Mac에 보관. DNS 연결, Tunnel 삭제 등 관리 명령에 필요

## 1번 작업 내용

| 파일 | 변경 |
|---|---|
| `infrastructure/gateway/gateway.yaml` | EnvoyProxy에 `envoyService.name: gateway-proxy`, `envoyDeployment.name: gateway-proxy` 추가. Gateway가 하나일 때만 유효하다는 주석 |
| `README.md` | 프록시 이름 반영 |
| `task3.md` | 2단계 작업 기록 시작 |

- push 시 Envoy 프록시가 `gateway-proxy`로 새로 생성됨. 외부 트래픽이 없어 영향 없음

---

# cloudflared CLI 실행 위치

> 로컬 맥에서 cloudfalred 설치해야하는 이유가 있어? 로컬에 kubectl 안깔았고, 홈서버에만 있는데

## 답변

- Mac에서 할 이유 없음. 홈서버에서 하는 편이 나음
- Mac을 제안한 이유는 로그인에 브라우저가 필요해서였지만, 서버에서도 URL을 Mac 브라우저로 열면 됨

## 홈서버가 나은 이유

| | Mac | 홈서버 |
|---|---|---|
| kubectl, kubeseal | 없음 → 설치 필요 | 이미 있음 |
| 인증 파일 이동 | Mac에서 만들어 서버로 복사 | 만든 자리에서 바로 암호화 |
| 브라우저 로그인 | 바로 열림 | 터미널에 나온 URL을 Mac 브라우저에서 열기 |

## 서버 설치 시 주의

- CLI로만 사용. `cloudflared service install`은 실행하지 않음. 실제 Tunnel 연결은 클러스터의 cloudflared Pod가 담당하고, 서버의 cloudflared는 Tunnel 생성, DNS 연결 명령에만 사용
- 로그인 인증서 `~/.cloudflared/cert.pem`은 `choimory.dev` 영역의 Tunnel, DNS를 관리할 수 있는 파일이라 권한 600

## 서버 진행 순서

```bash
# 0. 아키텍처 확인 (amd64 또는 arm64)
dpkg --print-architecture

# 1. 설치 (amd64 기준. arm64면 cloudflared-linux-arm64.deb)
curl -L -o /tmp/cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i /tmp/cloudflared.deb && rm /tmp/cloudflared.deb
cloudflared --version

# 2. 로그인: 출력된 URL을 Mac 브라우저에서 열고 choimory.dev 선택 → 서버에 ~/.cloudflared/cert.pem 생성
cloudflared tunnel login

# 3. Tunnel 생성 → ~/.cloudflared/<UUID>.json 생성, UUID 출력
cloudflared tunnel create choimory-tunnel
chmod 600 ~/.cloudflared/*
```

- 4번(암호화), 6번(DNS 연결)도 같은 서버에서 진행. 6번은 cloudflared Pod 배포 후 진행해도 됨
- 2단계 진행 순서([추가2 분석](#2단계-진행-순서-gitops-방식))의 3, 4, 6번 위치가 Mac에서 홈서버로 바뀜

## 완료 후 정리

- `<UUID>.json`(인증 파일): SealedSecret으로 Git에 들어가면 서버 원본 삭제 추천. 필요하면 sealed-secrets 키로 복구하거나 Tunnel 재생성
- `cert.pem`: 이후 Tunnel, DNS 관리에 필요하므로 보관. 관리할 일이 없으면 삭제하고 필요할 때 `cloudflared tunnel login`으로 다시 발급
- Docker로 한 번만 실행하는 방법도 있지만 볼륨 권한 처리가 번거로워 `.deb` 설치 추천

---

# 인증 파일 암호화와 gitops 작성 계획

> ```
> choimory@choimory-server:~$ 
> cloudflared tunnel create choimory-tunnel
> chmod 600 ~/.cloudflared/*
> Tunnel credentials written to /home/choimory/.cloudflared/<TUNNEL_ID>.json. cloudflared chose this file based on where your origin certificate was found. Keep this file secret. To revoke these credentials, delete the tunnel.
>
> Created tunnel choimory-tunnel with id <TUNNEL_ID>
> ```

## 답변

| 항목 | 값 |
|---|---|
| Tunnel 이름 | `choimory-tunnel` |
| Tunnel ID | `<TUNNEL_ID>` (ConfigMap 작성에 사용, 문서에는 마스킹) |
| 인증 파일 | `~/.cloudflared/<TUNNEL_ID>.json` (비밀값) |

## 4번 인증 파일 암호화 (서버)

```bash
kubectl create secret generic choimory-cloudflared-credentials \
  -n choimory-infra \
  --from-file=credentials.json=$HOME/.cloudflared/<TUNNEL_ID>.json \
  --dry-run=client -o yaml \
| kubeseal --controller-namespace choimory-infra -o yaml \
> ~/choimory-cloudflared-credentials.yaml

cat ~/choimory-cloudflared-credentials.yaml
```

- 결과 파일은 `kind: SealedSecret`, `encryptedData.credentials.json: AgB...` 형태
- 클러스터의 sealed-secrets 키로만 풀 수 있어 채팅 공유, 저장소 커밋 가능 (기존 SealedSecret과 동일)
- SealedSecret은 이름(`choimory-cloudflared-credentials`)과 네임스페이스(`choimory-infra`)에 묶여 암호화되므로 바꾸면 안 됨

## 5번 gitops 작성

| 파일 | 내용 |
|---|---|
| `infrastructure/apps/cloudflared.yaml` | Application `choimory-cloudflared` |
| `infrastructure/cloudflared/kustomization.yaml` | 리소스 목록 |
| `infrastructure/cloudflared/config.yaml` | ConfigMap: Tunnel ID, `choimory.dev`, `*.choimory.dev` → `gateway-proxy`, 나머지 404 |
| `infrastructure/cloudflared/deployment.yaml` | `cloudflare/cloudflared:2026.11.0`, `/ready` probe |
| `infrastructure/cloudflared/choimory-cloudflared-credentials.yaml` | 서버에서 만든 SealedSecret |
| `README.md` | 구성, 디렉토리, sync wave 반영 |

## 5번 전에 확인할 것

- 5번 작성 시작 허락
- 1번(`gateway-proxy`) push 여부와 확인 결과. cloudflared가 `gateway-proxy` Service로 연결하므로 먼저 있어야 함

---

# 5번 gitops 작성 계획

> 5번 gitops 작성계획 한번 말해줘

## 답변

- `infrastructure/cloudflared/` 디렉토리와 Application 하나 추가. 서버 수동 작업 없이 push만 하면 됨

## 파일 구성

```
infrastructure/
├── apps/
│   └── cloudflared.yaml                      # Application choimory-cloudflared (신규)
└── cloudflared/                              # 신규
    ├── kustomization.yaml                    # 네임스페이스, 리소스, ConfigMap 생성
    ├── config.yaml                           # cloudflared 설정 원문 (라우팅 규칙)
    ├── deployment.yaml                       # Deployment choimory-cloudflared
    └── choimory-cloudflared-credentials.yaml # 서버에서 만든 SealedSecret
```

## 파일별 내용

### 1. infrastructure/apps/cloudflared.yaml

- `apps/gateway.yaml`과 같은 형식의 Application `choimory-cloudflared`. 경로 `infrastructure/cloudflared`, 대상 `choimory-infra`, prune, selfHeal
- sync wave 6 → 7 제안. cloudflared가 `gateway-proxy`(wave 6)를 향하므로 그 뒤가 자연스러움. `choimory-dev`도 7이지만 서로 무관

### 2. infrastructure/cloudflared/config.yaml

```yaml
tunnel: <TUNNEL_ID>                 # 비밀값 아님. 실제 파일에는 실제 값
credentials-file: /etc/cloudflared/creds/credentials.json
metrics: 0.0.0.0:2000               # /ready 상태 확인용
no-autoupdate: true                 # 버전은 이미지 태그로 관리
ingress:
  - hostname: choimory.dev
    service: http://gateway-proxy.choimory-infra.svc.cluster.local:80
  - hostname: "*.choimory.dev"
    service: http://gateway-proxy.choimory-infra.svc.cluster.local:80
  - service: http_status:404        # 그 외 호스트
```

### 3. infrastructure/cloudflared/kustomization.yaml

- `namespace: choimory-infra`, 리소스는 `deployment.yaml`, SealedSecret
- ConfigMap은 `configMapGenerator`로 `config.yaml`에서 `choimory-cloudflared-config` 생성
  - 이름 뒤에 내용 해시가 붙어 라우팅 규칙을 바꾸면 Deployment가 자동 재시작
  - 일반 ConfigMap이면 설정을 바꿔도 Pod가 예전 설정을 계속 사용

### 4. infrastructure/cloudflared/deployment.yaml

| 항목 | 값 |
|---|---|
| 이름, 라벨 | `choimory-cloudflared`, `app: choimory-cloudflared` |
| 이미지 | `cloudflare/cloudflared:2026.11.0` (서버 CLI와 같은 버전) |
| 실행 | `tunnel --config /etc/cloudflared/config/config.yaml run` |
| 마운트 | ConfigMap → `/etc/cloudflared/config`, Secret → `/etc/cloudflared/creds` (읽기 전용) |
| probe | liveness, readiness 모두 `GET /ready :2000` |
| 자원 | 요청 cpu 10m, 메모리 32Mi / 상한 메모리 128Mi (CPU 상한 없음) |
| 레플리카 | 1 (노드 1대) |

### 5. infrastructure/cloudflared/choimory-cloudflared-credentials.yaml

- 서버에서 만든 SealedSecret 그대로 (맨 앞 `---`만 제외). 기존 SealedSecret 파일처럼 맨 위에 설명 주석

### 6. README.md

- 구성 표에 외부 접속(Cloudflare Tunnel `choimory-tunnel`, 클러스터 안 cloudflared → `gateway-proxy`) 추가
- 디렉토리 구조에 `cloudflared/`, sync wave 표에 `7 | cloudflared`
- 도메인 항목의 "외부 연결(Cloudflare Tunnel)은 아직 구성 전" 문구 수정

### 7. task3.md

- 5번 작업 기록

## 작성 후 검증

- `kubectl kustomize infrastructure/cloudflared`로 ConfigMap 해시 이름, Deployment 참조, 마운트 경로 확인
- ConfigMap 안의 Tunnel ID가 실제 값과 같은지 확인

## push 후 확인 (5번)

```bash
kubectl -n choimory-infra get pod -l app=choimory-cloudflared
kubectl -n choimory-infra logs deploy/choimory-cloudflared | grep -i "registered tunnel connection"
cloudflared tunnel info choimory-tunnel
```

- 기대 결과: Pod Running, 로그에 연결 등록 메시지, `tunnel info`에 연결 표시, 대시보드 Healthy
- DNS 레코드가 아직 없어 브라우저 접속은 6번(`cloudflared tunnel route dns`) 이후

## 참고 (5번)

- cloudflared는 기본으로 QUIC(UDP 7844 아웃바운드)으로 연결하고 막혀 있으면 HTTP/2로 자동 전환. 연결이 안 되면 `protocol: http2` 지정
- 1번(`gateway-proxy`)과 같이 push하면 cloudflared가 먼저 떠서 잠깐 502가 날 수 있으나 Service가 생기면 자동 해결

## 결정 필요 사항 (5번)

1. sync wave 7로 변경 여부 (추천)
2. 이 계획대로 작성 시작 여부

---

# sync wave 설명

> 응 7로 하고 작업 시작해줘
>
> sync wave 설명한번

## 답변

- sync wave 7로 확정. 작업 시작 직후 설명 요청으로 중단되어 파일 변경은 없음
- sync wave는 ArgoCD가 리소스를 적용하는 순서를 정하는 번호

## sync wave란

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "6"
```

- 숫자가 작은 것부터 적용 (음수 가능, 기본값 0)
- 한 wave의 리소스가 모두 Healthy가 되어야 다음 wave로 진행
- 같은 wave 안의 리소스는 순서 보장 없음 (동시에 적용)

## 필요한 이유

| 먼저 있어야 하는 것 | 그걸 쓰는 것 | 순서가 틀리면 |
|---|---|---|
| sealed-secrets 컨트롤러 | SealedSecret을 쓰는 앱 | Secret이 안 풀려 앱이 안 뜸 |
| envoy-gateway 컨트롤러 (Gateway CRD) | Gateway, HTTPRoute | "Gateway라는 종류가 없음" 오류로 적용 실패 |
| cloudnative-pg 컨트롤러 | PostgreSQL Cluster | 같은 이유로 실패 |

- ArgoCD는 기본적으로 한꺼번에 적용하므로 순서가 중요한 곳에 wave로 지정

## 이 저장소의 구성

| wave | Application | 앞 wave가 필요한 이유 |
|---|---|---|
| -1 | Namespace `choimory-infra` | 모든 것이 이 네임스페이스에 설치됨 |
| 0 | argocd | ArgoCD 자신 |
| 1 | sealed-secrets | 이후 SealedSecret을 쓰는 것들보다 먼저 |
| 2 | local-path-provisioner | DB PVC보다 먼저 |
| 3 | metrics-server | |
| 4 | cloudnative-pg | PostgreSQL Cluster CRD 제공 |
| 5 | envoy-gateway | Gateway API CRD와 컨트롤러 제공 |
| 6 | gateway | Gateway, `gateway-proxy` 생성 (5번 CRD 필요) |
| 7 | choimory-dev | 프로젝트 앱 (HTTPRoute, DB 등이 4~6번 필요) |
| 7 (신규) | choimory-cloudflared | `gateway-proxy`(6)로 연결하므로 그 뒤 |

- Application Health 판단: ArgoCD는 원래 자식 Application의 Health를 기다리지 않아 wave를 붙여도 순서가 지켜지지 않음. `infrastructure/argocd/values.yaml`에 Health 판단을 추가해 "앞 wave Application이 Healthy일 때까지 대기"가 실제로 동작
- wave는 같은 묶음 안에서만 의미: `gateway.yaml` 안 EnvoyProxy의 `sync-wave: "-1"`은 그 Application 안에서 GatewayClass보다 먼저라는 뜻이며 바깥 Application wave와 별개

## cloudflared를 7로 하는 이유

- cloudflared는 SealedSecret(1번 필요)과 `gateway-proxy`(6번 필요)에 의존
- 6이면 gateway와 같은 wave라 `gateway-proxy`보다 먼저 뜰 수 있음 (잠깐 502, 이후 자동 해결)
- 7이면 `gateway-proxy`가 준비된 뒤 실행. `choimory-dev`와 같은 wave지만 서로 무관
- wave는 클러스터 최초 설치나 전체 재sync 때 의미가 큼. 이미 떠 있는 클러스터에 Application 하나를 추가하는 지금은 영향이 거의 없지만 재설치를 위해 맞게 둠

---

# 5번 이후 할 일

> 이제 해야하는거 뭐지?

## 답변

- 커밋, push 후 서버 확인과 DNS 연결로 2단계 마무리

## 순서

| 순서 | 위치 | 할 일 |
|---|---|---|
| 1 | 로컬 | 변경 사항 커밋, `main` push |
| 2 | 서버 | `gateway-proxy`, cloudflared 확인 |
| 3 | 서버 | DNS 레코드 2개 생성 |
| 4 | Mac | 도메인 접속 확인 (3단계 시작) |
| 5 | 서버 | 인증 파일 원본 삭제 |
| 6 | Cloudflare | 1단계 남은 설정 (SSL, Always Use HTTPS, AI Crawl Block) |

## 1. 커밋, push

- 커밋 안 된 변경: `infrastructure/apps/cloudflared.yaml`(신규), `infrastructure/cloudflared/`(신규 4개), `README.md`, `.agents/histories/2026/11/16/`(plan1~3, task1~3)
- `gateway-proxy` 변경(`gateway.yaml`)은 이미 커밋된 것으로 보임. push 전이면 같이 반영

## 2. 서버 확인

```bash
kubectl -n choimory-infra get deploy,svc | grep gateway-proxy
kubectl -n choimory-infra get pod -l app=choimory-cloudflared
kubectl -n choimory-infra logs deploy/choimory-cloudflared | grep -i "registered tunnel connection"
cloudflared tunnel info choimory-tunnel
```

## 3. DNS 레코드 생성

```bash
cloudflared tunnel route dns choimory-tunnel choimory.dev
cloudflared tunnel route dns choimory-tunnel "*.choimory.dev"
```

## 4. 접속 확인

```bash
curl -I https://choimory.dev
curl -I https://staging.choimory.dev
curl -I https://staging-member-api.choimory.dev
```

- 브라우저에서도 확인

## 5. 인증 파일 원본 삭제

- cloudflared 연결 확인 후 서버에서 `rm ~/.cloudflared/<TUNNEL_ID>.json`
