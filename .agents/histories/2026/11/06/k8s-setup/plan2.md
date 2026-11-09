# 목차

- [개요](#개요)
- [개발방향](#개발방향)
- [개발방향 분석](#개발방향-분석)
  - [답변](#답변)
  - [여러 프로젝트를 담는 gitops 레포 구조](#여러-프로젝트를-담는-gitops-레포-구조)
  - [환경 구성 (staging, prod)](#환경-구성-staging-prod)
  - [네임스페이스](#네임스페이스)
  - [DB와 Redis: 프로젝트별 vs 공용 infra](#db와-redis-프로젝트별-vs-공용-infra)
  - [Ingress 선택지](#ingress-선택지)
  - [결정 필요 사항](#결정-필요-사항)
- [추가1](#추가1)
- [추가1 분석](#추가1-분석)
  - [답변](#답변-1)
  - [Envoy Gateway: 서버 부담과 다른 스택과의 영향](#envoy-gateway-서버-부담과-다른-스택과의-영향)
  - [projects로 감싸기](#projects로-감싸기)
  - [환경: local, dev, prod](#환경-local-dev-prod)
  - [도메인](#도메인)
  - [남은 결정 사항](#남은-결정-사항)
- [추가2](#추가2)
- [추가2 분석](#추가2-분석)
  - [답변](#답변-2)
  - [확정된 것](#확정된-것)
  - [API 도메인: staging.api.choimory.dev 확인 필요](#api-도메인-stagingapichoimorydev-확인-필요)
  - [dev → prod 승격 방식](#dev--prod-승격-방식)
  - [front 빌드 방식](#front-빌드-방식)
  - [DB와 Redis 방식](#db와-redis-방식)
  - [남은 결정 사항](#남은-결정-사항-1)
- [추가3](#추가3)
- [추가3 분석](#추가3-분석)
  - [답변](#답변-3)
  - [API 도메인: 한 단계 서브도메인](#api-도메인-한-단계-서브도메인)
  - [DB와 Redis: 프로젝트별 vs 하나를 공유](#db와-redis-프로젝트별-vs-하나를-공유)
  - [승격과 gitops 브랜치](#승격과-gitops-브랜치)
  - [front 빌드: Jenkins와 Actions 비교](#front-빌드-jenkins와-actions-비교)
  - [local-path 경로 변경](#local-path-경로-변경)
  - [stg 브랜치 유지와 도메인 충돌](#stg-브랜치-유지와-도메인-충돌)
  - [남은 결정 사항](#남은-결정-사항-2)
- [추가4](#추가4)
- [추가4 분석](#추가4-분석)
  - [답변](#답변-4)
  - [도메인 확정](#도메인-확정)
  - [front 빌드와 승격 확정](#front-빌드와-승격-확정)
  - [환경 변경안 검토: staging, prod 배포와 dev 서버 보류](#환경-변경안-검토-staging-prod-배포와-dev-서버-보류)
  - [DB와 Redis 해석 확인](#db와-redis-해석-확인)
  - [남은 결정 사항](#남은-결정-사항-3)
- [추가5](#추가5)
- [추가5 분석](#추가5-분석)
  - [답변](#답변-5)
  - [application-stg.yml 작성안](#application-stgyml-작성안)
  - [DB와 Redis: 환경별 공용 vs 프로젝트별·환경별](#db와-redis-환경별-공용-vs-프로젝트별환경별)
  - [이름 통일: stg와 staging](#이름-통일-stg와-staging)
  - [API 도메인: API 게이트웨이 없이 API를 계속 추가할 때](#api-도메인-api-게이트웨이-없이-api를-계속-추가할-때)
  - [남은 결정 사항](#남은-결정-사항-4)
- [추가6](#추가6)
- [추가6 분석](#추가6-분석)
  - [답변](#답변-6)
  - [확정 사항](#확정-사항)
  - [프로젝트별 DB와 Redis의 서버 영향](#프로젝트별-db와-redis의-서버-영향)
  - [application-stg.yml 작성 계획](#application-stgyml-작성-계획)
  - [남은 결정 사항](#남은-결정-사항-5)
- [추가6 남은 결정 사항 확인](#추가6-남은-결정-사항-확인)
  - [답변](#답변-7)
  - [gRPC](#grpc-1)
  - [local-path 경로와 pathPattern](#local-path-경로와-pathpattern)
  - [application-stg.yml 작성 시점](#application-stgyml-작성-시점)
  - [남은 결정 사항](#남은-결정-사항-6)
- [pathPattern 사용 확정](#pathpattern-사용-확정)
  - [답변](#답변-8)
- [DB와 Redis 데이터 경로: 프로젝트 폴더](#db와-redis-데이터-경로-프로젝트-폴더)
  - [답변](#답변-9)
  - [방법: 프로젝트별 StorageClass](#방법-프로젝트별-storageclass)
  - [장단점](#장단점)
  - [확정 사항](#확정-사항-1)

---

# 개요

- plan1에서 확정한 최종 스펙을 바탕으로 개발 방향을 정함
- plan1의 미정 항목(CNI·Gateway, DB·Redis 위치, 비밀값 관리, 환경 범위, GHCR 공개 여부)도 이 문서에서 이어서 결정

---

# 개발방향

- gitops 리포에서 여러 프로젝트들을 관리할거임
- 일단은 choimory-dev라는 프로젝트만 개발중이므로 해당 프로젝트만 추가하지만, 이후 내가 추가로 개발하는 다른 종류의 프로젝트들도 여기서 관리할거임
- profile은 local, dev, stg(staging), prod가 있는데 staging과 prod만 서버에서 관리할거임. local는 로컬용, dev는 docker-compose용으로 고려중
- 이러면 gitops 루트 디렉토리는 choimory-dev 외 프로젝트별로 가져갈거고, profile 구분은 kustomize로 하면 될거같음
- namespace는 현재 홈서버에서 프로젝트별로 두개의 profile을 구분한다면 project-staging, project-prod 이런식으로 구분해야하나
- DB, Redis는 프로젝트별로 클러스터에 포함하는게 나을지 infra라는 루트 디렉토리에서 같이 사용할지 고민중
- ingress는 선택사항들과 각각의 특징 알려줘 traefik이라던지 

---

# 개발방향 분석

> plan2 확인

## 답변

- 개발방향의 레포 구조, 환경 구성, 네임스페이스, DB·Redis 위치, Ingress 선택지를 분석함

## 여러 프로젝트를 담는 gitops 레포 구조

- 프로젝트별 루트 디렉토리 방향에 찬성. 다만 클러스터 공용 영역과 프로젝트 영역을 디렉토리로 한 단계 나누는 것을 추천

```txt
bootstrap/                      # ArgoCD Helm values + root app (최초 1회 수동 설치)
platform/                       # 클러스터 공용 (프로젝트와 무관)
  argocd/                       #   ArgoCD 자기 자신
  gateway/                      #   Gateway 구현체 + 공용 Gateway
  cloudflared/
  sealed-secrets/
  cloudnative-pg/               #   DB 오퍼레이터 (인스턴스는 프로젝트 쪽)
  local-path-provisioner/
  metrics-server/
projects/
  choimory-dev/
    argocd/                     #   AppProject, ApplicationSet (이 프로젝트의 ArgoCD 등록 정보)
    member-api/
      base/
      overlays/staging/
      overlays/prod/
    front/
      base/
      overlays/staging/
      overlays/prod/
    data/                       #   이 프로젝트의 PostgreSQL, Redis 인스턴스
      base/
      overlays/staging/
      overlays/prod/
  (다음 프로젝트)/
```

- `projects/`로 한 번 감싸는 이유: 프로젝트를 루트에 바로 두면 `bootstrap/`, `platform/`과 프로젝트 디렉토리가 같은 층에 섞임. ArgoCD ApplicationSet은 `projects/*` 하나만 지정하면 프로젝트를 자동으로 찾게 할 수 있음
- ArgoCD AppProject: 프로젝트마다 하나씩 만들고 배포 가능한 네임스페이스를 `choimory-dev-*`로 제한. 다른 프로젝트의 매니페스트가 실수로 이 프로젝트 영역을 건드리는 것을 막음
- 새 프로젝트 추가: `projects/<새 프로젝트>/` 디렉토리를 만들고 커밋하면 끝

## 환경 구성 (staging, prod)

- local은 로컬, dev는 docker-compose, 서버에는 staging과 prod만 두는 구성 찬성. Kustomize overlay를 `staging`과 `prod`로 나눔

### member-api의 stg 프로필

- 현재 `local`, `dev`, `prod`만 있어 `application-stg.yml`과 프로필 그룹 추가 필요 (앱 레포 작업)
- CORS에 들어갈 staging 도메인도 정해야 함

### front의 환경별 이미지

- `NEXT_PUBLIC_MEMBER_API_BASE_URL`이 빌드할 때 번들에 고정됨
- member-api는 이미지 하나를 staging에서 검증하고 그대로 prod로 올릴 수 있지만, front는 현재 구조로는 불가
- 방법 1: 환경별로 따로 빌드하고 태그를 나눔 (예: `stg-<sha>`, `prod-<sha>`)
- 방법 2: 브라우저도 Next 서버(Route Handler)를 거쳐 API를 부르게 바꿈. `NEXT_PUBLIC_` 변수가 필요 없어지고 이미지 하나로 두 환경 사용 가능

### staging → prod 승격 흐름

- 예시
  - `main` push → Actions가 staging overlay의 태그를 갱신해 자동 배포
  - 릴리스 태그 생성 또는 직접 PR → prod overlay의 태그 갱신

## 네임스페이스

- `프로젝트-환경` 방식이 표준적인 구성

```txt
choimory-dev-staging
choimory-dev-prod
argocd, cloudflared, cnpg-system, ...   # platform 공용
```

- 네임스페이스 단위로 권한(RBAC), 자원 상한(ResourceQuota, LimitRange), 네트워크 격리(NetworkPolicy)를 따로 걸 수 있음
- 노드 1대에서 staging과 prod가 같이 돌기 때문에 staging에 ResourceQuota를 걸어 staging 부하가 prod에 영향을 주지 않게 함

## DB와 Redis: 프로젝트별 vs 공용 infra

| | 프로젝트·환경별 인스턴스 | 공용 infra 인스턴스 하나 |
| --- | --- | --- |
| 구성 | `choimory-dev-prod` 안에 PostgreSQL 클러스터와 Redis | `infra` 네임스페이스에 PostgreSQL 하나(DB만 나눔)와 Redis 하나 |
| 격리 | staging 부하나 장애가 prod에 영향 없음 | 업그레이드나 장애가 모든 프로젝트와 환경에 영향 |
| 비밀값 | Secret이 앱과 같은 네임스페이스에 있어 단순함 | Secret은 다른 네임스페이스에서 못 읽어서 네임스페이스마다 복제하거나 따로 관리해야 함 |
| Redis 분리 | 인스턴스 단위로 깔끔하게 분리 | DB 번호나 키 접두어로 나눠야 해서 분리가 약함 |
| 자원 | 인스턴스 수만큼 메모리 사용 (PostgreSQL 약 100~300MB, Redis 수십 MB) | 가장 적음 |
| 프로젝트 삭제 | 디렉토리를 지우면 함께 정리됨 | DB와 계정을 따로 정리해야 함 |

- 추천: 혼합 방식
  - 오퍼레이터는 공용: CloudNativePG 오퍼레이터는 `platform/`에 한 번만 설치
  - 인스턴스는 프로젝트·환경별: `projects/choimory-dev/data/`에 두면 각 네임스페이스에 생성
- RAM 32GB라 인스턴스 4개(staging과 prod의 PostgreSQL, Redis) 정도는 여유 있음
- 노드가 1대라 복제본은 의미가 없으므로 인스턴스당 1개로 두고, 대신 prod는 백업을 챙김
- "infra 루트 디렉토리" 아이디어는 오퍼레이터처럼 정말 공용인 것을 두는 `platform/`으로 살림

## Ingress 선택지

- 쿠버네티스 Ingress API는 기능 추가가 멈췄고, 후속으로 Gateway API가 표준이 됨
- 가장 많이 쓰이던 ingress-nginx는 2026년 3월에 유지보수 종료
- 새로 구축한다면 Gateway API를 지원하는 구현체를 고르는 게 좋음

| 구현체 | 특징 | 장점 | 단점 |
| --- | --- | --- | --- |
| Traefik | k3s 기본 탑재. Ingress, 자체 CRD(IngressRoute), Gateway API 모두 지원 | 가볍고 대시보드가 있음. 미들웨어(인증, 리다이렉트, rate limit) 풍부. 설정이 쉬움 | 고급 기능은 자체 CRD 위주라 다른 구현체로 옮기기 어려움 |
| Envoy Gateway | Envoy 프로젝트의 Gateway API 전용 구현체 | Gateway API 표준을 가장 충실히 따름. 실무 채택이 늘고 있음 | 컨트롤러와 Envoy 프록시가 따로 떠서 구성요소가 많음 |
| Cilium Gateway | Cilium CNI에 내장된 Gateway API 구현 | Cilium을 CNI로 쓰면 추가 설치가 없음 (구성요소 하나로 해결) | 전용 제품보다 기능이 적음. 문제가 생기면 CNI와 같이 들여다봐야 함 |
| NGINX Gateway Fabric | F5(NGINX사)의 Gateway API 구현체 | NGINX에 익숙하면 친숙함 | 커뮤니티 규모가 작음 |
| Istio | 서비스 메시 + Gateway | 메시 기능까지 경험 가능 | 지금 규모에는 과함 |
| ingress-nginx | 유지보수 종료 | - | 제외 |

### 이 환경에서 영향을 주는 점

- TLS 인증서: Cloudflare Tunnel 앞단에서 TLS를 처리하므로 처음에는 Gateway가 HTTP만 받아도 됨. cert-manager도 당장은 불필요
- 도메인 라우팅: Tunnel을 `*.choimory.dev` 와일드카드로 Gateway에 연결하면 도메인별 라우팅은 HTTPRoute로 처리. 예: `choimory.dev`(prod), `stg.choimory.dev`(staging), `api.choimory.dev`

### 추천

- CNI를 Cilium으로 간다면 → Cilium Gateway (구성이 가장 단순)
- 대시보드와 미들웨어 같은 편의 기능이 중요하다면 → Traefik
- Gateway API 표준을 제대로 익히는 게 목적이라면 → Envoy Gateway

## 결정 필요 사항

- [ ] 디렉토리 구조: `projects/`로 감쌀지, 루트에 프로젝트를 바로 둘지
- [ ] front의 환경별 빌드: 이미지를 따로 빌드할지, 런타임 주입으로 바꿀지
- [ ] staging → prod 승격 방식 (예: `main` push는 staging 자동, 릴리스 태그는 prod)
- [ ] DB와 Redis: 혼합 방식(공용 오퍼레이터 + 프로젝트·환경별 인스턴스)으로 갈지
- [ ] Ingress: Cilium Gateway, Traefik, Envoy Gateway 중 선택

---

# 추가1

- Envoy Gateway로 가자. 다른 스택이랑 서버스펙에 영향주는 차이는 없나?
- projects로 감싸자
- profile은 바꾸자. local, dev, prod로 갈꺼고, dev랑 prod를 서버배포할거임. profile명도 dev/prod
- 대신 choimory-dev의 개발서버는 도메인만 staging.choimory.dev로 할거임
- staging을 하려던 이유가 choimory-dev에 dev라는 도메인을 붙여서 dev.choimory.dev하면 이상해서 였는데 그냥 도메인만 수정하는게 낫겠다

---

# 추가1 분석

> plan2의 추가1 확인

## 답변

- 추가1의 Envoy Gateway, projects 구조, 환경(profile), 도메인 항목을 분석함

## Envoy Gateway: 서버 부담과 다른 스택과의 영향

### 서버 자원

| | Envoy Gateway | Traefik | Cilium Gateway |
| --- | --- | --- | --- |
| 떠 있는 것 | 컨트롤러 1개 + Gateway마다 Envoy 프록시 1개 | Traefik 1개 | Cilium에 내장 (추가 없음) |
| 유휴 메모리 | 약 200~400MB | 약 50~100MB | 사실상 0 (Cilium에 포함) |

- 차이는 수백 MB 수준이라 RAM 32GB에서는 의미 없음
- 전역 rate limit을 쓰면 rate limit 서비스와 Redis가 추가로 뜨지만, 쓰지 않으면 생기지 않음

### 다른 스택과 맞물리는 부분

- 자원보다는 설정할 때 주의할 점이 있음

| 항목 | 내용 |
| --- | --- |
| LoadBalancer 대기 문제 | Envoy 프록시 Service는 기본이 `LoadBalancer` 타입. kubeadm에는 LB가 없어 외부 IP가 영원히 `Pending`이 되고, ArgoCD는 이 Service를 계속 "Progressing"으로 표시함. EnvoyProxy 설정에서 Service 타입을 `ClusterIP`로 변경 (cloudflared가 클러스터 안에서 접속하므로 충분) |
| cloudflared 연결 대상 | Envoy Service 이름은 기본적으로 해시가 붙어 자동 생성됨. EnvoyProxy 설정으로 이름을 고정해 두는 게 좋음 (정확한 필드는 설치할 버전 문서로 확인) |
| Cilium과 함께 쓸 때 | Cilium에도 Gateway와 Ingress 기능이 있어 끄지 않으면 Gateway 컨트롤러가 둘이 됨. Cilium 설치 때 `gatewayAPI.enabled=false`, `ingressController.enabled=false` |
| Gateway API CRD | Envoy Gateway Helm 차트가 CRD를 함께 설치함. CRD를 관리하는 곳은 한 군데로만 정함 |
| ArgoCD 동기화 | Gateway API와 Envoy Gateway CRD는 크기가 커서 일반 apply로는 "annotation too long" 오류 발생. 해당 Application에 `ServerSideApply=true` 설정 |
| 실제 사용자 IP | Cloudflare Tunnel을 거치면 Envoy에는 cloudflared의 IP만 보임. 로그나 rate limit에 실제 IP가 필요하면 ClientTrafficPolicy에서 `CF-Connecting-IP` 헤더를 신뢰하도록 설정 |

### 여러 프로젝트에 맞는 구성

- 공용 Gateway 하나를 `platform/gateway/`에 두고 `*.choimory.dev` 같은 리스너를 만듦. 각 프로젝트 네임스페이스는 HTTPRoute만 작성
- Gateway의 `allowedRoutes`로 어떤 네임스페이스가 Route를 붙일 수 있는지 제한 가능
- Envoy 프록시도 하나만 떠서 자원이 늘지 않음
- 운영(Gateway)과 개발(HTTPRoute)의 역할을 나누는 Gateway API 설계와도 맞음

## projects로 감싸기

- 확정

## 환경: local, dev, prod

- dev와 prod를 서버에 배포. Kustomize overlay는 `dev`와 `prod`

### 네임스페이스 이름

- `프로젝트-환경` 규칙대로 하면 `choimory-dev-dev`, `choimory-dev-prod`
- 프로젝트 이름에 dev가 들어 있어 어색하지만, 규칙을 일정하게 지켜야 ApplicationSet으로 자동 생성하기 쉬워 그대로 두는 것을 추천

### member-api 수정 범위

- `stg` 프로필을 새로 만들 필요 없음
- 대신 `application-dev.yml`의 CORS에 있는 `https://dev.choimory.dev`를 `https://staging.choimory.dev`로 변경 필요 (앱 레포 작업)

### docker-compose용 프로필

- 이전에는 dev를 docker-compose용으로 고려했지만 이제 dev는 서버용
- 로컬 docker-compose는 `local` 프로필로 돌릴지 결정 필요

## 도메인

| 환경 | front | member-api (브라우저가 직접 호출) |
| --- | --- | --- |
| dev | `staging.choimory.dev` | `staging-api.choimory.dev` |
| prod | `choimory.dev` | `api.choimory.dev` |

- API 도메인은 `api.staging.choimory.dev`가 아니라 `staging-api.choimory.dev`처럼 한 단계로 두는 것을 추천
- SSL 인증서 문제: Cloudflare 무료 인증서(Universal SSL)는 `*.choimory.dev` 한 단계까지만 적용. `api.staging.choimory.dev` 같은 두 단계 서브도메인은 유료 인증서(Advanced Certificate Manager) 필요
- 경로 방식은 충돌: 같은 도메인의 `/api` 경로로 member-api를 보내는 방법도 있지만, front에 이미 Next 자체 API 경로(`/api/auth/login`, `logout`, `reissue`)가 있어 겹침. 그래서 서브도메인으로 나눔

## 남은 결정 사항

- [x] Ingress: Envoy Gateway
- [x] 디렉토리 구조: `projects/`로 감싸기
- [x] 환경: local, dev, prod (dev와 prod를 서버에 배포, dev 도메인은 `staging.choimory.dev`)
- [ ] 네임스페이스를 `choimory-dev-dev`로 둘지
- [ ] 로컬 docker-compose는 `local` 프로필로 갈지
- [ ] API 도메인 (`staging-api.choimory.dev`, `api.choimory.dev`)
- [ ] front 빌드 방식: 환경별로 따로 빌드할지, 런타임 주입으로 바꿀지
- [ ] dev → prod 승격 방식
- [ ] DB와 Redis: 혼합 방식(공용 오퍼레이터 + 프로젝트·환경별 인스턴스)으로 갈지

---

# 추가2

- 네임스페이스 choimory-dev-dev
- 로컬 docker compose는 local 프로필
- api 도메인 api.choimory.dev, staging.api.choimory.dev
- dev -> prod 승격방식
- 프론트 빌드 방식이랑 DB 방식은 추가 고려 필요

---

# 추가2 분석

> plan2의 추가2 확인

## 답변

- 추가2의 네임스페이스, 로컬 프로필, API 도메인, 승격 방식, front 빌드 방식, DB 방식을 분석함
- 참고: 앱 레포 브랜치 구성(`dev`, `stg`, `main`), member-api의 Redis 사용처(`JwtRepository` 등)

## 확정된 것

- 네임스페이스: `choimory-dev-dev`, `choimory-dev-prod`
- 로컬 docker-compose: `local` 프로필

## API 도메인: staging.api.choimory.dev 확인 필요

- `staging.api.choimory.dev`는 두 단계 서브도메인 (`api` 아래에 `staging`이 붙은 형태)
- Cloudflare 무료 인증서(Universal SSL)는 `*.choimory.dev` 한 단계까지만 적용되어 HTTPS 인증서 오류 발생
- 쓰려면 유료 인증서(Advanced Certificate Manager, 월 약 $10) 필요
- 이름을 비슷하게 유지하면서 한 단계로 만드는 대안

| 환경 | front | API (한 단계) |
| --- | --- | --- |
| dev | `staging.choimory.dev` | `staging-api.choimory.dev` 또는 `api-staging.choimory.dev` |
| prod | `choimory.dev` | `api.choimory.dev` |

- front 방식(아래 c안)에 따라 API 도메인 자체가 필요 없어질 수도 있어 함께 결정

## dev → prod 승격 방식

| 방식 | 흐름 | 장점 | 단점 |
| --- | --- | --- | --- |
| A. 브랜치 기준 | `dev` push → 이미지 빌드 → dev overlay 갱신<br>`main` merge → 이미지 다시 빌드 → prod overlay 갱신 | 지금 브랜치 구조와 그대로 맞음. 가장 단순함. front를 환경별로 빌드해도 자연스러움 | prod 이미지가 dev에서 검증한 이미지와 같은 파일은 아님 (코드는 같음) |
| B. 이미지 승격 | `dev` push → 이미지 빌드 → dev 배포<br>릴리스 태그(`v1.2.0`) → dev에서 검증한 같은 이미지를 prod overlay에 반영 | 검증한 이미지를 그대로 운영에 올리는 실무 표준 방식 | front가 환경별 빌드면 front는 이 방식을 쓸 수 없음. 태그 관리 필요 |
| C. gitops 레포 PR | A나 B의 마지막 단계에서 prod 태그를 바로 커밋하지 않고 gitops 레포에 PR → 승인하면 반영 | 운영 배포 전에 사람이 확인하는 단계가 생김 | 배포할 때마다 PR 승인 필요 |

- A와 B는 "무엇을 올리는지", C는 "언제 올리는지"를 정하는 것이라 함께 쓸 수 있음. 예: A + C는 `main` merge 후 prod 반영 PR이 생기고 승인하면 배포
- 프로필이 dev와 prod 두 개로 줄었으니 `stg` 브랜치는 정리해도 될 것으로 보임
- 추천: 처음에는 A + C로 시작하고, front 빌드 방식이 정리되면 B로 전환

## front 빌드 방식

| 방식 | 내용 | 코드 수정 | 효과 |
| --- | --- | --- | --- |
| a. 환경별 빌드 | 브랜치마다 다른 `NEXT_PUBLIC_` 값으로 빌드 | 없음 | 승격 방식 A와 자연스럽게 맞음. 같은 이미지를 올리는 방식(B)은 불가 |
| b. 런타임 설정 주입 | 서버가 시작할 때 환경변수를 읽어 브라우저에 설정값을 내려줌 (layout에서 주입 등) | 작음 | 이미지 하나로 두 환경 사용 가능. 브라우저는 여전히 API 도메인을 직접 호출 |
| c. 전부 BFF 경유 | 브라우저는 Next 서버(`/api/*` Route Handler)만 부르고, Next 서버가 클러스터 내부 주소로 member-api 호출 | 큼 | `NEXT_PUBLIC_` 불필요. member-api를 외부에 노출하지 않아도 되므로 API 도메인과 CORS 불필요. 토큰을 httpOnly 쿠키로만 다룰 수 있어 보안상 유리 |

- front에는 이미 인증 쪽에 BFF 구조(`/api/auth/*`, `bffResponse.ts`)가 있어 c는 이 구조를 전체 API로 넓히는 방향
- 추천: 지금은 a로 코드 수정 없이 배포 구조를 먼저 완성. 장기적으로는 c를 고려하되, front 작업이므로 이 gitops 작업과는 별도 과제로 분리
- c로 간다면 API 도메인은 외부 클라이언트가 생길 때까지 불필요

## DB와 Redis 방식

### PostgreSQL 선택지

| | CloudNativePG (추천) | 공식 이미지 StatefulSet |
| --- | --- | --- |
| 설치 | 오퍼레이터 + `Cluster` 리소스 | StatefulSet, Service, PVC를 직접 작성 |
| 비밀번호 | 앱용 계정 Secret을 자동 생성 (`<클러스터명>-app`에 host, user, password, uri). member-api가 이 Secret을 바로 참조하면 되므로 DB 비밀번호를 Sealed Secret으로 관리할 필요 없음 | 직접 만들고 Sealed Secret으로 관리 |
| 백업 | 오브젝트 스토리지(S3 호환)로 자동 백업과 시점 복구 지원 | `pg_dump` CronJob을 직접 작성 |
| 업그레이드 | 선언적으로 버전만 바꾸면 됨 | 수동 |
| 학습 | 오퍼레이터 패턴 경험 (실무 가치 높음) | 쿠버네티스 기본 리소스 경험 |

### 백업 저장소

- CloudNativePG의 백업은 S3 호환 저장소가 필요
- Cloudflare R2: 무료 10GB, 서버 밖에 있어 진짜 외부 백업이 됨. 이미 Cloudflare를 쓰고 있으므로 추천
- MinIO: 2025년에 무료판 배포 방식이 바뀌어 새로 도입하기엔 애매함
- 간단한 방법: `pg_dump` CronJob → `/data/backup` → Kopia로 외부 전송 (노션 4차 계획과 연결)

### Redis

- member-api는 Redis에 Refresh Token과 Access Token 블랙리스트를 저장함
- 영속 저장을 끄면 재시작 시 모든 사용자가 로그아웃되고, 로그아웃한 토큰이 만료(30분) 전까지 다시 유효해짐 → AOF 영속 저장을 켜고 PVC를 붙이는 것을 추천
- 이미지는 공식 `redis` 이미지 StatefulSet (Valkey도 대안). Bitnami 차트와 이미지는 2025년 정책 변경 때문에 피함
- Redis 비밀번호와 JWT secret은 자동 생성이 안 되므로 Sealed Secrets로 관리

### 그 외

- Flyway는 앱 시작 시 실행되며, HPA로 파드가 여러 개 떠도 잠금 테이블로 한 번만 실행되게 처리되어 문제없음
- local-path 저장 경로 `/data/dev/choimory-dev/k8s`는 클러스터 전체 설정. 여러 프로젝트를 담을 거라면 `/data/dev/k8s`처럼 프로젝트 이름이 없는 경로가 맞음 (노션 4차에서도 메모해 둔 사항)

## 남은 결정 사항

- [x] 네임스페이스: `choimory-dev-dev`, `choimory-dev-prod`
- [x] 로컬 docker-compose: `local` 프로필
- [ ] API 도메인: 한 단계 서브도메인으로 바꿀지, 유료 인증서를 쓸지, c(BFF) 방식으로 없앨지
- [ ] 승격 방식: A + C로 시작할지
- [ ] front 빌드: 지금은 a로 가고 c는 별도 과제로 둘지
- [ ] DB: CloudNativePG로 갈지, 백업 저장소는 R2로 할지
- [ ] Redis: 공식 이미지 + AOF로 갈지
- [ ] local-path 경로를 `/data/dev/k8s`로 바꿀지
- [ ] `stg` 브랜치 정리 여부

---

# 추가3

- 한단계 서브도메인으로
- DB, Redis를 프로젝트별로 생성하여 관리할지, 서버 전체에(k8s 클러스터에) 하나 놓고 모든 프로젝트가 사용할지 고민중
  - 이때 DB, Redis를 프로젝트별로 생성하면 DB, Redis가 많이 생성될텐데 램 많이 먹지 않나 싶어서 걱정
  - 그냥 서버에 Postgres 하나 쓰면서 Database로 구분하면 괜찮지 않을까
  - Redis도 database로 구분하고
- 승격은 gitops를 말하는건가? main 브랜치 하나만 써도 되지 않나? 어차피 overlays로 나눌꺼니까
- 프론트 빌드는 젠킨스 쓸때는 그냥 profile별로 job 만들어서 명령어로 빌드 방식을 구분해서 docker build 하여 dev 이미지, prod 이미지를 생성했는데 github actions는 다른가?
- local-path 경로 변경
- stg 브랜치는 git flow에서 배포전 브랜치로 사용할거고, 이후 stg 서버도 올릴거면 stg 브랜치를 staging 서버로 사용할거임 (지금은 prod 제외하면 dev 서버만 있지만) 

---

# 추가3 분석

> 추가3 확인

## 답변

- 추가3의 API 도메인, DB·Redis 공유 여부, 승격 방식, front 빌드, local-path 경로, stg 브랜치 항목을 분석함

## API 도메인: 한 단계 서브도메인

- 확정. 이름은 아래 "stg 브랜치 유지와 도메인 충돌"과 함께 결정

## DB와 Redis: 프로젝트별 vs 하나를 공유

### 실제 메모리 사용량

| 인스턴스 | 작은 서비스 기준 유휴 메모리 |
| --- | --- |
| PostgreSQL (CloudNativePG 관리 프로세스 포함) | 약 150~250MB (`shared_buffers`를 낮추면 더 줄어듦) |
| Redis | 약 10~30MB (데이터 크기만큼 늘어남) |

| 프로젝트 수 | 프로젝트·환경별 인스턴스 | 하나만 공유 |
| --- | --- | --- |
| 1개 (지금) | PostgreSQL 2개 + Redis 2개 ≈ 0.5GB | ≈ 0.25GB |
| 5개 | PostgreSQL 10개 + Redis 10개 ≈ 2.5GB | ≈ 0.3GB |

- 프로젝트가 5개로 늘어도 32GB 중 2GB 남짓
- 메모리를 실제로 쓰는 건 PostgreSQL이고, Redis는 나눠도 거의 차이 없음

### 하나를 공유할 때 걸리는 점

- dev와 prod가 같은 인스턴스에 있게 됨. dev에서 무거운 쿼리를 실험하거나 실수로 인스턴스를 재시작하면 prod도 같이 영향받음
- 복구 단위: CloudNativePG의 시점 복구는 인스턴스 전체 단위라 프로젝트 하나만 되돌리기 어려움. DB별 `pg_dump`를 따로 받아야 함
- Redis의 database 번호 분리는 약함
  - 번호(0~15)마다 따로 비밀번호를 걸 수 없어 어느 앱이든 다른 번호의 데이터를 읽을 수 있음
  - `FLUSHALL` 한 번이면 전체가 지워짐
  - member-api는 지금 database 번호 설정이 없어(기본 0번) `spring.data.redis.database` 설정 추가 필요

### 중간안: 환경별로 공유 (추천)

```txt
PostgreSQL: dev용 1개, prod용 1개 → 그 안에서 프로젝트별 database와 계정으로 구분
Redis:      프로젝트·환경별 (어차피 가벼움)
```

- 프로젝트가 늘어도 PostgreSQL은 항상 2개라 메모리 걱정 없음
- dev와 prod는 분리되어 dev 실험이 prod에 영향을 주지 않음
- 공용 DB는 프로젝트 소속이 아니므로 `platform/` 쪽(예: `platform/postgres/dev`, `platform/postgres/prod`)에서 관리. "infra 루트 디렉토리" 아이디어와도 맞음
- 프로젝트를 추가할 때는 그곳에 database와 계정을 선언 (CloudNativePG의 `Database` 리소스와 managed roles)
- 대신 바뀌는 점: 계정 비밀번호 Secret이 DB 쪽 네임스페이스에 생기므로, 앱 네임스페이스에도 같은 비밀번호를 Sealed Secret으로 따로 넣어줘야 함. 프로젝트별 인스턴스였다면 자동 생성 Secret을 바로 쓸 수 있던 장점이 사라짐
- 완전히 하나로 가는 것도 가능은 하지만, dev와 prod가 같은 인스턴스를 공유한다는 점은 감수해야 함

## 승격과 gitops 브랜치

- gitops 레포는 `main` 브랜치 하나만 쓰고 환경은 overlay 디렉토리로 나누는 게 표준 방식. 환경을 브랜치로 나누는 건 안티패턴
- 앞서 말한 A, B, C는 gitops 레포가 아니라 앱 레포에서 어떤 이벤트가 어느 overlay를 갱신하는지에 대한 이야기

```txt
앱 레포 dev 브랜치 push  → Actions → gitops 레포 main의 overlays/dev  태그 갱신 → dev 배포
앱 레포 main 브랜치 push → Actions → gitops 레포 main의 overlays/prod 태그 갱신 → prod 배포
```

- 승격이란 결국 "prod overlay의 이미지 태그를 무엇으로 언제 바꾸느냐"
- C안(PR)은 prod overlay를 바꿀 때 gitops 레포의 `main`에 바로 커밋하지 않고 PR을 올려 승인 후 merge하는 방식. 이것도 브랜치는 `main` 하나

## front 빌드: Jenkins와 Actions 비교

- 같음. Jenkins에서 profile별 job을 만들어 `docker build`에 빌드 인자를 다르게 줬던 것을 그대로 Actions로 옮기면 됨

| Jenkins | GitHub Actions |
| --- | --- |
| profile별 job | 브랜치별 트리거 (또는 workflow 하나에서 브랜치로 분기) |
| job 파라미터, Credentials | GitHub Environments (`dev`, `prod`)마다 변수와 Secret을 따로 둠 |
| `docker build --build-arg ...` | 같음 (`docker/build-push-action`의 `build-args`) |

```yaml
# 예시 개념: 브랜치에 따라 환경을 고르고, 그 환경의 변수로 빌드
on:
  push:
    branches: [dev, main]
jobs:
  build:
    environment: ${{ github.ref_name == 'main' && 'prod' || 'dev' }}
    steps:
      - uses: docker/build-push-action@v6
        with:
          build-args: NEXT_PUBLIC_MEMBER_API_BASE_URL=${{ vars.MEMBER_API_BASE_URL }}
          tags: ghcr.io/choimory/choimory-dev-front:${{ vars.ENV }}-${{ github.sha }}
```

- 결과적으로 `dev-<sha>`, `prod-<sha>` 이미지가 따로 생김. 추천했던 a안(환경별 빌드)이 바로 이 방식

## local-path 경로 변경

- `/data/dev/k8s`로 확정
- 서버에 디렉토리를 만들어야 하고, 노션 4차의 디렉토리 구조도 나중에 맞춰 수정 필요

## stg 브랜치 유지와 도메인 충돌

| 앱 레포 브랜치 | 지금 | 나중에 (stg 서버 추가 시) |
| --- | --- | --- |
| `dev` | dev 서버 배포 (`overlays/dev`) | 동일 |
| `stg` | 배포 없음 (배포 전 확인용) | stg 서버 배포 (`overlays/stg`, `choimory-dev-stg` 네임스페이스) |
| `main` | prod 배포 | 동일 |

- gitops 구조는 나중에 `overlays/stg`만 추가하면 되므로 지금 설계를 바꿀 필요 없음
- 다만 도메인 충돌: 지금 dev 서버에 `staging.choimory.dev`를 붙이면 나중에 진짜 stg 서버가 생겼을 때 쓸 이름이 없음. 그때 dev 서버 도메인을 바꾸면 CORS, 빌드 인자, Tunnel, HTTPRoute를 전부 수정해야 함

| 안 | dev 서버 | 나중의 stg 서버 | prod |
| --- | --- | --- | --- |
| ① 지금 계획 그대로 | `staging.choimory.dev` | 이름 충돌 → 그때 변경 | `choimory.dev` |
| ② dev에 다른 이름 사용 | `beta.choimory.dev` (또는 `test`, `sandbox`) | `staging.choimory.dev` | `choimory.dev` |

- API는 한 단계 규칙에 따라 `beta-api.choimory.dev`, `staging-api.choimory.dev`, `api.choimory.dev`처럼 붙임

## 남은 결정 사항

- [x] API 도메인: 한 단계 서브도메인
- [x] local-path 경로: `/data/dev/k8s`
- [x] `stg` 브랜치: 유지 (git flow 배포 전 브랜치, 이후 stg 서버용)
- [x] gitops 레포: `main` 브랜치 하나, 환경은 overlay로 구분
- [x] front 빌드: 브랜치별 Actions 빌드로 환경별 이미지 생성 (a안)
- [ ] dev 서버 도메인: ① `staging.choimory.dev` 그대로 / ② 다른 이름 사용 (이름 결정 필요)
- [ ] DB: 환경별 공유(PostgreSQL dev용 1개, prod용 1개) / 완전히 하나로
- [ ] Redis: 프로젝트·환경별로 둘지 (메모리 차이가 거의 없어 추천)
- [ ] 승격: `main` 브랜치는 자동 배포할지, PR 승인 후 배포할지

---

# 추가4

- API 도메인
  - 페이지 도메인: choimory.dev, dev.choimory.dev, staging.choimory.dev
  - API 도메인: api.choimory.dev, dev-api.choimory.dev, staging-api.choimory.dev
- 프론트 빌드: A안 (dockerfile 이용해서 dev 이미지, prod 이미지 만들고, gitops에도 dev 이미지 태그, prod 이미지 태그 수정 후 커밋)
- DB, Redis
  - 둘다 개발용 하나, Redis 하나 쓰자
- 승격: 자동배포
- 그리고 생각의 변화가 생겼어
  - staging, prod로 배포하자. 브랜치도 stg, main으로 잡고 dev 서버는 추후 필요할때 하자 -> 이거 검토해줘 

---

# 추가4 분석

> 추가4 확인

## 답변

- 추가4의 도메인, front 빌드, DB·Redis, 승격 확정 사항을 정리하고, 환경 변경안(staging, prod 배포, dev 서버 보류)을 검토함

## 도메인 확정

| 환경 | 페이지 | API |
| --- | --- | --- |
| prod | `choimory.dev` | `api.choimory.dev` |
| staging | `staging.choimory.dev` | `staging-api.choimory.dev` |
| dev (나중에) | `dev.choimory.dev` | `dev-api.choimory.dev` |

- 모두 한 단계 서브도메인이라 무료 인증서로 해결되고, 이전의 이름 충돌도 해소됨

## front 빌드와 승격 확정

- front 빌드: 환경별로 이미지를 따로 만들고(`stg-<sha>`, `prod-<sha>`), Actions가 gitops 레포의 해당 overlay 태그를 수정해 커밋
- 승격: 자동 배포. gitops 레포에 PR을 올리는 C안은 사용하지 않음

## 환경 변경안 검토: staging, prod 배포와 dev 서버 보류

- 찬성. 지금 상황에 오히려 더 잘 맞음

### 좋은 점

- git flow와 바로 맞음

```txt
feature → dev (로컬 docker-compose로 확인)
        → stg  push → staging 자동 배포 (staging.choimory.dev)
        → main merge → prod 자동 배포 (choimory.dev)
```

- 운영 전 확인 단계가 자연스럽게 생김. `stg`에서 확인하고 `stg → main` PR을 merge하는 순간이 곧 prod 반영 시점이라, 승격을 자동 배포로 해도 사람이 확인하는 단계가 앱 레포 PR에 남아 있음
- dev 서버는 나중에 쉽게 추가 가능. `overlays/dev`, `choimory-dev-dev` 네임스페이스, `dev` 브랜치 트리거만 추가하면 되며 지금 구조를 바꿀 필요 없음
- 혼자 개발하는 단계에서는 dev 통합 확인을 로컬 docker-compose로 해도 충분

### 함께 바뀌는 점

| 항목 | 변경 |
| --- | --- |
| member-api 프로필 | `stg` 프로필 다시 필요. `application-stg.yml`을 만들고 CORS에 `https://staging.choimory.dev`. 기존 `dev` 프로필은 나중의 dev 서버용으로 남기고 CORS는 `https://dev.choimory.dev` 유지 (앱 레포 작업) |
| 이름 통일 | 브랜치와 프로필이 `stg`이므로 overlay `stg`, 네임스페이스 `choimory-dev-stg`, 이미지 태그 `stg-<sha>`로 짧은 이름 통일 추천. 도메인만 `staging` |
| Actions 트리거 | `stg` 브랜치는 staging, `main` 브랜치는 prod. `dev` 브랜치는 지금은 트리거하지 않음 |
| 이미지 태그 | `main`의 merge commit SHA는 `stg`의 SHA와 다름. 환경별로 따로 빌드하는 방식이라 문제없음 |
| hotfix | git flow에서 hotfix는 `main`에 바로 들어가 staging을 거치지 않고 prod에 배포됨. 이후 `stg`와 `dev`에 반영(back-merge) 필요 |

## DB와 Redis 해석 확인

- "둘다 개발용 하나, Redis 하나"가 두 가지로 읽혀 확인 필요

| 해석 | PostgreSQL | Redis |
| --- | --- | --- |
| ① 환경별로 하나씩 | staging용 1개 + prod용 1개 (프로젝트는 database로 구분) | staging용 1개 + prod용 1개 (프로젝트는 database 번호로 구분) |
| ② 전체에서 하나씩 | 전체 1개 (환경과 프로젝트 모두 database로 구분) | 전체 1개 (database 번호로 구분) |

- ②의 경우 감수해야 할 점
  - staging과 prod가 같은 인스턴스를 씀. staging에서 무거운 테스트를 하거나 재시작하면 prod가 영향받음
  - Redis 키 충돌 위험. member-api는 지금 Redis database 번호 설정이 없어 0번을 씀. staging과 prod가 같은 번호를 쓰면 Refresh Token 키가 섞이므로 `spring.data.redis.database`를 환경별로 반드시 다르게 줘야 함
  - 한쪽 비밀번호로 다른 database 번호도 읽을 수 있어 격리가 약함
- 메모리 차이는 PostgreSQL 하나(약 200MB) 정도라 ① 추천. ②가 의도라면 그대로 진행 가능

## 남은 결정 사항

- [x] 도메인: prod `choimory.dev`/`api.choimory.dev`, staging `staging.choimory.dev`/`staging-api.choimory.dev`, dev(나중에) `dev.choimory.dev`/`dev-api.choimory.dev`
- [x] front 빌드: 환경별 이미지 빌드 후 gitops overlay 태그 커밋
- [x] 승격: 자동 배포
- [x] 서버 배포 환경: staging(`stg` 브랜치), prod(`main` 브랜치). dev 서버는 추후
- [ ] DB와 Redis: ① 환경별로 하나씩 / ② 전체에서 하나씩
- [ ] 이름 통일: overlay, 네임스페이스, 이미지 태그를 `stg`로 할지 `staging`으로 할지

---

# 추가5

- api에 application-stg.yml 작성해줘
- DB, Redis: 전체에서 환경별로 하나씩을 생각하고 있었음. 별론가? 프로젝트별로 환경별로 하나씩 할까? 이건 스펙상 무린가
- overlay, 네임스페이스, 이미지 태그, 브랜치 모두 stg이고 도메인만 staging임. 아니면 그냥 모든걸 stage로 통일할까.. 근데 이미 브랜치가 stg라..
- api 도메인 말인데 api-gateway가 없고 향후 api를 계속 추가할거임. 그럼 stage-member-api.choimory.dev, stage-member-grpc.choimory.dev, member-api.choimory.dev(운영) 이런식으로 다뤄야하나

---

# 추가5 분석

> 추가5 확인

## 답변

- 추가5의 application-stg.yml 작성 요청, DB·Redis 구성 단위, 이름 통일, API 도메인 방식을 검토함

| 항목 | 추천 |
| --- | --- |
| application-stg.yml | 아래 안으로 작성(허락 후 진행) |
| DB, Redis | 환경별로 하나씩 공용으로 두고, 그 안을 프로젝트별로 나누기 |
| 이름 | 내부는 `stg`, 도메인만 `staging`인 지금 상태 유지 |
| API 도메인 | `api.choimory.dev/<서비스>` 경로 방식. Envoy Gateway가 API 게이트웨이 역할. gRPC는 공개하지 않음 |

## application-stg.yml 작성안

member-api 저장소(`choimory-dev-member-api`)를 확인한 결과, 파일 하나만 추가해서는 안 되고 두 곳을 고쳐야 함

### application-stg.yml 새로 작성

- 설정은 운영과 비슷하게, 로그만 조금 더 자세히
  - 운영과 같은 설정: `show-sql: false`, `format_sql: false`, `ddl-auto: none`
  - 로그 레벨: `root: info`, `dev.choimory: debug`, `org.hibernate.SQL: info`
  - CORS 허용 주소: `https://staging.choimory.dev`

### application.yml 프로필 묶음에 추가

```yaml
group:
  stg: stg, flyway
```

- 이 줄이 없으면 `stg` 프로필로 실행해도 flyway가 함께 켜지지 않음. DB 테이블 구조 변경(마이그레이션)이 적용되지 않은 채로 앱이 뜸
- 앱 저장소이므로 커밋은 사용자가 직접 진행

## DB와 Redis: 환경별 공용 vs 프로젝트별·환경별

- "전체에서 환경별로 하나씩"은 추가4 분석의 ①(stg에 하나, prod에 하나)과 같은 뜻이며 추천하는 방식
- 프로젝트별·환경별로 하나씩 두는 안도 서버 사양으로는 가능
  - 대략 필요한 메모리: PostgreSQL 하나에 약 300MB~1GB, Redis 하나에 수십 MB
  - 프로젝트 3개라면 PostgreSQL 6개, Redis 6개로 3~6GB 정도. Jellyfin, Immich와 같이 돌려도 32GB 안에 들어감
  - 문제는 관리 부담. 백업 설정, 업그레이드, 모니터링 대상이 프로젝트 수만큼 늘어남

### 추천 구성

- PostgreSQL
  - 환경별로 CloudNativePG 클러스터 1개씩
  - 그 안에서 프로젝트나 서비스마다 database와 user를 따로 생성(CloudNativePG의 `Database` 리소스로 선언 가능)
  - 서비스별로 DB를 나누는 원칙도 지킬 수 있음
- Redis
  - 환경별로 1개씩
  - 프로젝트끼리 키가 겹치지 않게 ACL 사용자로 키 접두어를 제한(예: member는 `~member:*` 키만 사용)
  - member-api는 이미 `redis-username`을 받고 있어 그대로 사용 가능
- 특정 프로젝트의 부하가 커지면 그때 그 프로젝트만 따로 분리

### 디렉토리 위치 변경

- DB와 Redis가 여러 프로젝트의 공용이 되므로 `projects/choimory-dev/data/`가 아니라 `platform/data/`(또는 `shared/`)에 둠
- 네임스페이스도 `data-stg`, `data-prod` 같은 공용 이름 사용

## 이름 통일: stg와 staging

- 지금 그대로(내부는 `stg`, 도메인만 `staging`) 두는 것을 추천
  - 도메인은 사람이 보는 이름이라 읽기 쉬운 쪽이 좋고, overlay, 네임스페이스, 태그, 브랜치는 짧게 쓰는 것이 흔한 방식
  - 섞어 쓸 때 생기는 문제는 CORS 주소와 front 빌드 변수에 도메인을 적을 때 헷갈릴 수 있다는 정도. README에 환경 대응표 하나만 두면 충분
- 전부 통일하고 싶다면 도메인을 `stg.choimory.dev`, `stg-api.choimory.dev`로 바꾸는 쪽이 비용이 가장 적음
- `stage`로 통일하는 것은 브랜치 이름까지 바꿔야 해서 얻는 것이 없음

## API 도메인: API 게이트웨이 없이 API를 계속 추가할 때

- Envoy Gateway가 이미 API 게이트웨이 역할을 할 수 있으므로 두 가지 방식이 가능

| | A. 서비스별 서브도메인 | B. 경로 방식 (추천) |
| --- | --- | --- |
| 예시 | `member-api.choimory.dev`, `staging-member-api.choimory.dev` | `api.choimory.dev/member/...`, `staging-api.choimory.dev/member/...` |
| 서비스 추가 시 할 일 | HTTPRoute 작성, CORS에 주소 추가 (cloudflared에서 `*.choimory.dev`를 Envoy로 보내두면 DNS는 매번 추가하지 않아도 됨) | HTTPRoute에 경로 규칙 하나 추가 |
| CORS, 쿠키 | 서비스마다 다른 주소라 각각 관리 | API 주소가 하나라 단순함 |
| 나중에 붙일 공통 기능 | 서비스마다 각각 적용 | 인증(SecurityPolicy, JWT 검증), 요청 제한(BackendTrafficPolicy)을 한 곳에 적용 |
| 주의할 점 | 이름이 길어짐. 와일드카드 인증서는 한 단계 서브도메인만 덮으므로 하이픈으로 연결해야 함 | 앱이 `/member` 경로 접두어를 알아야 함. HTTPRoute의 `URLRewrite`로 접두어를 떼고 넘기면 앱 코드는 고치지 않아도 됨 |

### gRPC

- `member-grpc`가 서비스끼리 호출하는 용도라면 공개 도메인이 필요 없음
  - 클러스터 안에서 `member-api.choimory-dev-prod.svc:9090` 같은 내부 주소로 바로 호출
  - 외부에 열지 않으니 보안상으로도 적합
- 브라우저나 외부 클라이언트가 직접 gRPC를 써야 한다면 Cloudflare Tunnel을 거친 gRPC 동작을 따로 확인해야 함
  - Cloudflare에서 gRPC 활성화, 원본 서버 쪽 HTTP/2 설정 필요
  - 직접 확인한 내용이 아니므로 필요할 때 검증

## 남은 결정 사항

- [ ] application-stg.yml 내용(로그 레벨 포함) 확인 및 작성 허락
- [ ] DB와 Redis: 환경별로 하나씩 공용으로 두고, 그 안을 프로젝트별로 나눌지
- [ ] 이름: 내부는 `stg`, 도메인만 `staging`으로 유지할지
- [ ] API 도메인: B(경로 방식)로 할지, gRPC는 내부 전용으로 할지

---

# 추가6

- 로그레벨은 계속 조정하면되니까 일단 너의 추천대로 해줘
- 프로젝트별로 하는게 낫겠다 왜냐면 이미 /data 아래에 프로젝트별로 디렉토리를 구성했네. 이때 서버에 어느정도 영향이 갈지 확인해줘
- 내부는 stg, 도메인만 staging으로 하자
- envoy gateway로 api gateway처럼 쓰자. 그럼 api.choimory.dev, staging-api.choimory.dev로 되는거네

---

# 추가6 분석

> 추가6 확인

## 답변

- 추가6의 확정 사항(로그 레벨, 이름, API 게이트웨이, 프로젝트별 DB·Redis)을 정리하고, 프로젝트별 DB·Redis가 서버에 주는 영향을 검토함

## 확정 사항

- 로그 레벨: 추천안(`root: info`, `dev.choimory: debug`, `org.hibernate.SQL: info`)으로 확정
- 이름: 내부(overlay, 네임스페이스, 이미지 태그, 브랜치)는 `stg`, 도메인만 `staging`
- API 게이트웨이: Envoy Gateway가 API 게이트웨이 역할
  - 도메인은 `api.choimory.dev`, `staging-api.choimory.dev` 두 개뿐이고 서비스는 경로로 구분(예: `api.choimory.dev/member/...`)
  - 서비스가 늘어나도 도메인, DNS, CORS는 그대로이고 HTTPRoute 경로 규칙만 추가
- DB와 Redis: 프로젝트별·환경별로 따로 둠
  - 추가5의 `platform/data/`로 옮기는 안은 취소하고, `projects/<project>/data/`에 두는 원래 구조 유지

## 프로젝트별 DB와 Redis의 서버 영향

### 프로젝트 하나당 늘어나는 것

| 구성 요소 | 개수 | 하나당 메모리(실사용 예상) | 합계 |
| --- | --- | --- | --- |
| PostgreSQL (CloudNativePG 인스턴스 1개) | stg, prod 2개 | 약 200~400MB | 약 0.4~0.8GB |
| Redis | stg, prod 2개 | 약 10~50MB | 약 0.1GB |
| DB와 Redis 합계 | | | 약 0.5~0.9GB |

- CloudNativePG 오퍼레이터는 프로젝트 수와 상관없이 하나(약 100MB)
- CPU는 평소 거의 쓰지 않음. 체크포인트와 R2 백업 때 잠깐 올라감
- 디스크는 프로젝트당 수 GB 정도(DB 데이터, WAL은 최대 1GB 정도로 제한 가능, Redis AOF 포함)

### 실제로 더 큰 것은 앱

| 구성 요소 | 프로젝트 하나당 (stg + prod) |
| --- | --- |
| Spring Boot API (하나에 약 400~700MB) | API 하나당 약 0.8~1.4GB |
| Next.js front (하나에 약 150MB) | 약 0.3GB |

- DB보다 Spring Boot API가 메모리를 더 많이 씀. API가 늘어날수록 이것이 먼저 부담이 됨

### 32GB 전체 예산 (대략)

| 영역 | 예상 |
| --- | --- |
| OS와 Docker Compose 앱 (Immich ML 포함, Jellyfin 트랜스코딩 피크 포함) | 약 5~8GB |
| k8s 기반 (control plane, Cilium, ArgoCD, Envoy, 오퍼레이터) | 약 3~4GB |
| choimory-dev (API 1개 + front + DB + Redis, stg와 prod) | 약 2~3GB |
| 합계 | 약 10~15GB |

- 지금 규모로는 넉넉함
- 프로젝트를 2~3개 더 추가하거나 API를 3~4개 늘려도 20GB 안팎

### 영향을 줄이는 방법

- stg는 가볍게
  - stg의 PostgreSQL에는 R2 백업을 붙이지 않음
  - stg의 Redis는 AOF(디스크 저장)를 끔
  - stg 앱은 JVM 힙 상한을 낮게 잡음
- requests와 limits 지정: DB, Redis, 앱 모두에 지정해서 한 프로젝트가 메모리를 독차지하지 못하게 함
- kubelet `systemReserved` 설정 (중요)
  - kubelet은 Docker Compose 앱이 쓰는 메모리를 모름
  - 그대로 두면 k8s는 32GB를 전부 쓸 수 있다고 보고 Pod를 계속 배치함. Immich나 Jellyfin과 메모리를 다투다 OOM이 날 수 있음
  - `systemReserved`를 6~8GB 정도로 잡아 Compose 앱 몫을 미리 빼둠
- 백업 시간 분산: 프로젝트마다 prod 백업 시간을 다르게 잡아 동시에 돌지 않게 함
- 디스크 I/O: Immich, Jellyfin과 같은 디스크를 쓰면 대용량 작업 중 DB 쓰기가 느려질 수 있음. 지금 규모에서는 문제없고, 나중에 느려지면 확인할 부분

### 디렉토리 구성

- local-path 경로는 `/data/dev/k8s`로 확정되어 있음
- 기본 설정이면 PVC들이 `pvc-<uid>_<namespace>_<pvc이름>` 형태로 한 폴더에 섞임
- StorageClass의 `pathPattern`을 `{{ .PVC.Namespace }}/{{ .PVC.Name }}`로 주면 `/data/dev/k8s/choimory-dev-prod/...`처럼 프로젝트·환경별 폴더로 나뉨. `/data` 아래를 프로젝트별로 정리해 둔 구조와 맞추기 위해 추천

## application-stg.yml 작성 계획

- 로그 레벨이 확정되어 바로 작성 가능. member-api 저장소에서 아래 순서로 진행
  1. `member-api/src/main/resources/application-stg.yml` 새로 작성 (운영과 비슷한 설정에 추천 로그 레벨, CORS는 `https://staging.choimory.dev`)
  2. `application.yml`의 group에 `stg: stg, flyway` 한 줄 추가
  3. 커밋은 하지 않음. 사용자가 직접 진행
- 앱 저장소 코드를 고치는 구현 작업이므로 작성 허락과 calendar 상태(TASK로 변경할지, PLAN 유지할지) 확인 필요

## 남은 결정 사항

- [x] 로그 레벨: 추천안으로 확정
- [x] DB와 Redis: 프로젝트별·환경별로 따로 둠 (`projects/<project>/data/`)
- [x] 이름: 내부는 `stg`, 도메인만 `staging`
- [x] API 도메인: Envoy Gateway를 API 게이트웨이로 쓰고 경로 방식 (`api.choimory.dev`, `staging-api.choimory.dev`)
- [ ] gRPC: 내부 전용으로 둘지
- [ ] local-path `pathPattern`으로 프로젝트·환경별 폴더를 나눌지
- [ ] application-stg.yml 작성 허락과 calendar 상태


---

# 추가6 남은 결정 사항 확인

> 1. grpc는 내부전용이 맞음. 일단 개발 안했으니까 이건 나중에.
> 2. local-path는 k8s 전체 설정으로 하기로 한거 아니였음?
> 3. 작업시작하면 그때 gitops도 적고 yml도 적고

## 답변

- local-path 경로는 k8s 전체 설정으로 `/data/dev/k8s` 하나로 정한 그대로임. `pathPattern`도 프로젝트마다 하는 설정이 아님
- gRPC는 내부 전용으로 확정하고 설계는 개발할 때로 미룸
- application-stg.yml은 구현 시작 때 gitops 작업과 함께 작성

## gRPC

- 내부 전용으로 확정
- 아직 개발하지 않았으므로 실제 설계(포트, Service, GRPCRoute 필요 여부)는 개발할 때 진행

## local-path 경로와 pathPattern

- 경로: 클러스터 전체 설정이며 `/data/dev/k8s` 하나. 변경 없음
- `pathPattern`: 그 경로 안에서 PVC 폴더 이름을 어떻게 지을지 정하는 옵션. StorageClass 하나에 한 번만 설정하며 클러스터 전체에 똑같이 적용됨

| | 기본 설정 | `pathPattern` 지정 |
| --- | --- | --- |
| 폴더 모양 | `/data/dev/k8s/pvc-<uid>_choimory-dev-prod_pgdata/` | `/data/dev/k8s/choimory-dev-prod/pgdata/` |
| 장점 | 설정할 것이 없음 | 폴더만 보고 어느 프로젝트, 어느 환경 데이터인지 알 수 있음 |
| 단점 | 이름이 길고 섞여 보임 | PVC를 지웠다가 같은 이름으로 다시 만들면 예전 폴더를 다시 쓸 수 있어 삭제 시 주의 필요 |

- 둘 다 클러스터 전체 설정이고 차이는 `/data/dev/k8s` 안의 폴더 이름뿐. 디스크 정리가 편해 지정하는 쪽을 추천하지만 기본값도 문제없음

### 프로젝트 디렉토리와의 관계

- k8s의 DB와 Redis 데이터는 `/data/dev/choimory-dev/` 같은 프로젝트 폴더가 아니라 `/data/dev/k8s/` 아래에 쌓임
- 프로젝트별로 DB와 Redis를 두더라도 디스크에서는 `/data/dev/k8s` 한 곳에 모임. 이것이 의도에 맞는지 확인 필요

## application-stg.yml 작성 시점

- 지금은 작성하지 않음
- 구현 시작 지시가 있으면 calendar를 TASK로 바꾸고, gitops 작업과 member-api의 yml 작업(`application-stg.yml` 작성, `application.yml` group에 `stg: stg, flyway` 추가)을 함께 진행

## 남은 결정 사항

- [x] gRPC: 내부 전용. 설계는 개발할 때
- [x] application-stg.yml: 구현 시작 때 gitops 작업과 함께 작성
- [ ] local-path `pathPattern`: 지정할지, 기본값으로 둘지
- [ ] DB와 Redis 데이터가 프로젝트 폴더가 아닌 `/data/dev/k8s` 아래에 모이는 것이 의도에 맞는지

---

# pathPattern 사용 확정

> pathpattern 사용하자

## 답변

- local-path StorageClass에 `pathPattern`을 쓰는 것으로 확정

```yaml
# StorageClass (local-path)
parameters:
  pathPattern: "{{ .PVC.Namespace }}/{{ .PVC.Name }}"
```

- PVC 폴더가 `<저장 경로>/<네임스페이스>/<pvc이름>/` 형태로 만들어짐
- 같은 이름으로 PVC를 다시 만들면 예전 폴더를 그대로 쓸 수 있으므로, 데이터를 완전히 지우려면 폴더도 직접 지워야 함
- 구현 시 `platform/local-path-provisioner/`의 StorageClass에 반영

---

# DB와 Redis 데이터 경로: 프로젝트 폴더

> db, redis 데이터는 data/dev/choimory-dev에 프로젝트별로 하는게 낫지 않아?

## 답변

- 가능하며 `/data`를 이미 프로젝트별로 정리해 둔 구조와 더 일관됨
- 처음에 `/data/dev/k8s`로 정한 것은 저장 경로가 클러스터 전체에 하나뿐이라고 봤기 때문인데, StorageClass를 프로젝트마다 하나씩 두면 프로젝트별 경로를 쓸 수 있음
- 프로젝트별 경로로 확정

## 방법: 프로젝트별 StorageClass

- local-path-provisioner 설정(ConfigMap)의 `storageClassConfigs`에 StorageClass별 경로를 따로 지정

```json
{
  "nodePathMap": [
    { "node": "DEFAULT_PATH_FOR_NON_LISTED_NODES", "paths": ["/data/dev/k8s"] }
  ],
  "storageClassConfigs": {
    "local-path-choimory-dev": {
      "nodePathMap": [
        { "node": "DEFAULT_PATH_FOR_NON_LISTED_NODES", "paths": ["/data/dev/choimory-dev/k8s"] }
      ]
    }
  }
}
```

- StorageClass `local-path-choimory-dev`를 만들고 `pathPattern: "{{ .PVC.Namespace }}/{{ .PVC.Name }}"`을 그대로 넣음
- CloudNativePG의 `storage.storageClass`와 Redis의 PVC에 이 StorageClass를 지정
- 데이터 위치
  - `/data/dev/choimory-dev/k8s/choimory-dev-stg/<pvc이름>/`
  - `/data/dev/choimory-dev/k8s/choimory-dev-prod/<pvc이름>/`
- 기본 StorageClass(`/data/dev/k8s`)는 그대로 둠. 프로젝트에 속하지 않는 공용 데이터가 생기면 여기에 쌓임
- `storageClassConfigs`는 비교적 최근 버전(v0.0.28 무렵)에 추가된 기능으로 알고 있음. 구현 시 설치 버전의 지원 여부 확인 필요
  - 지원하지 않으면 프로젝트마다 provisioner를 하나씩 띄우는 방법으로 변경

## 장단점

| | 공통 경로 (`/data/dev/k8s`) | 프로젝트별 경로 (`/data/dev/choimory-dev/k8s`) |
| --- | --- | --- |
| 기존 `/data` 구조와 맞는지 | 따로 놂 | 맞음 |
| 백업 (Kopia 등) | 프로젝트 단위로 고르기 번거로움 | 프로젝트 폴더째로 백업 가능 |
| 프로젝트 정리 | 네임스페이스 폴더를 찾아서 지워야 함 | 프로젝트 폴더만 지우면 됨 |
| 프로젝트 추가 시 | 할 일 없음 | `platform/local-path-provisioner`의 ConfigMap에 항목 하나, StorageClass 하나 추가 |
| 실수할 여지 | 적음 | PVC에 StorageClass 지정을 빠뜨리면 기본 경로에 쌓임 |

- 프로젝트가 자주 늘어나지 않는다면 추가 수고는 작으므로 프로젝트별 경로 추천

## 확정 사항

- [x] `pathPattern` 사용: `{{ .PVC.Namespace }}/{{ .PVC.Name }}`
- [x] 기본 경로: `/data/dev/k8s` (공용, 기본 StorageClass)
- [x] 프로젝트 경로: `/data/<분류>/<project>/k8s/<namespace>/<pvc>` (프로젝트마다 StorageClass `local-path-<project>`)
- [x] DB와 Redis 데이터: 프로젝트 경로 사용 (choimory-dev는 `/data/dev/choimory-dev/k8s`)
- [ ] 구현 시 local-path-provisioner 버전의 `storageClassConfigs` 지원 여부 확인
