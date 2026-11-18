# 목차

- [개요](#개요)
  - [계획, 알아볼것](#계획-알아볼것)
- [계획, 알아볼것 분석](#계획-알아볼것-분석)
  - [답변](#답변)
  - [1. 도메인 구매와 외부 접속 방식](#1-도메인-구매와-외부-접속-방식)
    - [구매처](#구매처)
    - [Tunnel 외 다른 방식](#tunnel-외-다른-방식)
  - [2. Ingress가 없는 이유](#2-ingress가-없는-이유)
  - [3. SSL 인증서](#3-ssl-인증서)
  - [4. DB, Redis Tailscale 전용 접속](#4-db-redis-tailscale-전용-접속)
  - [정리한 순서](#정리한-순서)
- [추가1](#추가1)
- [추가1 분석](#추가1-분석)
  - [답변](#답변-1)
  - [확정 스택](#확정-스택)
    - [가비아 구매 시 할 일](#가비아-구매-시-할-일)
  - [이름 변경](#이름-변경)
    - [Gateway는 Tunnel 전에 변경](#gateway는-tunnel-전에-변경)
    - [ArgoCD](#argocd)
    - [의견](#의견)
  - [결정 필요 사항](#결정-필요-사항)
- [추가2](#추가2)
- [추가2 분석](#추가2-분석)
  - [답변](#답변-2)
  - [변경 대상](#변경-대상)
  - [유지 대상](#유지-대상)
  - [이번 작업에서 새로 만들 것](#이번-작업에서-새로-만들-것)
  - [진행 시점](#진행-시점)
  - [확인 필요 사항](#확인-필요-사항)
- [이름 전체 검수](#이름-전체-검수)
  - [답변](#답변-3)
  - [공용 영역 (choimory-infra)](#공용-영역-choimory-infra)
  - [프로젝트 영역 (choimory-dev)](#프로젝트-영역-choimory-dev)
    - [local-path-choimory-dev](#local-path-choimory-dev)
  - [bootstrap](#bootstrap)
  - [결론](#결론)
- [gateway가 두 개인 이유](#gateway가-두-개인-이유)
  - [답변](#답변-4)
  - [envoy-gateway와 gateway의 역할](#envoy-gateway와-gateway의-역할)
  - [shared-gateway를 gateway로 변경](#shared-gateway를-gateway로-변경)
  - [gateway와 choimory-gateway 비교](#gateway와-choimory-gateway-비교)
- [Application을 gateway-application으로 변경](#application을-gateway-application으로-변경)
  - [답변](#답변-5)
  - [gateway 유지를 추천하는 이유](#gateway-유지를-추천하는-이유)
- [gateway Application의 소속](#gateway-application의-소속)
  - [답변](#답변-6)
  - [Application과 Gateway 리소스 관계](#application과-gateway-리소스-관계)
- [Application을 argocd-gateway로 변경](#application을-argocd-gateway로-변경)
  - [답변](#답변-7)
  - [argocd-gateway의 문제](#argocd-gateway의-문제)
  - [Application 이름 선택지](#application-이름-선택지)
- [argocd gateway 의미 확인](#argocd-gateway-의미-확인)
  - [답변](#답변-8)
  - [같은 기준을 전체에 적용하면](#같은-기준을-전체에-적용하면)
  - [이름 결정 정리](#이름-결정-정리)
- [gateway 이름 전체 확인](#gateway-이름-전체-확인)
  - [답변](#답변-9)
  - [1. Envoy Gateway 컨트롤러 계열](#1-envoy-gateway-컨트롤러-계열)
  - [2. 공용 입구 계열](#2-공용-입구-계열)
  - [3. 표준 이름이라 상관없는 것](#3-표준-이름이라-상관없는-것)
  - [이름 변경과의 관계](#이름-변경과의-관계)
- [이름 변경 범위 확정](#이름-변경-범위-확정)
  - [답변](#답변-10)
  - [변경 계획](#변경-계획)
  - [push 후 동작](#push-후-동작)
  - [시작 전 할 일](#시작-전-할-일)

---

# 개요

- k8s 구축 작업(k8s-setup)으로 클러스터, ArgoCD, Envoy Gateway(`shared-gateway`), 앱별 HTTPRoute까지 구성이 끝난 상태
- HTTPRoute에 도메인(`choimory.dev`, `staging.choimory.dev`, `member-api.choimory.dev`, `staging-member-api.choimory.dev`)은 지정되어 있으나, 외부에서 클러스터로 들어오는 연결(Cloudflare Tunnel)은 아직 구성 전
- 실제 도메인으로 접속했을 때 Cloudflare를 거쳐 클러스터의 Gateway로 요청이 전달되도록 도메인 연동 작업을 진행해야 함
- README.md의 도메인 항목과 `.agents/histories/2026/11/06/k8s-setup` 문서들을 참고해서 진행

## 계획, 알아볼것

1. cloudflare tunnel에 사용할 도메인을 구매한다
   - 도메인 구매처는 어디가 좋은지
   - 원룸 공유기 아래에 있는 미니pc를 모두가 접속하게 하려고 할때 cloudflare tunnel 말고 다른 방식은 어떤 것들이 있는지
2. project, infrastructure ingress에 도메인을 연동한다
   - 현재 gitops에 ingress가 없다
   - ingress를 사용하여 도메인을 연동할 계획이 맞는건지
3. SSL 인증서 적용
   - Lets Encrypt?
4. DB, Redis 외부 접속을 오픈한다 (tailscale 내부망에서만)
   - tailscale operator, tailscale serve, nodeport 각각의 방식 설명, 어느게 좋을지

---

# 계획, 알아볼것 분석

> 14번줄 확인
>
> 1. cloudflare tunnel에 사용할 도메인을 구매한다
>    - 도메인 구매처는 어디가 좋은지
>    - 원룸 공유기 아래에 있는 미니pc를 모두가 접속하게 하려고 할때 cloudflare tunnel 말고 다른 방식은 어떤 것들이 있는지
> 2. project, infrastructure ingress에 도메인을 연동한다
>    - 현재 gitops에 ingress가 없다
>    - ingress를 사용하여 도메인을 연동할 계획이 맞는건지
> 3. SSL 인증서 적용
>    - Lets Encrypt?
> 4. DB, Redis 외부 접속을 오픈한다 (tailscale 내부망에서만)
>    - tailscale operator, tailscale serve, nodeport 각각의 방식 설명, 어느게 좋을지

## 답변

- 도메인은 Cloudflare Registrar, 외부 접속은 Cloudflare Tunnel 추천
- Ingress 대신 이미 Gateway API(Envoy Gateway + HTTPRoute)를 사용 중이므로 Ingress는 불필요
- SSL은 Cloudflare Universal SSL로 처리되므로 Let's Encrypt 불필요
- DB, Redis는 Tailscale Operator 추천

## 1. 도메인 구매와 외부 접속 방식

- README와 HTTPRoute에 `choimory.dev`가 이미 지정되어 있음. 실제 구매 여부 확인 필요

### 구매처

| 구매처 | 특징 |
|---|---|
| **Cloudflare Registrar (추천)** | 원가 판매, 갱신가 동일. DNS가 처음부터 Cloudflare라 Tunnel과 바로 연결 |
| Porkbun, Namecheap | 저렴하지만 네임서버를 Cloudflare로 옮기는 단계가 추가 |
| 가비아 등 국내 | 한국어 지원, `.dev`는 비싼 편 |

- `.dev`는 브라우저가 HTTPS만 허용(HSTS preload)하지만 Cloudflare가 HTTPS를 처리하므로 문제 없음

### Tunnel 외 다른 방식

| 방식 | 원룸 환경 적합성 |
|---|---|
| 공유기 포트포워딩 + DDNS | 공유기 관리자 권한 필요. 원룸은 보통 권한이 없고 사설 IP일 수 있어 어려움. 집 IP 노출 |
| Tailscale Funnel | 인바운드 없이 공개 가능. 기본 `*.ts.net` 도메인이고 개인 도메인 사용이 어려움. 포트 443, 8443, 10000만 가능 |
| VPS 중계 (VPS + WireGuard/frp) | 어디서든 가능. VPS 비용과 관리 서버 추가 |
| ngrok 등 | 개인 도메인은 유료 |
| **Cloudflare Tunnel (추천)** | 무료. 미니PC에서 Cloudflare로 나가는 연결만 사용해 공유기 설정 불필요. IP 숨김, DDoS 방어, Access 연동 |

- 원룸 공유기 아래에서는 Cloudflare Tunnel이 사실상 최선

## 2. Ingress가 없는 이유

- 이 클러스터는 Ingress의 후속 표준인 **Gateway API**를 사용 중이므로 Ingress는 불필요
- 현재 구성
  - `infrastructure/gateway/gateway.yaml`: `shared-gateway` (Envoy, ClusterIP, HTTP 80)
  - `projects/.../base/httproute.yaml`: 앱 도메인을 HTTPRoute로 지정
- 요청 흐름: 브라우저 → Cloudflare → Tunnel → cloudflared Pod → `shared-gateway` Service → HTTPRoute 호스트 매칭 → 앱 Service
- 이번 작업에서 할 일: cloudflared 배포, ArgoCD 등 인프라 도구용 HTTPRoute 추가
- ArgoCD 연결 시 확인할 점
  - Gateway는 `gateway-access: "true"` 라벨이 있는 네임스페이스의 HTTPRoute만 허용. `choimory-infra`에 라벨을 붙일지 다른 방법을 쓸지 결정 필요
  - ArgoCD 서버는 기본적으로 자체 TLS를 사용. Gateway가 HTTP로 전달하므로 `server.insecure: true` 설정 필요

## 3. SSL 인증서

- Cloudflare Tunnel을 쓰면 Let's Encrypt는 불필요

| 구간 | 암호화 |
|---|---|
| 브라우저 → Cloudflare | Cloudflare Universal SSL (무료, 자동 발급·갱신) |
| Cloudflare → cloudflared | Tunnel 연결 자체가 암호화 |
| cloudflared → Gateway → 앱 | 클러스터 내부 HTTP. 현재 Gateway 설정도 이 구조를 전제 |

- Universal SSL은 `choimory.dev`와 `*.choimory.dev`(서브도메인 한 단계)까지만 적용. 그래서 `staging-member-api`처럼 하이픈을 사용
- Let's Encrypt(cert-manager)는 Cloudflare를 거치지 않고 직접 공개할 때만 필요

## 4. DB, Redis Tailscale 전용 접속

| 방식 | 동작 | 장점 | 단점 |
|---|---|---|---|
| **A. Tailscale Operator (추천)** | operator 설치 후 DB Service에 annotation을 붙이면 DB마다 tailnet 기기 생성 (예: `stg-postgres:5432`) | gitops 관리. tailnet에서만 접속. Service IP 변경과 무관 | operator 설치와 OAuth 클라이언트 발급 필요 (SealedSecret 관리). 기기 4개 추가지만 무료 플랜 한도 내 |
| B. `tailscale serve --tcp` | 서버 호스트의 tailscale이 포트를 받아 ClusterIP로 전달 | 설치할 것 없음 | 서버 직접 설정이라 gitops 밖. Service IP가 바뀌면 재설정 필요 |
| C. NodePort | 노드 30000번대 포트로 공개 | 가장 간단 | 같은 공유기의 다른 기기에서도 접속 가능. Tailscale 전용 제한이 어려워 요구사항에 맞지 않음 |

- 요구사항인 Tailscale 전용 접속과 gitops 관리를 둘 다 만족하는 것은 A뿐

## 정리한 순서

1. 도메인 확인 (Cloudflare 등록 여부)
2. cloudflared 배포 + Tunnel
3. 앱 도메인 4개 접속 확인
4. ArgoCD HTTPRoute + Cloudflare Access + 계정 설정
5. Tailscale Operator로 DB 접속

---

# 추가1

- 스택 및 방향 설정
- 도메인: 가비아
- 호스팅: cloudflare tunnel
- 인증서: cloudflare universal ssl
- 내부 접속: tailscale operator
- 그 외: infra 리소스들 이름 변경
  - 홈서버에 하나만 놓고 여러 프로젝트들이 같이 사용하는 공용 자원들은 다 choimory를 앞에 붙임
  - 이를테면 gateway 이름이 shared-gateway로 되어있는데 choimory-gateway로
  - argocd도 choimory-argocd
  - prometheus, grafana는 프로젝트별로 만든다면 프로젝트명-grafana, prometheus겠지만, 홈서버 전체에 하나만 놓는다면 choimory-grafana, prometheus가 될거임

---

# 추가1 분석

> - 스택 및 방향 설정
> - 도메인: 가비아
> - 호스팅: cloudflare tunnel
> - 인증서: cloudflare universal ssl
> - 내부 접속: tailscale operator
> - 그 외: infra 리소스들 이름 변경
>   - 홈서버에 하나만 놓고 여러 프로젝트들이 같이 사용하는 공용 자원들은 다 choimory를 앞에 붙임
>   - 이를테면 gateway 이름이 shared-gateway로 되어있는데 choimory-gateway로
>   - argocd도 choimory-argocd
>   - prometheus, grafana는 프로젝트별로 만든다면 프로젝트명-grafana, prometheus겠지만, 홈서버 전체에 하나만 놓는다면 choimory-grafana, prometheus가 될거임

## 답변

- 스택은 그대로 확정. 가비아 구매 시 네임서버를 Cloudflare로 변경 필요
- 이름 변경은 대상별 비용 차이가 커서, 다른 프로젝트가 참조하는 Gateway와 신규 리소스만 `choimory-` 접두어를 붙이는 것을 추천

## 확정 스택

| 항목 | 결정 | 메모 |
|---|---|---|
| 도메인 | 가비아 | 네임서버를 Cloudflare로 변경 필요 |
| 외부 접속 | Cloudflare Tunnel | |
| 인증서 | Cloudflare Universal SSL | |
| 내부 접속 | Tailscale Operator | OAuth 클라이언트 발급 필요 |

### 가비아 구매 시 할 일

- Tunnel은 Cloudflare가 관리하는 DNS에만 연결되므로, 등록은 가비아에서 하고 DNS는 Cloudflare로 넘김

1. Cloudflare에 사이트(`choimory.dev`) 추가 (무료 플랜) → 네임서버 2개 발급
2. 가비아 도메인 관리에서 네임서버를 Cloudflare 것으로 변경
3. Cloudflare에서 Active 상태 확인 (보통 수 분~수 시간)

- 가비아에서 DNSSEC를 켜 두었다면 변경 전에 끄기

## 이름 변경

| 대상 | 현재 | 변경 비용 | 의견 |
|---|---|---|---|
| Gateway | `shared-gateway` | 낮음. `gateway.yaml`과 HTTPRoute 2개 수정 | 추천 |
| GatewayClass, EnvoyProxy | `envoy`, `envoy-proxy` | 낮음 | Gateway와 같이 바꾸면 일관성 있음 |
| cloudflared, Tailscale Operator (신규) | - | 없음 | 처음부터 `choimory-` 접두어로 생성 |
| ArgoCD Application 이름 | `argocd` | 중간 | 아래 참고 |
| ArgoCD 리소스 이름 (Helm 릴리스) | `argocd-server` 등 | 높음 | 비추천 |
| sealed-secrets, cnpg, metrics-server 등 | 차트 기본 이름 | 대상마다 다름 | 범위 결정 필요 |

### Gateway는 Tunnel 전에 변경

- Envoy가 만드는 Service 이름에 Gateway 이름이 들어감 (`envoy-choimory-infra-<gateway이름>-<해시>`)
- cloudflared가 이 Service를 바라보므로 Tunnel 전에 바꿔야 두 번 작업하지 않음
- 아직 외부 트래픽이 없어 지금 바꿔도 영향 없음

### ArgoCD

- Application 이름만 변경 (`argocd` → `choimory-argocd`)
  - Application 이름 변경은 삭제 후 재생성과 같음
  - prune과 finalizer 때문에 ArgoCD 자신이 지워질 수 있어 plan10처럼 finalizer를 먼저 제거해야 함
  - 가능하지만 조심해서 진행해야 함
- 리소스 이름까지 변경 (`choimory-argocd-server` 등)
  - Helm 릴리스 이름이 바뀌어 사실상 재설치. plan10의 네임스페이스 변경과 비슷한 규모
  - Secret, 설정, CRD 소유권도 다시 정리해야 함

### 의견

- 네임스페이스 `choimory-infra`가 이미 홈서버 공용이라는 구분을 하고 있음
- 다른 프로젝트가 이름으로 참조하는 것(Gateway: 각 프로젝트 HTTPRoute가 참조)과 새로 만드는 것(cloudflared, Tailscale Operator)만 `choimory-` 접두어 추천
- 차트로 설치한 도구(argocd, sealed-secrets 등)는 차트 기본 이름 유지 추천. 공식 문서와 명령어 예시가 기본 이름(`argocd-server` 등)을 전제로 함
- Grafana, Prometheus는 설치할 때 결정. 홈서버 전체에 하나만 둔다면 `choimory-grafana` 방식에 동의

## 결정 필요 사항

1. 도메인을 이미 보유 중인지, 지금 구매해야 하는지
2. 이름 변경 범위
   - A: Gateway 계열 + 신규 리소스만 (추천)
   - B: A + ArgoCD Application 이름
   - C: 리소스 이름 전체
3. ArgoCD 도메인 이름 (예: `argocd.choimory.dev`)

---

# 추가2

- 이름변경은 그럼 임의로 붙인 이름을 대상으로만 진행하자
  - 이를테면 argocd, argocd-server, cloudnativepg 같은건 기본이름이니까 변경 x 
  - shared-gateway 이건 임의로 붙인 이름이니까 shared 같은거 붙일바에 choimory를 붙이는걸 원함

---

# 추가2 분석

> - 이름변경은 그럼 임의로 붙인 이름을 대상으로만 진행하자
>   - 이를테면 argocd, argocd-server, cloudnativepg 같은건 기본이름이니까 변경 x 
>   - shared-gateway 이건 임의로 붙인 이름이니까 shared 같은거 붙일바에 choimory를 붙이는걸 원함

## 답변

- 차트, 도구 기본 이름은 유지하고 임의로 붙인 이름만 `choimory-` 접두어로 변경
- 저장소 전체를 확인해 대상 정리

## 변경 대상

| 대상 | 종류 | 현재 | 변경 | 참조하는 곳 |
|---|---|---|---|---|
| 공용 Gateway | Gateway | `shared-gateway` | `choimory-gateway` | `gateway.yaml`, front·member-api `httproute.yaml`, README |
| Gateway 종류 | GatewayClass | `envoy` | `choimory-envoy` | `gateway.yaml` |
| Envoy 실행 설정 | EnvoyProxy | `envoy-proxy` | `choimory-envoy-proxy` | `gateway.yaml` (GatewayClass가 참조) |
| Gateway Application | ArgoCD Application | `gateway` | `choimory-gateway` | `infrastructure/apps/gateway.yaml` |

- GatewayClass, EnvoyProxy도 임의 이름. Gateway를 바꾸면 어차피 다시 만들어지므로 함께 바꿔도 추가 비용 없음
- Application `gateway`도 임의 이름
  - 이름을 바꾸면 기존 Application 삭제 후 새로 생성되며, 그 안의 Gateway도 지워졌다가 다시 생성됨
  - Gateway 이름을 바꾸는 시점이고 외부 트래픽도 없어 영향 없음
  - 다른 공용 도구 Application(`argocd`, `cloudnative-pg` 등)은 차트 이름과 같아 유지

## 유지 대상

| 대상 | 현재 | 유지 이유 |
|---|---|---|
| argocd, cloudnative-pg, envoy-gateway, metrics-server, local-path-provisioner | 차트·공식 배포 기본 이름 | 기본 이름 |
| `sealed-secrets-controller` | kubeseal 기본 컨트롤러 이름 | 바꾸면 kubeseal 명령마다 `--controller-name` 필요 |
| StorageClass `local-path` | 공식 기본 | 기본 이름 |
| StorageClass `local-path-choimory-dev` | 프로젝트 전용 | 공용 자원이 아니며 이미 프로젝트명 포함 |
| 네임스페이스 `choimory-infra`, `choimory-dev-*` | - | 이미 규칙에 맞음 |
| root Application `root` | - | bootstrap 진입점. 바꾸려면 서버에서 수동 재등록 필요. 공용 자원보다는 클러스터 진입점 |
| 라벨 `gateway-access: "true"` | - | 이름이 아닌 허용 표시라 유지해도 될 것으로 봄 |

## 이번 작업에서 새로 만들 것

| 대상 | 이름 |
|---|---|
| cloudflared Deployment / Application | `choimory-cloudflared` |
| Cloudflare Tunnel (대시보드) | `choimory-tunnel` |
| Tunnel 토큰 SealedSecret | `choimory-cloudflared-token` |
| Tailscale Operator | 차트 기본 이름 유지 (`operator`) |
| Tailscale 기기 이름 (DB) | 프로젝트 자원이므로 `choimory-dev-stg-postgres` 형식 |

## 진행 시점

- Envoy Service 이름에 Gateway 이름이 들어가므로 Gateway 이름 변경을 cloudflared 배포보다 먼저 진행
- 한 커밋으로 묶음: `gateway.yaml`, HTTPRoute 2개, `apps/gateway.yaml`, README

## 확인 필요 사항

1. GatewayClass, EnvoyProxy, `gateway` Application까지 함께 변경할지
2. `root` Application과 `gateway-access` 라벨 유지 여부
3. 도메인 보유 여부와 ArgoCD 도메인 이름

---

# 이름 전체 검수

> 이름 한번 싹 검수해봐

## 답변

- bootstrap, infrastructure, projects의 리소스 이름 54개를 모두 확인
- 바꿀 대상은 추가2에서 정리한 Gateway 관련 4개뿐이며 새로 나온 대상은 없음

## 공용 영역 (choimory-infra)

| 이름 | 종류 | 구분 | 판정 |
|---|---|---|---|
| `choimory-infra` | Namespace | 임의 | 규칙에 맞음 |
| `argocd`, `cloudnative-pg`, `envoy-gateway`, `local-path-provisioner`, `metrics-server`, `sealed-secrets` | Application | 차트·도구 이름 | 유지 |
| `sealed-secrets-controller` | Helm fullnameOverride | kubeseal 기본값 | 유지 |
| `local-path` | StorageClass | 공식 기본 | 유지 |
| `shared-gateway` | Gateway | 임의 | `choimory-gateway`로 변경 |
| `envoy` | GatewayClass | 임의 | `choimory-envoy`로 변경 |
| `envoy-proxy` | EnvoyProxy | 임의 | `choimory-envoy-proxy`로 변경 |
| `gateway` | Application | 임의 | `choimory-gateway`로 변경 |
| `root` | Application | 임의 | 유지 추천. bootstrap 진입점이라 바꾸려면 서버에서 수동 재등록 필요 |
| `gateway-access: "true"` | Namespace 라벨 | 임의 | 유지 추천. 리소스 이름이 아닌 허용 표시 |

## 프로젝트 영역 (choimory-dev)

| 이름 | 종류 | 판정 |
|---|---|---|
| `choimory-dev-front`, `choimory-dev-member-api` | Deployment, Service, HTTPRoute, 컨테이너, `app` 라벨 | 일관됨 |
| `choimory-dev-postgres`, `choimory-dev-redis` | CNPG Cluster, StatefulSet, Service | 일관됨 |
| `choimory-dev-member-api-secret`, `choimory-dev-redis-secret` | SealedSecret | 일관됨 |
| `choimory-dev` | AppProject, Application, 네임스페이스 base | 일관됨 |
| `choimory-dev-apps`, `choimory-dev-protected` | ApplicationSet | 일관됨 |
| `choimory-dev-{env}-{component}` | 생성되는 Application | 일관됨 |
| `memory-quota`, `default-limits`, `allow-same-namespace-and-infra` | ResourceQuota, LimitRange, NetworkPolicy | 유지. 프로젝트 네임스페이스 안에서만 쓰는 설정이라 일반 이름으로 충분 |
| `local-path-choimory-dev` | StorageClass | 아래 참고 |

### local-path-choimory-dev

- 다른 프로젝트 리소스는 `choimory-dev-*`처럼 프로젝트명이 앞에 오는데, 이것만 뒤에 붙음
- 유지 추천
  - PVC의 `storageClassName`은 생성 후 변경 불가. 이름을 바꾸면 stg·prod의 postgres, redis PVC를 새로 만들고 데이터를 옮겨야 함
  - `local-path-` 접두어가 저장 방식을 묶어 주는 의미도 있음

## bootstrap

| 이름 | 판정 |
|---|---|
| kubeadm `clusterName` (기본 `kubernetes`) | 기본값이라 유지 |
| certSANs `choimory-server` | 서버 호스트명 |
| Cilium values | 임의 이름 없음 |

## 결론

- 변경 대상: Gateway, GatewayClass, EnvoyProxy, `gateway` Application (추가2와 동일)
- 함께 수정: HTTPRoute 2개의 `parentRefs`, README의 `shared-gateway` 2곳
- 유지 확인 필요: `root`, `gateway-access`, `local-path-choimory-dev` (셋 다 유지 추천)

---

# gateway가 두 개인 이유

> envoy 이런건 그냥 그대로 둬도 될거같음 shared-gateway를 그냥 gateway로 바꾸면 될거같은데 gateway가 두개야?

## 답변

- 이름에 gateway가 들어간 Application이 두 개 있으며 역할이 다름
- `envoy-gateway`는 컨트롤러, `gateway`는 그 컨트롤러로 만든 공용 입구

## envoy-gateway와 gateway의 역할

| Application | 설치하는 것 | 역할 | 비유 |
|---|---|---|---|
| `envoy-gateway` | Envoy Gateway 컨트롤러 (Helm 차트) | Gateway 리소스를 감시하다가 Envoy 프록시 Pod와 Service를 만들어 주는 관리자 | nginx를 설치·관리하는 프로그램 |
| `gateway` | `infrastructure/gateway/gateway.yaml` (EnvoyProxy, GatewayClass, `shared-gateway`) | 실제로 요청을 받는 입구 | 그 프로그램으로 띄운 nginx 서버 하나와 설정 |

- Gateway 같은 리소스 종류(CRD)는 컨트롤러 차트가 설치되어야 생기므로 따로 설치해야 함
- sync wave도 `envoy-gateway`가 먼저, `gateway`가 6번째

## shared-gateway를 gateway로 변경

- 가능. Application `gateway`와 Gateway 리소스 `gateway`의 이름이 같아지지만 종류가 달라 충돌하지 않음
- Envoy Service 이름이 `envoy-choimory-infra-shared-gateway-<해시>` → `envoy-choimory-infra-gateway-<해시>`로 바뀜. cloudflared가 이 Service를 가리키므로 Tunnel 전에 변경
- 함께 수정: `gateway.yaml`, HTTPRoute 2개의 `parentRefs`, README 2곳

## gateway와 choimory-gateway 비교

- 추가1에서는 `shared` 대신 `choimory` 접두어를 원했으나, 이번엔 접두어 없는 `gateway`
- `gateway`: 네임스페이스 `choimory-infra`가 이미 공용임을 나타내므로 충분. GatewayClass `envoy`, EnvoyProxy `envoy-proxy`와 결이 맞음
- `choimory-gateway`: 다른 프로젝트 HTTPRoute에 `parentRefs: choimory-gateway`로 적혀 어느 Gateway인지 더 분명함
- 둘 다 괜찮음

---

# Application을 gateway-application으로 변경

> gateway, gateway-application 이렇게 바꾸면 될거같은데

## 답변

| 대상 | 현재 | 변경 |
|---|---|---|
| Gateway 리소스 | `shared-gateway` | `gateway` |
| ArgoCD Application | `gateway` | `gateway-application` |

- Gateway 리소스 → `gateway`: 문제없음
- Application → `gateway-application`: 가능하지만 `gateway` 유지 추천

## gateway 유지를 추천하는 이유

1. 다른 Application과 이름 규칙이 어긋남
   - `root`가 관리하는 Application은 모두 설치 대상 이름만 사용 (`argocd`, `envoy-gateway`, `sealed-secrets`, `cloudnative-pg`, `choimory-dev`)
   - 일관성을 맞추려면 전부 `-application`을 붙여야 하는데, Application 이름 변경은 하나하나가 삭제 후 재생성이라 비용이 큼
2. 같은 이름이어도 헷갈릴 일이 적음
   - 종류가 달라 충돌하지 않고 조회도 따로 함 (`kubectl get gateway`, `kubectl get application`)
   - 이미 `choimory-dev`가 AppProject, Application, 네임스페이스 base에서 같은 이름으로 쓰이고 있음

- 변경하더라도 외부 트래픽이 없어 영향은 없음. `gateway.yaml`, HTTPRoute 2개, `apps/gateway.yaml`, README를 한 커밋으로 묶음

---

# gateway Application의 소속

> 아 gateway라는 이름의 application은 argocd꺼야?

## 답변

- 맞음. ArgoCD의 Application은 "Git의 이 경로를 클러스터의 이 네임스페이스에 배포하라"는 배포 단위 등록 정보

| 이름 | 종류 | 누구 것 | 하는 일 |
|---|---|---|---|
| `gateway` | Application (`argoproj.io`) | ArgoCD | `infrastructure/gateway` 폴더를 `choimory-infra`에 배포하고 Git과 계속 맞추라는 ArgoCD의 등록 정보 |
| `shared-gateway` | Gateway (`gateway.networking.k8s.io`) | Kubernetes Gateway API | 실제로 요청을 받는 입구. Envoy Gateway가 보고 프록시 Pod를 띄움 |

## Application과 Gateway 리소스 관계

```
root (Application)                      ← infrastructure/apps 폴더를 배포
 └─ gateway (Application)               ← infrastructure/gateway 폴더를 배포
     ├─ envoy-proxy (EnvoyProxy)
     ├─ envoy (GatewayClass)
     └─ shared-gateway (Gateway)        ← 실제 입구
```

- `infrastructure/apps/`의 파일들(`argocd.yaml`, `gateway.yaml`, `envoy-gateway.yaml` 등)은 모두 ArgoCD Application
- 각 Application의 이름은 배포하는 폴더나 차트 이름을 따름. ArgoCD 화면의 카드 하나하나가 Application
- Gateway 리소스를 `gateway`로 바꾸면 "`gateway` 앱 안에 `gateway` Gateway"가 되며, `choimory-dev` 앱 안에 `choimory-dev` 네임스페이스 base가 있는 것과 같은 형태

---

# Application을 argocd-gateway로 변경

> 그럼 shared-gateway는 gateway, gateway는 argocd-gateway로

## 답변

- Gateway 리소스는 `gateway`로 확정
- Application 이름 `argocd-gateway`는 오해를 부를 수 있어 재고 추천

## argocd-gateway의 문제

1. ArgoCD 전용 입구로 읽힘
   - 이번 작업에서 ArgoCD 도메인용 HTTPRoute를 만들 예정이라 "ArgoCD 화면 접속용 Gateway"로 착각하기 쉬움
   - 실제로는 front, member-api, ArgoCD가 모두 같이 쓰는 공용 입구
2. 다른 Application과 규칙이 어긋남
   - 다른 Application은 `argocd`, `envoy-gateway`, `sealed-secrets`처럼 접두어가 없음
   - Application은 모두 ArgoCD 것이라 `argocd-`를 붙여도 구분 효과가 없음

## Application 이름 선택지

| Application 이름 | 의미 | 의견 |
|---|---|---|
| `gateway` (유지) | `infrastructure/gateway` 폴더와 같은 이름 | 추천. 변경 작업도 없음 |
| `argocd-gateway` | - | 비추천 |
| `shared-gateway`처럼 역할을 드러내는 이름 | - | 피하기로 한 `shared` 계열이라 제외 |

- 화면과 명령어에서 구분됨: ArgoCD 화면에서 Application 카드는 `gateway`, 안의 리소스는 `Gateway/gateway`. 명령어는 `kubectl get application gateway`, `kubectl get gateway gateway`

---

# argocd gateway 의미 확인

> 이름: gateway
>   종류: Application (argoproj.io)
>   누구 것: ArgoCD            
>   하는 일: "infrastructure/gateway 폴더를 choimory-infra에 배포하고 Git과
>   계속 맞춰라"라는 ArgoCD의 등록 정보 
>
> argocd gateway 아니야?

## 답변

- 맞음. "gateway를 배포하는 ArgoCD Application"
- 다만 그 기준이면 다른 Application들도 모두 ArgoCD Application

## 같은 기준을 전체에 적용하면

| Application | 정체 | 같은 기준으로 지으면 |
|---|---|---|
| `gateway` | gateway를 배포하는 ArgoCD Application | `argocd-gateway` |
| `envoy-gateway` | envoy-gateway를 배포하는 ArgoCD Application | `argocd-envoy-gateway` |
| `sealed-secrets` | sealed-secrets를 배포하는 ArgoCD Application | `argocd-sealed-secrets` |
| `argocd` | ArgoCD 자신을 배포하는 ArgoCD Application | `argocd-argocd` |
| `choimory-dev-stg-front` | front를 배포하는 ArgoCD Application | `argocd-choimory-dev-stg-front` |

- Application 이름은 ArgoCD 안에서만 쓰이므로 "누구 것인지"는 이미 정해져 있음. "무엇을 배포하는지"를 이름으로 쓰는 것이 ArgoCD의 일반적인 방식이며 지금 저장소도 그렇게 되어 있음
- `gateway` 하나만 `argocd-gateway`로 바꾸면 한 줄만 다른 규칙이 됨. 전부 바꾸면 Application 10여 개를 삭제 후 재생성해야 해서 비추천

## 이름 결정 정리

| 대상 | 결정 |
|---|---|
| Gateway 리소스 `shared-gateway` | `gateway`로 변경 |
| Application `gateway` | 유지 (추천) |
| GatewayClass `envoy`, EnvoyProxy `envoy-proxy` | 유지 |

---

# gateway 이름 전체 확인

> 확인해보니까 gateway가 두개인게 하나는 shared-gateway고 하나는 envoy-gateway같은데 더 있어?

## 답변

- 저장소와 클러스터를 합치면 더 있지만 모두 두 개에서 파생된 것
- "Envoy Gateway 컨트롤러 계열"과 "공용 입구 계열" 두 묶음

## 1. Envoy Gateway 컨트롤러 계열

| 이름 | 종류 | 어디 | 만든 주체 |
|---|---|---|---|
| `envoy-gateway` | Application | `infrastructure/apps/envoy-gateway.yaml` | 저장소 |
| `infrastructure/envoy-gateway/` | 디렉토리 (Helm values) | 저장소 | 저장소 |
| `envoy-gateway` | Deployment, Service 등 | 클러스터 `choimory-infra` | Helm 차트 (기본 이름) |

## 2. 공용 입구 계열

| 이름 | 종류 | 어디 | 만든 주체 |
|---|---|---|---|
| `gateway` | Application | `infrastructure/apps/gateway.yaml` | 저장소 |
| `infrastructure/gateway/` | 디렉토리 | 저장소 | 저장소 |
| `shared-gateway` | Gateway | `infrastructure/gateway/gateway.yaml` | 저장소 |
| `envoy-choimory-infra-shared-gateway-<해시>` | Envoy 프록시 Deployment, Service | 클러스터 `choimory-infra` | Envoy Gateway 자동 생성 |
| `gateway-access` | 네임스페이스 라벨 | `projects/.../namespace.yaml` | 저장소 |

## 3. 표준 이름이라 상관없는 것

- Gateway API CRD: `gateways.gateway.networking.k8s.io`, `httproutes.gateway.networking.k8s.io` 등
- HTTPRoute의 `parentRefs`: 리소스 이름이 아니라 `shared-gateway`를 가리키는 참조

## 이름 변경과의 관계

- `shared-gateway`를 `gateway`로 바꾸면 2번 묶음의 Envoy 프록시 이름(`envoy-choimory-infra-gateway-<해시>`, 자동)과 HTTPRoute 2개의 `parentRefs`(직접 수정)가 따라 바뀜
- 1번 묶음(`envoy-gateway`)은 차트 기본 이름이라 유지
- 클러스터 확인: `kubectl -n choimory-infra get all,gateway,gatewayclass,envoyproxy | grep -i gateway`

---

# 이름 변경 범위 확정

> 그냥 shared-gateway만 gateway로 이름 바꿔주면 될듯

## 답변

| 대상 | 결정 |
|---|---|
| Gateway `shared-gateway` | `gateway`로 변경 |
| Application `gateway`, `envoy-gateway` | 유지 |
| GatewayClass `envoy`, EnvoyProxy `envoy-proxy` | 유지 |
| `root`, `gateway-access` 라벨, `local-path-choimory-dev` | 유지 |

## 변경 계획

| 파일 | 변경 |
|---|---|
| `infrastructure/gateway/gateway.yaml` | `metadata.name: shared-gateway` → `gateway` |
| `projects/choimory-dev/choimory-dev-front/base/httproute.yaml` | `parentRefs` → `gateway` |
| `projects/choimory-dev/choimory-dev-member-api/base/httproute.yaml` | `parentRefs` → `gateway` |
| `README.md` | `shared-gateway` 2곳 → `gateway` |

## push 후 동작

1. ArgoCD가 새 Gateway `gateway`를 만들고 옛 `shared-gateway`는 prune으로 삭제
2. Envoy Gateway가 프록시를 `envoy-choimory-infra-gateway-<해시>`로 새로 생성
3. HTTPRoute가 새 Gateway에 연결

- 외부 트래픽이 없어 영향 없음. push 후 서버에서 `kubectl -n choimory-infra get gateway`와 HTTPRoute 상태 확인

## 시작 전 할 일

- calendar를 `TASK:도메인 연동 작업(domain-connect)`로 변경
- 작업 기록은 `task1.md`
- 커밋과 push는 직접 진행
