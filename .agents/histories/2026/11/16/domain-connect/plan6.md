# 목차

- [개요](#개요)
- [5단계 계획 (DB, Redis Tailscale 접속)](#5단계-계획-db-redis-tailscale-접속)
  - [답변](#답변)
  - [목표](#목표)
  - [현재 구성](#현재-구성)
  - [접속 경로](#접속-경로)
  - [노출 방식](#노출-방식)
    - [PostgreSQL](#postgresql)
    - [Redis](#redis)
  - [제안 값](#제안-값)
  - [작업 단계](#작업-단계)
    - [A. 결정](#a-결정)
    - [B. gitops 작성](#b-gitops-작성)
    - [C. 확인](#c-확인)
    - [D. 클라이언트 접속](#d-클라이언트-접속)
    - [E. 접근 제한 (선택)](#e-접근-제한-선택)
  - [보안](#보안)
  - [주의할 점과 사전 확인](#주의할-점과-사전-확인)
  - [결정 필요 사항](#결정-필요-사항)
- [추가1](#추가1)
- [추가1 분석](#추가1-분석)
  - [답변](#답변-1)
  - [1. 이름](#1-이름)
  - [2. push](#2-push)
  - [3. 노출 방식 설명](#3-노출-방식-설명)
    - [전제: Service와 Tailscale 표시](#전제-service와-tailscale-표시)
    - [PostgreSQL 방식 비교](#postgresql-방식-비교)
    - [Redis 방식 비교](#redis-방식-비교)
  - [4. 접근 제한 설명](#4-접근-제한-설명)
    - [지금 상태](#지금-상태)
    - [제한하면 어떻게 되나](#제한하면-어떻게-되나)
  - [결정 필요 사항 (추가1)](#결정-필요-사항-추가1)
- [Service를 만든다는 의미](#service를-만든다는-의미)
  - [답변](#답변-2)
  - [Redis (R1): Service 매니페스트 추가](#redis-r1-service-매니페스트-추가)
  - [PostgreSQL (P1): Cluster 설정에 추가](#postgresql-p1-cluster-설정에-추가)
  - [정리 (Service 생성 주체)](#정리-service-생성-주체)
- [Redis, PostgreSQL에 Service, Deployment 파일이 없는 이유](#redis-postgresql에-service-deployment-파일이-없는-이유)
  - [답변](#답변-3)
  - [앱 (front, member-api)](#앱-front-member-api)
  - [Redis: 한 파일에 Service와 StatefulSet](#redis-한-파일에-service와-statefulset)
  - [PostgreSQL: CNPG Cluster 하나](#postgresql-cnpg-cluster-하나)
  - [정리 (리소스 작성 방식)](#정리-리소스-작성-방식)
- [추가2](#추가2)
- [추가2 분석](#추가2-분석)
  - [답변](#답변-4)
  - [1. 확정 사항](#1-확정-사항)
  - [2. 접근 제한 설명](#2-접근-제한-설명)
    - [DB까지 거치는 잠금](#db까지-거치는-잠금)
    - [grants 제한이 의미 있는 경우](#grants-제한이-의미-있는-경우)
    - [좁힌다면 이런 모양](#좁힌다면-이런-모양)
    - [바꾸기 전에 주의할 것](#바꾸기-전에-주의할-것)
    - [추천](#추천)
  - [결정 필요 사항 (추가2)](#결정-필요-사항-추가2)
- [접근 제한 요약 확인](#접근-제한-요약-확인)
  - [답변](#답변-5)
  - [맞는 부분](#맞는-부분)
  - [보완할 부분](#보완할-부분)
  - [한 줄 요약](#한-줄-요약)
- [추가3](#추가3)
- [추가3 분석](#추가3-분석)
  - [답변](#답변-6)
  - [1. Redis 파일 분리](#1-redis-파일-분리)
  - [2. 접근 제한 유지](#2-접근-제한-유지)
  - [5단계 최종 작업 내용](#5단계-최종-작업-내용)
- [5단계 최종 작업계획](#5단계-최종-작업계획)
  - [답변](#답변-7)
  - [사전 확인 결과 (CNPG 1.30)](#사전-확인-결과-cnpg-130)
  - [확정 값](#확정-값)
  - [작업 단계 (최종)](#작업-단계-최종)
    - [A. gitops 작성 (최종)](#a-gitops-작성-최종)
    - [B. 검증 (최종)](#b-검증-최종)
    - [C. push (최종)](#c-push-최종)
    - [D. 확인 (최종)](#d-확인-최종)
    - [E. 접속 확인 (최종)](#e-접속-확인-최종)
  - [주의할 점 (최종)](#주의할-점-최종)
  - [기록](#기록)

---

# 개요

- 4단계(ArgoCD 내부 접속, 계정 정리) 완료. task4 참고
- 5단계는 stg, prod의 PostgreSQL, Redis를 Tailscale 전용으로 열어 SSH 터널, port-forward 없이 접속하는 작업
- 4단계에서 설치한 Tailscale Operator(`choimory-k8s-operator`)를 재사용. Tailscale 관리 화면 작업(태그, OAuth)은 추가로 필요 없음
- 참고 문서: [plan1.md](./plan1.md)(Tailscale Operator 선택 이유), [plan5.md](./plan5.md)(4단계 최종 작업계획), [task4.md](./task4.md)(Operator 설치와 infra-gateway 노출)

---

# 5단계 계획 (DB, Redis Tailscale 접속)

> 5단계 계획 plan6에 작성해줘

## 답변

- DB와 Redis는 HTTP가 아니라 TCP라 `infra-gateway`(HTTPS)를 거치지 않음
- `infra-gateway`처럼 대상마다 Tailscale 전용 Service를 하나씩 만들어 tailnet 기기 4개로 노출
- gitops 변경은 프로젝트 영역(`projects/choimory-dev/postgres`, `redis`)에만 생기고, 인프라 영역은 그대로

## 목표

| 대상 | stg | prod |
|---|---|---|
| PostgreSQL | tailnet에서 접속 | tailnet에서 접속 (조회 위주, 클라이언트 Read-only) |
| Redis | tailnet에서 접속 | tailnet에서 접속 |

- Mac DataGrip 등에서 SSH 터널 없이 접속
- 인터넷, 공유기 LAN에는 열지 않음

## 현재 구성

| 대상 | 리소스 | 접속 정보 |
|---|---|---|
| PostgreSQL | CloudNativePG `Cluster` `choimory-dev-postgres` (인스턴스 1개). CNPG가 Service `choimory-dev-postgres-rw`(쓰기, primary), `-ro`, `-r` 자동 생성 | DB, 사용자 `member`. 비밀번호는 CNPG가 만든 Secret `choimory-dev-postgres-app` |
| Redis | StatefulSet `choimory-dev-redis` + Service `choimory-dev-redis` (ClusterIP 6379) | 비밀번호는 SealedSecret `choimory-dev-redis-secret` |

- 두 대상 모두 ApplicationSet `choimory-dev-protected` 소속 (prune 꺼짐, 삭제 시에도 리소스 유지)
- CloudNativePG 차트 `0.29.1` = 오퍼레이터 `1.30.0`
- 현재 접속: 서버 port-forward + Mac SSH 터널

## 접속 경로

```
내 기기(Tailscale) → choimory-dev-{env}-{postgres|redis} (tailnet 기기, 100.x)
  → Tailscale 프록시 Pod (choimory-infra)
  → Tailscale 전용 Service (choimory-dev-{env})
  → PostgreSQL / Redis Pod
```

- 프록시 Pod는 Operator 네임스페이스(`choimory-infra`)에 생김
- 프로젝트 NetworkPolicy가 `choimory-infra`에서 오는 요청을 이미 허용하므로 NetworkPolicy 변경 불필요
- 앱(member-api)이 쓰는 기존 Service는 건드리지 않음

## 노출 방식

### PostgreSQL

| 방식 | 내용 | 장점 | 단점 |
|---|---|---|---|
| P1. CNPG 추가 Service (추천) | `Cluster`의 `spec.managed.services.additional`로 rw 대상 Service를 하나 더 정의 (`LoadBalancer` + `loadBalancerClass: tailscale` + 기기 이름 annotation) | CNPG가 primary를 따라가며 관리. `infra-gateway`와 같은 방식. `Cluster` 하나에 설정이 모임 | CNPG와 Operator가 같은 Service를 만짐. 충돌 여부 확인 필요 (아래 사전 확인) |
| P2. 기존 `-rw` Service에 annotation | `choimory-dev-postgres-rw`에 `tailscale.com/expose` annotation | 리소스 추가 없음 | CNPG가 관리하는 Service라 annotation이 되돌려질 수 있음. 앱이 쓰는 Service를 건드림 |
| P3. Service 직접 작성 | CNPG Pod 라벨(`cnpg.io/cluster`, primary 역할 라벨)로 selector를 잡은 Service 별도 작성 | CNPG와 무관하게 동작 | CNPG 라벨 이름에 의존 (버전 변경 시 확인 필요) |

- P1으로 진행, 사전 확인에서 문제가 있으면 P3

### Redis

| 방식 | 내용 | 장점 | 단점 |
|---|---|---|---|
| R1. 전용 Service 추가 (추천) | 기존 Service와 같은 selector로 `LoadBalancer` + `loadBalancerClass: tailscale` Service 추가 | PostgreSQL, `infra-gateway`와 방식 통일. 앱이 쓰는 Service는 그대로 | Service 하나 추가 |
| R2. 기존 Service에 annotation | `choimory-dev-redis`에 `tailscale.com/expose: "true"` | 가장 간단 | 앱이 쓰는 Service를 건드림. PostgreSQL과 방식이 다름 |

## 제안 값

| 항목 | stg | prod |
|---|---|---|
| PostgreSQL 추가 Service | `choimory-dev-postgres-tailnet` | `choimory-dev-postgres-tailnet` |
| PostgreSQL tailnet 기기 | `choimory-dev-stg-postgres` | `choimory-dev-prod-postgres` |
| Redis 추가 Service | `choimory-dev-redis-tailnet` | `choimory-dev-redis-tailnet` |
| Redis tailnet 기기 | `choimory-dev-stg-redis` | `choimory-dev-prod-redis` |
| 접속 주소 (MagicDNS) | `choimory-dev-stg-postgres:5432`, `choimory-dev-stg-redis:6379` | `choimory-dev-prod-postgres:5432`, `choimory-dev-prod-redis:6379` |

- 기기 이름은 네임스페이스 이름(`choimory-dev-stg`) 규칙을 따름. 도메인이 아니라 tailnet 기기 이름이라 `staging-` 규칙과는 별개
- 기기 태그는 Operator 기본값 `tag:k8s`
- 공통 부분은 base, 기기 이름은 환경마다 달라 overlay에서 지정

## 작업 단계

### A. 결정

- 제안 값, 노출 방식(P1, R1) 확정
- Tailscale 관리 화면 작업 없음 (`tag:k8s` 소유자, OAuth 이미 준비됨)

### B. gitops 작성

| 파일 | 내용 |
|---|---|
| `projects/choimory-dev/postgres/base/postgres.yaml` | `spec.managed.services.additional`에 rw 대상 `choimory-dev-postgres-tailnet` (`LoadBalancer`, `loadBalancerClass: tailscale`) |
| `projects/choimory-dev/postgres/overlays/{stg,prod}/kustomization.yaml` | 기기 이름 annotation `tailscale.com/hostname` 환경별 지정 |
| `projects/choimory-dev/redis/base/redis.yaml` | `choimory-dev-redis-tailnet` Service 추가 |
| `projects/choimory-dev/redis/overlays/{stg,prod}/kustomization.yaml` | 기기 이름 annotation 환경별 지정 |
| `README.md` | 관리 화면과 DB 접속 표를 Tailscale 접속으로 변경, port-forward는 비상용 |

- Claude 작성, `kubectl kustomize`로 4개 overlay 빌드 검증, 사용자 push
- 순서: stg 먼저 push해 확인 후 prod도 가능 (또는 한 번에)

### C. 확인

```bash
kube -n choimory-dev-stg get svc choimory-dev-postgres-tailnet choimory-dev-redis-tailnet
kube -n choimory-dev-prod get svc choimory-dev-postgres-tailnet choimory-dev-redis-tailnet
kube -n choimory-infra get pods | grep ts-
```

- 4개 Service의 EXTERNAL-IP에 `100.x`, Tailscale Machines에 기기 4개
- CNPG `Cluster` 상태 정상 (`kube -n choimory-dev-stg get cluster`)

### D. 클라이언트 접속

| 대상 | 호스트 | 포트 | 계정 |
|---|---|---|---|
| PostgreSQL | `choimory-dev-{env}-postgres` | 5432 | DB, 사용자 `member`, 비밀번호는 `choimory-dev-postgres-app` Secret |
| Redis | `choimory-dev-{env}-redis` | 6379 | 비밀번호는 `choimory-dev-redis-secret` Secret |

- 비밀번호 확인: `kube -n choimory-dev-{env} get secret <이름> -o jsonpath='{.data.<키>}' | base64 -d; echo`
- DataGrip prod 연결은 Read-only 체크, 연결 색상 구분
- 확인 후 Tailscale 끄고 접속 안 되는지 확인

### E. 접근 제한 (선택)

- 지금 Tailscale 정책은 `grants` 전체 허용이라 tailnet의 모든 기기가 DB에 닿음
- tailnet 기기가 내 기기뿐이면 그대로 둬도 됨
- 좁히려면 예: `{"src": ["autogroup:member"], "dst": ["tag:k8s"], "ip": ["tcp:443", "tcp:5432", "tcp:6379"]}` + 기존 전체 허용 규칙 조정. 다른 기기 간 통신까지 바뀌므로 별도로 검토

## 보안

| 항목 | 내용 |
|---|---|
| 전송 구간 | tailnet은 WireGuard로 암호화. PostgreSQL은 CNPG가 기본으로 TLS 제공 (클라이언트 `sslmode=require` 가능, 인증서는 CNPG 자체 CA) |
| 인증 | PostgreSQL `member` 비밀번호, Redis `requirepass`. 둘 다 기존 그대로 |
| prod | 조회 위주. 클라이언트 Read-only 설정. 필요하면 나중에 읽기 전용 DB 역할(CNPG managed roles + SealedSecret) 추가 |
| 노출 범위 | tailnet 기기만. 인터넷, 공유기 LAN에는 포트가 열리지 않음 |

## 주의할 점과 사전 확인

| 항목 | 내용 |
|---|---|
| CNPG와 Operator 충돌 | Operator가 Service에 finalizer와 status를 붙이고, CNPG는 추가 Service를 계속 관리함. 작성 전에 CNPG `1.30` 문서로 `managed.services.additional`의 `serviceTemplate`(annotations, `type`, `loadBalancerClass`)과 갱신 방식 확인. 문제 있으면 P3 |
| ArgoCD Health | `LoadBalancer` Service는 EXTERNAL-IP가 붙기 전까지 Progressing. Operator가 붙여 주므로 정상화됨 (`infra-gateway`와 동일) |
| prune 꺼짐 | `choimory-dev-protected`는 prune이 꺼져 있어, 나중에 Service 이름을 바꾸거나 지우면 예전 Service를 직접 삭제해야 함. Service가 지워지면 tailnet 기기도 사라짐 |
| 메모리 | 프록시 Pod 4개 추가 (`choimory-infra`). 기존 `ts-infra-gateway-proxy` 사용량을 `kube top pod`로 보고 필요하면 ProxyClass로 requests 지정 |
| 프록시 묶기 | Tailscale ProxyGroup으로 프록시 수를 줄이는 방법도 있으나 구성이 복잡해 이번에는 제외 |
| port-forward 정리 | 접속 확인 후 README의 DB port-forward는 비상용으로만 남김 |

## 결정 필요 사항

1. 노출 방식: PostgreSQL P1(CNPG 추가 Service), Redis R1(전용 Service) 추천
2. 이름: Service `*-tailnet`, 기기 `choimory-dev-{env}-{postgres|redis}`
3. push 순서: stg 먼저 확인 후 prod / 한 번에
4. 접근 제한(E): 이번에 할지, 나중에 할지

--- 

# 추가1

- 이름: {project-name}-{db-name}-{profile}
- push는 내가 알아서 함
- 노출방식이랑 접근제한은 설명좀

---

# 추가1 분석

> - 이름: {project-name}-{db-name}-{profile}
> - push는 내가 알아서 함
> - 노출방식이랑 접근제한은 설명좀

## 답변

- 이름 확정, push는 사용자 진행. 노출 방식과 접근 제한은 아래 설명

## 1. 이름

| 대상 | stg | prod |
|---|---|---|
| PostgreSQL | `choimory-dev-postgres-stg` | `choimory-dev-postgres-prod` |
| Redis | `choimory-dev-redis-stg` | `choimory-dev-redis-prod` |

- tailnet 기기 이름이자 접속 주소 (예: `choimory-dev-postgres-stg:5432`)
- prod도 접미사를 붙이는 것을 추천. 도메인은 prod에서 접두어를 뺐지만, DB는 어느 환경에 붙었는지 이름에 분명히 보여야 실수가 줄어듦
- k8s 안의 Service 이름은 네임스페이스로 환경이 나뉘므로 `choimory-dev-postgres-tailnet`처럼 환경 없이 두는 것을 제안

## 2. push

- 사용자가 직접 진행. stg, prod를 나누지 않고 한 번에 작성

## 3. 노출 방식 설명

### 전제: Service와 Tailscale 표시

- Service는 Pod 앞에 붙는 고정 주소. member-api는 `choimory-dev-postgres-rw`, `choimory-dev-redis` Service로 DB에 접속
- Tailscale Operator는 표시가 붙은 Service를 보면 프록시 Pod를 띄워 그 Service를 tailnet 기기로 등록
  - `loadBalancerClass: tailscale`을 쓴 LoadBalancer Service (`infra-gateway` 방식)
  - 일반 Service에 `tailscale.com/expose: "true"` annotation
- 결정할 것은 어느 Service에 표시를 붙이느냐

```
내 기기 ─tailnet─▶ 프록시 Pod ─▶ [표시가 붙은 Service] ─▶ DB Pod
member-api ──────────────────▶ [기존 Service]       ─▶ DB Pod
```

### PostgreSQL 방식 비교

- PostgreSQL은 CNPG가 Service를 자동으로 만들고 관리하는 점이 다름

| 방식 | 하는 일 | 핵심 차이 |
|---|---|---|
| P1. CNPG에 추가 Service 요청 (추천) | `Cluster` 설정에 쓰기(primary)용 Service를 Tailscale 표시와 함께 하나 더 만들도록 적음 | CNPG가 만들고 관리해 DB 재생성 시에도 따라감. 설정이 `Cluster` 한 곳에 모임 |
| P2. 기존 `-rw` Service에 표시 | member-api가 쓰는 Service에 annotation을 직접 붙임 | CNPG가 자기 Service를 되돌리면 표시가 사라질 수 있음. 앱이 쓰는 Service를 건드림 |
| P3. Service 직접 작성 | CNPG와 상관없이 Service를 작성해 primary Pod를 라벨로 연결 | CNPG와 충돌은 없지만 CNPG 내부 라벨 이름에 의존 |

- P1의 걱정: Operator가 Service에 붙이는 기록(finalizer, 상태)을 CNPG가 지우지 않는지. 작성 전 CNPG 문서로 확인, 문제 있으면 P3

### Redis 방식 비교

- Redis Service는 직접 만든 것이라 관리자가 없음

| 방식 | 하는 일 | 핵심 차이 |
|---|---|---|
| R1. 전용 Service 추가 (추천) | 같은 Redis Pod를 가리키는 Service를 하나 더 만들고 Tailscale 표시 | member-api용 Service는 그대로. PostgreSQL, `infra-gateway`와 같은 모양 |
| R2. 기존 Service에 표시 | `choimory-dev-redis`에 annotation 한 줄 | 가장 짧지만 앱이 쓰는 Service에 외부 접속 용도가 섞임 |

- P1, R1은 앱이 쓰는 Service는 그대로 두고 사람이 접속하는 용도를 별도 Service로 분리하는 방식. 외부 접속을 끌 때도 그 Service만 지우면 됨

## 4. 접근 제한 설명

### 지금 상태

- `grants`가 `{"src": ["*"], "dst": ["*"], "ip": ["*"]}` → tailnet 안의 모든 기기가 모든 기기의 모든 포트에 접속 가능
- tailnet 밖(인터넷, 공유기 LAN)에서는 어차피 접속 불가
- 판단 기준: tailnet에 내 기기 말고 다른 것이 있는지

| tailnet 구성 | 판단 |
|---|---|
| 내 Mac, 휴대폰, 홈서버 등 내 기기만 | 그대로 둬도 됨. DB 비밀번호까지 있어 이중 잠금 |
| 가족, 지인 계정, 공유받은 기기, 회사 기기가 있음 | 제한 권장 |

### 제한하면 어떻게 되나

- `grants`는 누가(src) → 어디로(dst) → 어떤 포트(ip)를 허용하는 목록. 목록에 없는 것은 모두 차단

```jsonc
"grants": [
  // 내 계정 기기 → k8s 기기(tag:k8s)는 HTTPS, PostgreSQL, Redis 포트만
  {"src": ["autogroup:member"], "dst": ["tag:k8s"], "ip": ["tcp:443", "tcp:5432", "tcp:6379"]},
  // 내 계정 기기끼리는 지금처럼 전부 허용 (홈서버 Docker 앱 등)
  {"src": ["autogroup:member"], "dst": ["autogroup:member"], "ip": ["*"]},
],
```

- 효과: k8s 기기(ArgoCD, DB)에는 내 계정 기기만 필요한 포트로 접속
- 주의: 전체 허용을 지우면 다른 통신에도 영향. 홈서버 Docker 앱(Immich, Jellyfin 등)에 다른 계정이나 공유 기기가 접속하던 경우 차단됨 → 정책 전체를 점검하면서 따로 하는 게 안전
- 추천: tailnet에 내 기기만 있으면 이번에는 넘어가고, 5단계 접속 확인 후 필요할 때 따로 진행

## 결정 필요 사항 (추가1)

1. prod 기기 이름도 `-prod` 접미사 (추천)
2. 노출 방식 P1, R1
3. 접근 제한: 이번에 / 나중에. 이번에 한다면 tailnet에 내 기기 외 다른 기기가 있는지 확인

---

# Service를 만든다는 의미

> service를 만든다는게 gitops에 service.yml 만드는건가?

## 답변

- 반은 맞음. Redis는 Service 매니페스트를 추가하고, PostgreSQL(P1)은 Service 파일 없이 기존 `Cluster` 설정에 몇 줄을 넣음
- 결과적으로 클러스터에는 둘 다 Service가 생기지만 만드는 주체가 다름

## Redis (R1): Service 매니페스트 추가

- `projects/choimory-dev/redis/base/redis.yaml`에 Service 추가 (파일 분리도 가능). ArgoCD가 그대로 생성

```yaml
# (예시) tailnet에서 Redis에 접속하기 위한 Service
apiVersion: v1
kind: Service
metadata:
  name: choimory-dev-redis-tailnet
spec:
  type: LoadBalancer
  loadBalancerClass: tailscale        # Tailscale Operator가 맡는 Service라는 표시
  selector:
    app: choimory-dev-redis           # 기존 Service와 같은 Redis Pod를 가리킴
  ports:
    - name: redis
      port: 6379
      targetPort: redis
```

- 기기 이름 annotation은 환경마다 달라 overlay에서 지정 (stg `tailscale.com/hostname: choimory-dev-redis-stg`, prod `choimory-dev-redis-prod`)

## PostgreSQL (P1): Cluster 설정에 추가

- `choimory-dev-postgres-rw` 등은 원래 파일로 만든 게 아니라 CNPG가 `Cluster`를 보고 자동 생성한 Service
- Service 파일 대신 `projects/choimory-dev/postgres/base/postgres.yaml`의 `Cluster`에 Service를 하나 더 만들도록 적음

```yaml
# (예시) 기존 Cluster spec 안에 추가
spec:
  managed:
    services:
      additional:
        - selectorType: rw              # primary(쓰기) Pod를 가리킴
          serviceTemplate:
            metadata:
              name: choimory-dev-postgres-tailnet
            spec:
              type: LoadBalancer
              loadBalancerClass: tailscale
```

- 기기 이름 annotation은 overlay에서 환경별 지정
- 정확한 필드 이름과 위치는 작성 시 CNPG `1.30` 문서로 확인

## 정리 (Service 생성 주체)

| 대상 | gitops에서 바꾸는 것 | 클러스터에 생기는 것 |
|---|---|---|
| Redis (R1) | Service 매니페스트 추가 (`redis.yaml`에 추가 또는 새 파일) | `choimory-dev-redis-tailnet` Service (ArgoCD가 생성) |
| PostgreSQL (P1) | 기존 `postgres.yaml`의 `Cluster`에 `managed.services` 추가 | `choimory-dev-postgres-tailnet` Service (CNPG가 생성) |
| PostgreSQL (P3였다면) | Service 매니페스트 추가 | Service (ArgoCD가 생성) |
| 공통 | overlay(stg, prod)에 기기 이름 annotation 패치 | |

- 결정 필요: Redis Service를 `redis.yaml`에 같이 둘지, `service-tailnet.yaml` 등으로 분리할지

---

# Redis, PostgreSQL에 Service, Deployment 파일이 없는 이유

> 그러고보니 왜 redis, postgres는 왜 service, deploy.yml이 없어?

## 답변

- Redis는 파일이 합쳐져 있는 것이고, PostgreSQL은 실제로 파일이 없음 (CNPG 오퍼레이터가 대신 생성)

## 앱 (front, member-api)

```
choimory-dev-member-api/base/
├── deployment.yaml
├── service.yaml
├── httproute.yaml
└── kustomization.yaml
```

- 일반적인 형태. Deployment(Pod 실행)와 Service(고정 주소)를 직접 작성

## Redis: 한 파일에 Service와 StatefulSet

- `redis/base/redis.yaml`에 `---`로 나뉜 문서 두 개

| 순서 | 종류 | 이름 |
|---|---|---|
| 1 | Service | `choimory-dev-redis` |
| 2 | StatefulSet | `choimory-dev-redis` |

- 합친 이유: Redis 하나를 구성하는 짝이라 같이 보이게 둔 것. 쿠버네티스는 파일 이름과 무관해 나눠도 결과는 같음
- Deployment가 아니라 StatefulSet인 이유: Redis는 데이터를 디스크(AOF)에 저장하는 상태 있는 앱

| | Deployment | StatefulSet |
|---|---|---|
| Pod 이름 | 매번 랜덤 (`...-7f9c-x2k`) | 고정 (`choimory-dev-redis-0`) |
| 디스크 | 여러 Pod가 하나를 같이 쓰거나 직접 연결 | Pod마다 전용 PVC 자동 생성(`volumeClaimTemplates`), 재시작해도 같은 디스크에 다시 붙음 |
| 용도 | 상태 없는 앱 (front, member-api) | DB, 캐시처럼 데이터를 저장하는 앱 |

## PostgreSQL: CNPG Cluster 하나

- `postgres/base/postgres.yaml`에는 `Cluster` 리소스 하나만 있음. 쿠버네티스 기본 리소스가 아니라 CloudNativePG(CNPG)가 추가한 커스텀 리소스

```
우리가 쓰는 것:   Cluster "choimory-dev-postgres" (인스턴스 1개, DB member, 저장소 5Gi ...)
                       │
                       ▼  CNPG 오퍼레이터가 보고 알아서 생성, 관리
CNPG가 만드는 것: Pod        choimory-dev-postgres-1
                 PVC        choimory-dev-postgres-1
                 Service    choimory-dev-postgres-rw / -ro / -r
                 Secret     choimory-dev-postgres-app (접속 정보), CA 인증서 등
```

- 오퍼레이터를 쓰는 이유: PostgreSQL은 초기화, 장애 시 primary 전환, 백업, 업그레이드 등 운영할 일이 많아 CNPG가 대신함. 우리는 원하는 DB를 `Cluster`에 적기만 함
- CNPG는 Deployment나 StatefulSet 없이 Pod를 직접 관리. `kube -n choimory-dev-stg get pods,svc,pvc`로 확인 가능
- 5단계에서 PostgreSQL용 Service를 하나 더 원할 때도 Service 파일이 아니라 `Cluster`에 적어 CNPG에 맡기는 방식(P1)을 추천한 이유

## 정리 (리소스 작성 방식)

| 대상 | 직접 작성하는 것 | 실제 Pod를 만드는 주체 |
|---|---|---|
| front, member-api | Deployment, Service (파일 분리) | Deployment |
| Redis | StatefulSet, Service (한 파일) | StatefulSet |
| PostgreSQL | `Cluster` 하나 | CNPG 오퍼레이터 |

---

# 추가2

- p1, r1으로 가자
- p1은 특성상 그대로 가고, r1은 대신 service.yaml을 한 파일에 넣지 말고 분리
- 이름은 {project-name}-{db-name}-{profile}
- 마지막으로 접근제한 설명좀 

---

# 추가2 분석

> - p1, r1으로 가자
> - p1은 특성상 그대로 가고, r1은 대신 service.yaml을 한 파일에 넣지 말고 분리
> - 이름은 {project-name}-{db-name}-{profile}
> - 마지막으로 접근제한 설명좀

## 답변

- P1, R1과 이름 확정. 접근 제한을 기초부터 다시 설명

## 1. 확정 사항

| 항목 | 결정 |
|---|---|
| PostgreSQL | P1. `Cluster`의 `managed.services.additional` (CNPG가 Service 생성) |
| Redis | R1. 전용 Service를 별도 파일로 |
| 기기 이름 | `choimory-dev-postgres-{stg,prod}`, `choimory-dev-redis-{stg,prod}` |

```
redis/base/
├── kustomization.yaml       # resources에 service-tailnet.yaml 추가
├── redis.yaml               # 기존 Service + StatefulSet (그대로)
└── service-tailnet.yaml     # 새 tailnet용 Service
```

- 기존 `redis.yaml`을 앱처럼 `service.yaml`, `statefulset.yaml`로 나누는 것도 가능. 리소스 이름이 그대로라 클러스터 영향 없음(prune 꺼져 있어도 문제없음). 이번 범위 밖이라 선택

## 2. 접근 제한 설명

### DB까지 거치는 잠금

```
[인터넷, 공유기 LAN] ──✕──  tailnet 밖은 경로 자체가 없음
        │
[tailnet 안 기기] ─── ① Tailscale 정책(grants): 누가 어디에 접속할 수 있나  ← 지금 "전부 허용"
        │
   프록시 Pod (choimory-infra)
        │
        ├── ② NetworkPolicy: choimory-infra에서 오는 요청은 허용 (프록시가 여기서 오므로 통과)
        │
   DB Pod ─── ③ DB 인증: PostgreSQL member 비밀번호, Redis requirepass
```

- 접근 제한은 ①을 "전부 허용"에서 "필요한 것만 허용"으로 좁히는 작업
- ②는 프록시가 `choimory-infra`에서 오므로 tailnet 기기를 구분하지 못함. 사람 단위 제한은 ①에서만 가능
- ③은 지금도 있음. 비밀번호를 모르면 접속해도 사용 불가

### grants 제한이 의미 있는 경우

- grants는 tailnet에 들어온 기기 사이의 규칙이라, 효과는 tailnet에 누가 있느냐에 달림

| tailnet 상황 | 지금(전부 허용)의 위험 | 제한의 효과 |
|---|---|---|
| 내 계정, 내 기기만 | 거의 없음. 모든 기기가 나 | 작음. 내 기기 하나가 털렸을 때 DB까지 바로 닿지 않게 하는 정도 |
| 다른 사용자 초대, 기기 공유받음 | 그 사람 기기에서 DB, ArgoCD 포트에 접속 시도 가능 | 큼. 그 사람은 k8s 기기에 닿지 못함 |
| 서버, 기기가 많아짐 | 한 기기가 뚫리면 tailnet 전체로 이동 가능 | 큼. 기기별로 필요한 포트만 열림 |

- 확인: Tailscale 관리 화면 Users에 본인만 있는지, Machines에 모르는 기기나 공유받은(Shared) 기기가 없는지

### 좁힌다면 이런 모양

```jsonc
"grants": [
  // 내 계정 기기 → k8s 기기(infra-gateway, DB 4개)는 HTTPS, PostgreSQL, Redis 포트만
  {"src": ["autogroup:member"], "dst": ["tag:k8s"], "ip": ["tcp:443", "tcp:5432", "tcp:6379"]},
  // 내 계정 기기끼리는 지금처럼 전부 허용 (홈서버 Docker 앱, SSH 등)
  {"src": ["autogroup:member"], "dst": ["autogroup:member"], "ip": ["*"]},
],
```

| 용어 | 뜻 |
|---|---|
| `autogroup:member` | 내 tailnet의 사용자 계정(로그인한 사람의 기기). 태그 기기는 포함 안 됨 |
| `tag:k8s` | Operator가 만든 기기 전체 (`choimory-infra-gateway`, DB 4개) |
| `ip` | 허용할 포트. 나머지는 막힘 |

- k8s 기기에는 HTTPS, 5432, 6379 외 포트로 접속 불가
- 태그 기기(k8s)에서 다른 기기로 먼저 접속하는 것도 규칙에 없어 차단
- 공유받은 기기, 다른 사용자는 규칙에 없어 모두 차단

### 바꾸기 전에 주의할 것

1. 전부 허용 줄을 지우는 순간 그 밖의 통신은 모두 막힘. 지금 되는 것(휴대폰 → 홈서버 Immich, Jellyfin, Mac → 서버 SSH, kubectl 등)이 두 번째 규칙에 들어가는지 확인. 내 계정 기기끼리면 계속 됨
2. Tailscale SSH(`ssh` 블록)는 별개 설정이라 영향 없음
3. 안전하게 바꾸기: 정책 파일 `tests`에 기대 결과(예: 내 Mac → `tag:k8s:5432` 허용, `tag:k8s` → 내 Mac 차단)를 적으면 저장 시 Tailscale이 검증하고 틀리면 저장 거부. 관리 화면 Preview rules로 특정 기기에서 닿는 대상 미리 확인
4. 문제 시 관리 화면의 변경 이력으로 이전 정책으로 되돌림

### 추천

- Users에 본인만 있고 공유 기기가 없으면: 이번 5단계에서는 넘어감. DB 비밀번호와 tailnet 자체가 이미 잠금 역할
- 다른 사용자나 공유 기기가 있으면: 5단계 접속 확인 직후 위 모양으로 좁히고 `tests`도 함께 작성

## 결정 필요 사항 (추가2)

1. 기존 `redis.yaml`도 이번에 `service.yaml`, `statefulset.yaml`로 분리할지, 새 파일만 추가할지
2. 접근 제한: Users, Machines 확인 후 이번에 할지

---

# 접근 제한 요약 확인

> 접근제한은 요약하자면 taiscale에 아까 했던 작업에 또 추가해서 tailscale 멤버 중 특정 멤버만 db 접근 할수 있게 하자는거지?

## 답변

- 대체로 맞음. 몇 가지 보완

## 맞는 부분

- 같은 곳을 고침: 아까 `tagOwners`를 추가한 Tailscale Access controls 정책 파일
- `grants`를 "전부 허용" 대신 "허용할 것만 나열"로 바꿈
- 정한 대상만 DB(그리고 ArgoCD)에 접근

## 보완할 부분

1. "누구"를 얼마나 좁힐지 선택 가능

| src에 넣는 값 | 의미 |
|---|---|
| `autogroup:member` (앞의 예시) | tailnet의 모든 사용자 계정 기기. 공유받은 기기는 제외 |
| 특정 계정 (이메일) | 그 사람의 기기만 |
| `group:db-admins` 같은 그룹 | `groups`에 정의한 사람들만 |
| 특정 기기 태그 (예: `tag:my-mac`) | 사람이 아니라 특정 기기만 (휴대폰 제외 등) |

- 지금 tailnet 사용자가 본인뿐이면 `autogroup:member`가 곧 "나". 사실상 아래 2, 3번 효과가 큼

2. 포트도 제한: k8s 기기에는 443(ArgoCD), 5432(PostgreSQL), 6379(Redis)만 열림
3. 반대 방향도 차단: k8s 기기(태그 기기)에서 내 Mac, 홈서버로 먼저 접속하는 것을 막음. 클러스터 안이 뚫려도 tailnet의 다른 기기로 넘어오지 못함

## 한 줄 요약

- Tailscale 정책 파일에서 "전부 허용"을 지우고 "정한 사람(또는 기기) → k8s 기기, 정한 포트만" 허용하는 규칙으로 바꾸는 것
- 본인 혼자 쓰는 tailnet이면 우선순위 낮음. 다른 사람을 초대하거나 기기를 공유할 때 꼭 진행

---

# 추가3

- redis 파일 다 분리해줘. 한 파일에 넣는거 별로 안좋아함. 어차피 리소스별로 디렉토리 팠으니 파일 하나에 다 쓸 필요 없지
- 접근제한은 그냥 유지하자. tailscale에 추가할 사람도 없고, 추가해도 개발쪽으로 익숙한 사람 없고, 개발쪽으로 익숙한 사람을 추가한다면 그건 애초에 개발을 같이 할 사람일거라 오히려 열어주는게 논리에 맞음. 즉 tailscale 자체가 같이 개발할사람만 추가한다는 개념이 될 듯. vpn 서버에 접근할 devops 개발자 계정을 추가해주는 느낌으로.

---

# 추가3 분석

> - redis 파일 다 분리해줘. 한 파일에 넣는거 별로 안좋아함. 어차피 리소스별로 디렉토리 팠으니 파일 하나에 다 쓸 필요 없지
> - 접근제한은 그냥 유지하자. tailscale에 추가할 사람도 없고, 추가해도 개발쪽으로 익숙한 사람 없고, 개발쪽으로 익숙한 사람을 추가한다면 그건 애초에 개발을 같이 할 사람일거라 오히려 열어주는게 논리에 맞음. 즉 tailscale 자체가 같이 개발할사람만 추가한다는 개념이 될 듯. vpn 서버에 접근할 devops 개발자 계정을 추가해주는 느낌으로.

## 답변

- 5단계 결정 사항 모두 확정

## 1. Redis 파일 분리

- 앱 디렉토리와 같은 규칙(리소스별 파일)

```
redis/base/
├── kustomization.yaml       # resources: service.yaml, statefulset.yaml, service-tailnet.yaml
├── service.yaml             # 기존 Service choimory-dev-redis (member-api용)
├── statefulset.yaml         # 기존 StatefulSet choimory-dev-redis
└── service-tailnet.yaml     # 새 Service choimory-dev-redis-tailnet
```

- 기존 두 리소스는 이름과 내용 그대로 파일만 이동. ArgoCD 입장에서 같은 리소스라 재생성, 재시작 없음
- `redis.yaml`은 삭제

## 2. 접근 제한 유지

- "tailnet 멤버 = 함께 개발, 운영하는 사람"으로 운영. VPN 서버에 DevOps 계정을 발급하는 것과 같은 개념. Tailscale 초대 자체가 접근 권한 부여
- 지킬 원칙 (README에 한 줄로 기록)
  - 초대는 같이 개발할 사람만. 지인 기기 공유(Share) 등도 같은 기준
  - 함께하던 사람이 빠지면 Users에서 제거 (그 사람 기기도 함께 빠짐)
  - 인원이 늘거나 역할이 나뉘면(예: prod DB는 일부만) 그때 `grants`로 좁힘

## 5단계 최종 작업 내용

| 파일 | 내용 |
|---|---|
| `projects/choimory-dev/postgres/base/postgres.yaml` | `Cluster`에 `managed.services.additional` (rw 대상 `choimory-dev-postgres-tailnet`, `LoadBalancer`, `loadBalancerClass: tailscale`) |
| `projects/choimory-dev/postgres/overlays/{stg,prod}/kustomization.yaml` | 기기 이름 `choimory-dev-postgres-{stg,prod}` |
| `projects/choimory-dev/redis/base/` | `redis.yaml` → `service.yaml`, `statefulset.yaml`로 분리, `service-tailnet.yaml` 추가, kustomization 수정 |
| `projects/choimory-dev/redis/overlays/{stg,prod}/kustomization.yaml` | 기기 이름 `choimory-dev-redis-{stg,prod}` |
| `README.md` | DB 접속 방식(Tailscale), port-forward는 비상용, tailnet 멤버 원칙 |

1. CNPG `1.30` 문서로 `managed.services.additional` 필드와 Operator 충돌 여부 확인. 문제 있으면 P3로 바꾸고 공유
2. 파일 작성, `kubectl kustomize`로 4개 overlay 빌드 검증
3. push는 사용자 진행. 확인 명령과 DataGrip 접속 정보 제공

---

# 5단계 최종 작업계획

> 5단계 작업 시작 전 최종계획 이야기해줘

## 답변

- CNPG `1.30` 공식 문서(Service management)를 확인한 결과 P1 그대로 사용 가능
- 확인 내용을 반영한 최종 계획

## 사전 확인 결과 (CNPG 1.30)

| 확인 | 결과 | 영향 |
|---|---|---|
| `managed.services.additional` | 지원. `selectorType: rw`, `serviceTemplate`(metadata 이름, 라벨, annotation, spec 전체) | P1 그대로 진행 |
| `type: LoadBalancer` | 문서 예시가 바로 이 용도(primary용 LoadBalancer) | `loadBalancerClass: tailscale`도 spec에 그대로 |
| `selector` | 지정 금지. CNPG가 primary를 따라가도록 관리 | 템플릿에 넣지 않음 |
| `updateStrategy` | 기본 `patch`(템플릿 값만 덮어씀), `replace`는 삭제 후 재생성 | 기본 `patch`. Operator가 붙이는 finalizer, 상태는 템플릿에 없는 값이라 건드리지 않음 |
| 이름 | `<Cluster>-rw/ro/r`는 예약 | `choimory-dev-postgres-tailnet` 사용 가능 |

## 확정 값

| 대상 | k8s Service | tailnet 기기 (접속 주소) |
|---|---|---|
| PostgreSQL stg | `choimory-dev-postgres-tailnet` (CNPG가 생성) | `choimory-dev-postgres-stg:5432` |
| PostgreSQL prod | 〃 | `choimory-dev-postgres-prod:5432` |
| Redis stg | `choimory-dev-redis-tailnet` (ArgoCD가 생성) | `choimory-dev-redis-stg:6379` |
| Redis prod | 〃 | `choimory-dev-redis-prod:6379` |

- 태그 `tag:k8s`(Operator 기본값). 접근 제한 없음 (tailnet 멤버 = 함께 개발하는 사람)
- NetworkPolicy, Tailscale 관리 화면, 인프라 영역 변경 없음

## 작업 단계 (최종)

### A. gitops 작성 (최종)

- Claude 작성

| 파일 | 변경 |
|---|---|
| `postgres/base/postgres.yaml` | `Cluster.spec.managed.services.additional`에 rw 대상 `choimory-dev-postgres-tailnet` (`LoadBalancer`, `loadBalancerClass: tailscale`) |
| `postgres/overlays/stg/kustomization.yaml` | 기기 이름 annotation `choimory-dev-postgres-stg` 패치 |
| `postgres/overlays/prod/kustomization.yaml` | 기기 이름 annotation `choimory-dev-postgres-prod` 패치 |
| `redis/base/redis.yaml` | 삭제 |
| `redis/base/service.yaml` | 기존 Service `choimory-dev-redis` (내용 그대로 이동) |
| `redis/base/statefulset.yaml` | 기존 StatefulSet `choimory-dev-redis` (내용 그대로 이동) |
| `redis/base/service-tailnet.yaml` | 새 Service `choimory-dev-redis-tailnet` |
| `redis/base/kustomization.yaml` | resources를 위 3개 파일로 변경 |
| `redis/overlays/{stg,prod}/kustomization.yaml` | 기기 이름 annotation `choimory-dev-redis-{stg,prod}` 패치 |
| `README.md` | DB 접속을 Tailscale로 변경, port-forward는 비상용, tailnet 멤버 원칙 한 줄, 디렉토리 구조의 redis 항목 |

- overlay 패치 방식
  - PostgreSQL: CRD 안의 목록 항목이라 일반 병합 패치는 목록 전체를 덮어씀 → JSON 패치로 `additional[0]`의 annotation만 추가
  - Redis: 일반 Service라 기존처럼 병합 패치로 annotation 추가

### B. 검증 (최종)

- `kubectl kustomize`로 4개 overlay(postgres stg/prod, redis stg/prod) 빌드
  - 기기 이름이 환경별로 맞게 들어가는지
  - 분리 전후 Redis 기존 Service, StatefulSet이 내용상 같은지 (diff)

### C. push (최종)

- 사용자 진행
- 반영 대상: `choimory-dev-protected`의 postgres, redis Application 4개
- Redis 기존 리소스는 이름, 내용이 같아 재시작 없음. PostgreSQL도 `Cluster`에 Service 설정만 추가되어 DB Pod 재시작 없음

### D. 확인 (최종)

```bash
kube -n choimory-dev-stg get svc choimory-dev-postgres-tailnet choimory-dev-redis-tailnet
kube -n choimory-dev-prod get svc choimory-dev-postgres-tailnet choimory-dev-redis-tailnet
kube -n choimory-infra get pods | grep ts-
kube -n choimory-dev-stg get cluster choimory-dev-postgres
```

- Service 4개 EXTERNAL-IP `100.x`, 프록시 Pod 4개 추가, Tailscale Machines에 기기 4개, `Cluster` Healthy

### E. 접속 확인 (최종)

- Mac, Tailscale 켬

```bash
nc -vz choimory-dev-postgres-stg 5432
nc -vz choimory-dev-redis-stg 6379
```

| 대상 | 호스트 | 계정 | 비밀번호 확인 (서버) |
|---|---|---|---|
| PostgreSQL | `choimory-dev-postgres-{env}` : 5432, DB `member` | `member` | `kube -n choimory-dev-{env} get secret choimory-dev-postgres-app -o jsonpath='{.data.password}' \| base64 -d; echo` |
| Redis | `choimory-dev-redis-{env}` : 6379 | (없음) | `kube -n choimory-dev-{env} get secret choimory-dev-redis-secret -o jsonpath='{.data.password}' \| base64 -d; echo` |

- PostgreSQL SSL `require` (CNPG 기본 TLS)
- prod 연결은 Read-only 체크, 색상 구분
- Tailscale 끄면 접속 안 되는지 확인

## 주의할 점 (최종)

| 항목 | 내용 |
|---|---|
| prune 꺼짐 | postgres, redis는 `choimory-dev-protected`라 prune 꺼짐. Redis 파일 분리는 이름이 같아 문제없음. 나중에 `service-tailnet`을 지우면 클러스터에서 직접 삭제 (PostgreSQL은 `additional`에서 빼면 CNPG가 삭제) |
| MagicDNS | 짧은 이름은 Mac Tailscale MagicDNS가 켜져 있어야 풀림 (기본 켜짐). 안 되면 `<기기>.<tailnet>.ts.net` 전체 이름 |
| 메모리 | 프록시 Pod 4개 추가. 반영 후 `kube top pod -n choimory-infra`로 확인, 필요하면 ProxyClass로 requests 지정 (후속) |
| 되돌리기 | PostgreSQL은 `additional`을 빼고 push. Redis는 `service-tailnet.yaml`을 빼고 push 후 Service 수동 삭제. 기기는 Service 삭제 시 함께 사라짐 |

## 기록

- 작업 기록은 `task5.md`, calendar는 `TASK` 유지
