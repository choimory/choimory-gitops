# 목차

- [개요](#개요)
- [DB와 Redis 네임스페이스](#db와-redis-네임스페이스)
  - [답변](#답변)
  - [같은 네임스페이스에 두는 이유](#같은-네임스페이스에-두는-이유)
- [기존 프로젝트 폴더 사용 여부](#기존-프로젝트-폴더-사용-여부)
  - [답변](#답변-1)
  - [디스크 구조](#디스크-구조)
  - [노션 4차 경로](#노션-4차-경로)
- [공용 경로 /data/dev/k8s 필요 여부](#공용-경로-datadevk8s-필요-여부)
  - [답변](#답변-2)
  - [지금 PVC를 쓰는 곳](#지금-pvc를-쓰는-곳)
  - [추천: 기본 StorageClass 두지 않기](#추천-기본-storageclass-두지-않기)
- [모니터링과 통합 대상](#모니터링과-통합-대상)
  - [답변](#답변-3)
  - [모니터링을 프로젝트별로 붙일 때](#모니터링을-프로젝트별로-붙일-때)
  - [통합 대상과 PVC](#통합-대상과-pvc)
  - [정리](#정리)
- [공용 경로 이름: infra](#공용-경로-이름-infra)
  - [답변](#답변-4)
  - [infra/k8s로 두는 이유](#infrak8s로-두는-이유)
  - [gitops 디렉토리 이름과의 관계](#gitops-디렉토리-이름과의-관계)
- [infra 네임스페이스](#infra-네임스페이스)
  - [답변](#답변-5)
  - [구성 요소별로 infra에 넣을 때](#구성-요소별로-infra에-넣을-때)
  - [한 네임스페이스로 묶을 때의 장단점](#한-네임스페이스로-묶을-때의-장단점)
  - [추천](#추천)
- [네임스페이스 규칙](#네임스페이스-규칙)
  - [답변](#답변-6)
  - [규칙에 맞춘 데이터 경로](#규칙에-맞춘-데이터-경로)
- [디렉토리 구성](#디렉토리-구성)
  - [답변](#답변-7)
  - [gitops 저장소](#gitops-저장소)
  - [서버 디스크](#서버-디스크)
  - [남은 결정 사항](#남은-결정-사항)
- [추가1](#추가1)
- [추가1 분석](#추가1-분석)
  - [답변](#답변-8)
  - [default 네임스페이스](#default-네임스페이스)
  - [local-path 쉽게 설명](#local-path-쉽게-설명)
  - [세 가지 계획 따로 보기](#세-가지-계획-따로-보기)
- [추가1 재분석](#추가1-재분석)
  - [답변](#답변-9)
  - [원하는 구조](#원하는-구조)
  - [k8s 데이터 저장 위치](#k8s-데이터-저장-위치)
  - [이번에 확정할 것](#이번에-확정할-것)
- [추가2](#추가2)
- [추가2 분석](#추가2-분석)
  - [답변](#답변-10)
  - [저장 위치 구성](#저장-위치-구성)
  - [필요한 설정](#필요한-설정)
  - [확인 사항](#확인-사항)
- [공용 폴더와 프로젝트 폴더 확정](#공용-폴더와-프로젝트-폴더-확정)
  - [답변](#답변-11)
  - [프로젝트 추가 시 데이터 쪽 할 일](#프로젝트-추가-시-데이터-쪽-할-일)
  - [폴더 규칙](#폴더-규칙)
- [결정 현황 확인](#결정-현황-확인)
  - [답변](#답변-12)
  - [아직 확정하지 않은 것](#아직-확정하지-않은-것)
  - [구현 때 확인하거나 미룬 것](#구현-때-확인하거나-미룬-것)
- [local-path와 StorageClass 구성](#local-path와-storageclass-구성)
  - [답변](#답변-13)
  - [전체 그림](#전체-그림)
  - [provisioner 설정 (ConfigMap)](#provisioner-설정-configmap)
  - [StorageClass 두 개](#storageclass-두-개)
  - [프로젝트에서 쓰는 법](#프로젝트에서-쓰는-법)
  - [gitops 위치](#gitops-위치)
  - [project-b를 추가할 때](#project-b를-추가할-때)
  - [구현 때 확인할 것](#구현-때-확인할-것)
- [정해진 것과 정해야 할 것 요약](#정해진-것과-정해야-할-것-요약)
  - [답변](#답변-14)
  - [확정 사항 요약](#확정-사항-요약)
  - [결정 필요 사항](#결정-필요-사항)
  - [구현 시 확인 사항](#구현-시-확인-사항)
  - [보류 사항](#보류-사항)
- [추가3](#추가3)
- [추가3 분석](#추가3-분석)
  - [답변](#답변-15)
  - [reclaimPolicy](#reclaimpolicy)
  - [CNI](#cni)
  - [DB 구현](#db-구현)
  - [Redis 구성](#redis-구성)
  - [비밀값 관리](#비밀값-관리)
  - [GHCR 공개 여부](#ghcr-공개-여부)
  - [자원 설정](#자원-설정)
  - [구현 순서](#구현-순서)
  - [추가3 결정 현황](#추가3-결정-현황)
- [최종결정](#최종결정)
- [최종결정 분석](#최종결정-분석)
  - [답변](#답변-16)
  - [확정](#확정)
  - [자원: 최소한으로 다시 잡은 값](#자원-최소한으로-다시-잡은-값)
  - [남은 것](#남은-것)

---

# 개요

- plan2에 이어 네임스페이스, 저장 경로, 디렉토리 구성을 정리함
- plan2에서 확정한 사항
  - DB와 Redis는 프로젝트별·환경별로 따로 둠
  - local-path는 `pathPattern`(`{{ .PVC.Namespace }}/{{ .PVC.Name }}`)을 사용함
  - 프로젝트 데이터는 프로젝트별 StorageClass로 프로젝트 폴더에 저장함
- 참고 문서: [plan1.md](./plan1.md)(스펙 결정), [plan2.md](./plan2.md)(개발 방향)

---

# DB와 Redis 네임스페이스

> 그럼 db, redis도 ns별로 한다는거지?

## 답변

- DB와 Redis는 앱과 같은 네임스페이스에 환경별로 하나씩 둠

| 네임스페이스 | 들어가는 것 | 데이터 경로 |
| --- | --- | --- |
| `choimory-dev-stg` | member-api, front, PostgreSQL, Redis | `/data/dev/choimory-dev/k8s/choimory-dev-stg/<pvc>/` |
| `choimory-dev-prod` | member-api, front, PostgreSQL, Redis | `/data/dev/choimory-dev/k8s/choimory-dev-prod/<pvc>/` |

## 같은 네임스페이스에 두는 이유

- `data-stg` 같은 별도 네임스페이스가 아니라 앱과 같은 네임스페이스에 둠
- Secret을 그대로 쓸 수 있음
  - CloudNativePG가 만드는 `<cluster>-app` Secret(DB 접속 정보)은 클러스터와 같은 네임스페이스에 생김
  - k8s에서는 다른 네임스페이스의 Secret을 Pod가 읽을 수 없으므로, 같은 네임스페이스에 있어야 member-api가 바로 참조 가능
- 접속 주소가 짧음: `choimory-dev-pg-rw:5432`, `redis:6379`처럼 네임스페이스 없이 서비스 이름만으로 접속
- 경계가 맞음: 환경 하나가 네임스페이스 하나, 데이터 폴더 하나와 그대로 대응. 정리할 때도 네임스페이스와 폴더만 지우면 됨
- gitops에서는 `projects/choimory-dev/data/`에 base와 overlays(`stg`, `prod`)를 두고, overlay마다 네임스페이스를 `choimory-dev-stg`, `choimory-dev-prod`로 지정
- API가 늘어나도 DB 인스턴스는 늘리지 않음. 프로젝트·환경당 PostgreSQL 하나 안에서 서비스마다 database와 user를 나누고, Redis는 ACL 키 접두어로 나눔

---

# 기존 프로젝트 폴더 사용 여부

> 그럼 기존 /data/dev/choimory-dev 이건 안쓰는거지?

## 답변

- 사용함. 그 폴더가 프로젝트 폴더가 되고, 그 아래 `k8s/`에 데이터가 쌓임

## 디스크 구조

```
/data/dev/
├── k8s/                          ← 기본 StorageClass (공용, 프로젝트에 속하지 않는 데이터)
└── choimory-dev/                 ← 기존 프로젝트 폴더 (그대로 사용)
    └── k8s/                      ← StorageClass local-path-choimory-dev
        ├── choimory-dev-stg/
        │   ├── <postgres pvc>/
        │   └── <redis pvc>/
        └── choimory-dev-prod/
            ├── <postgres pvc>/
            └── <redis pvc>/
```

- `/data/dev/choimory-dev/` 바로 아래의 다른 것들(백업 폴더 등)은 그대로 둠. k8s는 `k8s/` 하위 폴더만 사용
- 이 시점의 `/data/dev/k8s` 공용 경로는 이후 논의에서 두지 않기로 함([공용 경로 /data/dev/k8s 필요 여부](#공용-경로-datadevk8s-필요-여부) 참고)

## 노션 4차 경로

- 프로젝트별 데이터 경로가 처음 노션에 정리해 두었던 `/data/dev/choimory-dev/k8s`로 다시 돌아옴
- 지난번에 미뤄둔 "노션 4차 경로를 `/data/dev/k8s`로 바꾸는 일"은 할 필요가 없어짐
- 대신 공용 경로와 StorageClass로 프로젝트 경로를 나눈다는 점을 덧붙이면 됨

---

# 공용 경로 /data/dev/k8s 필요 여부

> 그럼 /data/dev/k8s가 필요 없는거 아닌가?

## 답변

- 지금 계획대로라면 `/data/dev/k8s`를 쓰는 곳이 없음

## 지금 PVC를 쓰는 곳

| 구성 요소 | PVC 사용 |
| --- | --- |
| ArgoCD, Envoy Gateway, cloudflared, Sealed Secrets, metrics-server, CloudNativePG 오퍼레이터 | 없음 |
| PostgreSQL, Redis | 있음. 프로젝트 StorageClass(`/data/dev/choimory-dev/k8s`)를 씀 |

- 나중에 모니터링 도구 같은 공용 데이터가 생기면 그때 처음 필요해짐

## 추천: 기본 StorageClass 두지 않기

- local-path StorageClass에서 기본(default) 지정을 뺌
- StorageClass를 지정하지 않은 PVC는 `/data/dev/k8s`에 조용히 쌓이지 않고 Pending 상태로 멈춤
  - "PVC에 StorageClass 지정을 빠뜨리면 기본 경로에 쌓인다"는 실수를 바로 알아챌 수 있음
  - 데이터가 엉뚱한 곳에 쌓였다가 백업에서 빠지는 것보다 처음부터 안 뜨는 편이 나음
- 공용 데이터가 생기면 그때 별도 StorageClass를 하나 더 만들고 경로를 정함
- 확인 필요: local-path-provisioner 설정에 최상위 `nodePathMap`이 필수일 수 있음
  - 그렇다면 항목만 남겨두고 어떤 StorageClass도 그 경로를 쓰지 않게 함
  - 폴더는 PVC가 처음 만들어질 때 생기므로 쓰지 않으면 실제 폴더도 만들어지지 않음
  - 구현 시 `storageClassConfigs` 지원 여부와 함께 확인

---

# 모니터링과 통합 대상

> 모니터링 도구도 프로젝트별로 붙일건데?
>
> 통합으로 굳이 다룬다면 Jenkins, ArgoCD 정도 아닐까?

## 답변

- 통합으로 둘 것은 그 정도가 맞음
- 다만 모니터링은 전부 프로젝트별로 나눌 수는 없고, 클러스터에 하나만 있어야 하는 부분이 있음

## 모니터링을 프로젝트별로 붙일 때

| 구성 요소 | 단위 | 이유 |
| --- | --- | --- |
| Prometheus Operator | 클러스터에 하나 | CRD와 오퍼레이터는 클러스터 단위 |
| node-exporter, kube-state-metrics | 클러스터에 하나 | 노드와 클러스터 전체 상태를 수집. 여러 개 띄우면 같은 데이터를 중복 수집 |
| Prometheus | 프로젝트별 | 오퍼레이터로 여러 개 띄우고, 각자 자기 프로젝트 네임스페이스의 ServiceMonitor만 수집 |
| Grafana, Loki | 프로젝트별 | 프로젝트 Prometheus와 Loki만 데이터 소스로 연결 |

- 프로젝트별 데이터(Prometheus, Loki 저장소)는 프로젝트 StorageClass로 `/data/dev/<project>/k8s/`에 쌓임
- 비용: 프로젝트 하나당 메모리 약 1~2GB 증가(Prometheus, Grafana, Loki). 프로젝트 2~3개면 감당 가능
- 클러스터 공용 부분(오퍼레이터, exporter)은 PVC가 필요 없음

## 통합 대상과 PVC

| 통합 대상 | PVC 필요 |
| --- | --- |
| ArgoCD | 없음. 상태는 k8s 리소스에 있고, Redis는 캐시라 사라져도 다시 만들어짐 |
| Jenkins | 있음. `JENKINS_HOME`(작업 기록, 플러그인, 설정) 저장 필요 |
| 클러스터 공용 모니터링 부분 | 없음 |

- 공용 저장 경로가 필요한 것은 Jenkins뿐

## 정리

- 기본 StorageClass는 두지 않음. PVC에 StorageClass를 빠뜨리면 Pending으로 멈추게 함
- 프로젝트 데이터(DB, Redis, 모니터링)는 `local-path-<project>` 사용
- Jenkins를 도입하면 그때 공용 StorageClass를 추가
- 확인 필요: CI는 GitHub Actions로 정했으므로 Jenkins를 k8s에 올릴 계획이 있는지

---

# 공용 경로 이름: infra

> k8s라는 네이밍대신 그럼 /data/dev/infra는 어때?

## 답변

- 공용 경로에는 `infra`가 `k8s`보다 뜻이 분명함. 그 아래에 `k8s/`를 한 단계 더 두는 것을 추천

```
/data/dev/
├── infra/                        ← 공용 (Jenkins 등)
│   └── k8s/                      ← StorageClass local-path-infra
│       └── <namespace>/<pvc>/
└── choimory-dev/                 ← 프로젝트
    └── k8s/                      ← StorageClass local-path-choimory-dev
        └── <namespace>/<pvc>/
```

## infra/k8s로 두는 이유

- 구조가 대칭이 됨: 공용도 `/data/dev/<이름>/k8s/<namespace>/<pvc>` 규칙을 따름. 공용을 `infra`라는 프로젝트 하나처럼 다룰 수 있음
- k8s 밖의 데이터와 섞이지 않음: 나중에 Docker Compose로 띄운 공용 도구나 백업 파일을 `infra/` 아래에 둘 일이 생겨도 k8s 데이터와 구분됨

## gitops 디렉토리 이름과의 관계

- gitops에서는 공용 구성 요소를 `platform/`에 두기로 했음
- StorageClass 이름을 `local-path-infra`로 하면 디스크 경로와 맞지만 gitops 디렉토리(`platform`)와는 달라짐
- 맞추려면 디스크를 `/data/dev/platform/k8s`로 하거나, gitops를 `infra/`로 바꾸는 방법이 있음
- 의미는 같아 기능상 문제는 없음. 디스크는 `infra`, gitops는 `platform`으로 두고 README 대응표에 한 줄 적어두는 정도면 충분
- 이후 논의에서 gitops에 `infra/` 최상위 디렉토리를 별도로 두는 안으로 정리됨([디렉토리 구성](#디렉토리-구성) 참고)

---

# infra 네임스페이스

> 실제로 공용스택은 infra라는 네임스페이스를 만들어서 관리할거임

## 답변

- `infra` 네임스페이스로 관리하는 것도 가능. 다만 구성 요소에 따라 잘 맞는 것과 손이 더 가는 것이 있음

## 구성 요소별로 infra에 넣을 때

| 구성 요소 | 기본 네임스페이스 | `infra`에 넣을 때 |
| --- | --- | --- |
| Jenkins | 자유 | 문제없음 |
| cloudflared | 자유 | 문제없음 |
| ArgoCD | `argocd` | 가능. 다만 Application과 ApplicationSet이 모두 ArgoCD 네임스페이스에 있어야 해서 `infra`에 같이 쌓임. 문서와 CLI 예제가 대부분 `argocd`를 가정하므로 그때마다 `-n infra` 필요 |
| Sealed Secrets | `kube-system` | 가능. `kubeseal`을 쓸 때마다 `--controller-namespace infra` 필요 |
| Envoy Gateway 컨트롤러 | `envoy-gateway-system` | 가능. Envoy 프록시 Pod가 생기는 위치도 함께 바뀌므로 cloudflared가 바라보는 Service 주소도 맞춰야 함 |
| CloudNativePG 오퍼레이터 | `cnpg-system` | 가능 |
| metrics-server | `kube-system` | 가능하지만 보통 `kube-system`에 둠 |

## 한 네임스페이스로 묶을 때의 장단점

- 좋은 점
  - 공용 스택을 `kubectl -n infra` 한 번에 볼 수 있음
  - 데이터 경로가 `/data/dev/infra/k8s/infra/<pvc>`로 정리됨
- 신경 쓸 점
  - 권한과 장애 범위가 한 곳에 모임. 예: ArgoCD에 `infra` 전체 권한을 주면 Jenkins Secret까지 함께 열림
  - 차트끼리 Service나 ConfigMap 이름이 겹치면 설치 실패 가능(드묾)
  - 기본값을 바꿔야 하는 곳이 많아 설정이 조금씩 늘어남

## 추천

| 구분 | 위치 |
| --- | --- |
| 앱 성격의 공용 스택 (Jenkins, cloudflared, 나중의 공용 도구) | `infra` 네임스페이스 |
| 컨트롤러와 오퍼레이터 (ArgoCD, Envoy Gateway, Sealed Secrets, CloudNativePG, metrics-server) | 각자 기본 네임스페이스 |

- 설정을 바꿀 일이 거의 없음. 기본 네임스페이스의 도구들은 PVC를 쓰지 않으므로 디스크 경로 규칙에도 영향 없음

---

# 네임스페이스 규칙

> 기본은 default, 인프라 스택은 infra 네임스페이스, 프로젝트는 프로젝트명-프로필 네임스페이스

## 답변

| 구분 | 네임스페이스 | 대상 |
| --- | --- | --- |
| 기본 | 각 도구의 기본값 그대로 | ArgoCD(`argocd`), Envoy Gateway(`envoy-gateway-system`), Sealed Secrets와 metrics-server(`kube-system`), CloudNativePG 오퍼레이터(`cnpg-system`), local-path-provisioner(`local-path-storage`) |
| 인프라 스택 | `infra` | cloudflared, Jenkins(도입 시), 나중의 공용 도구 |
| 프로젝트 | `<프로젝트명>-<프로필>` | `choimory-dev-stg`, `choimory-dev-prod` (앱, PostgreSQL, Redis, 프로젝트 모니터링) |

- "기본은 default"를 각 도구의 기본 네임스페이스를 그대로 쓴다는 뜻으로 이해함
- k8s의 `default` 네임스페이스 자체에 넣는 것은 추천하지 않음
  - `default`는 네임스페이스를 지정하지 않은 리소스가 모두 들어가는 곳이라 실수로 만든 리소스와 섞임
  - 각 차트는 기본 네임스페이스에 맞춰져 있어 옮기려면 설정을 모두 바꿔야 함

## 규칙에 맞춘 데이터 경로

- 인프라: `/data/dev/infra/k8s/infra/<pvc>/` (StorageClass `local-path-infra`, Jenkins 도입 시 생성)
- 프로젝트: `/data/dev/<project>/k8s/<project>-<profile>/<pvc>/` (StorageClass `local-path-<project>`)
- 기본 네임스페이스의 도구들은 PVC를 쓰지 않으므로 경로 불필요

---

# 디렉토리 구성

> 그럼 디렉토리는 어떻게 구성?

## 답변

- gitops 저장소와 서버 디스크 두 가지로 정리
- gitops는 최상위 디렉토리 하나가 네임스페이스 그룹 하나에 대응하도록 구성

## gitops 저장소

```
choimory-gitops/
├── bootstrap/                      ← 최초 1회 수동 설치 (ArgoCD Helm values, root Application)
│
├── platform/                       ← 기본 네임스페이스 (각 도구의 기본값)
│   ├── argocd/                     → argocd
│   ├── envoy-gateway/              → envoy-gateway-system (컨트롤러만)
│   ├── sealed-secrets/             → kube-system
│   ├── metrics-server/             → kube-system
│   ├── cloudnative-pg/             → cnpg-system (오퍼레이터만)
│   └── local-path-provisioner/     → local-path-storage
│       └── (ConfigMap storageClassConfigs + StorageClass local-path-infra, local-path-<project>)
│
├── infra/                          ← infra 네임스페이스
│   ├── namespace.yaml
│   ├── gateway/                    (GatewayClass, EnvoyProxy, 공용 Gateway, ClientTrafficPolicy)
│   ├── cloudflared/
│   └── jenkins/                    (도입 시)
│
└── projects/
    └── choimory-dev/               ← choimory-dev-stg, choimory-dev-prod 네임스페이스
        ├── argocd/                 (AppProject, ApplicationSet)
        ├── namespaces/             (choimory-dev-stg, choimory-dev-prod)
        ├── member-api/
        │   ├── base/               (Deployment, Service, HTTPRoute)
        │   └── overlays/{stg,prod}/
        ├── front/
        │   ├── base/
        │   └── overlays/{stg,prod}/
        └── data/
            ├── base/               (CloudNativePG Cluster, Redis StatefulSet)
            └── overlays/{stg,prod}/
```

### 기존 확정안에서 바뀐 점

- `infra/` 최상위 디렉토리 추가: 원래 `platform/`에 있던 cloudflared를 여기로 옮김
- Envoy Gateway 분리
  - 컨트롤러 설치는 `platform/envoy-gateway/`, 공용 Gateway 리소스는 `infra/gateway/`
  - 프로젝트의 HTTPRoute가 `infra` 네임스페이스의 Gateway를 참조하고, Gateway의 `allowedRoutes`로 프로젝트 네임스페이스만 허용
  - Envoy 프록시 Pod는 기본 설정상 `envoy-gateway-system`에 생김. cloudflared가 바라보는 주소는 `envoy-gateway-system`의 Envoy Service
- StorageClass는 `platform/local-path-provisioner/` 한 곳에 모음. 프로젝트별 경로도 provisioner ConfigMap에 등록해야 하므로, 프로젝트 디렉토리로 나누면 두 군데를 고쳐야 함
- 프로젝트 모니터링은 붙일 때 `projects/<project>/monitoring/` 추가. 클러스터 공용 부분(오퍼레이터, exporter)은 `platform/`에 둠

## 서버 디스크

```
/data/dev/
├── infra/
│   └── k8s/                        ← local-path-infra (Jenkins 도입 시 생성)
│       └── infra/<pvc>/
└── choimory-dev/
    └── k8s/                        ← local-path-choimory-dev
        ├── choimory-dev-stg/
        │   ├── <pg-cluster>-1/     (PostgreSQL)
        │   └── data-redis-0/       (Redis)
        └── choimory-dev-prod/
            ├── <pg-cluster>-1/
            └── data-redis-0/
```

- 폴더 이름은 `pathPattern`(`{{ .PVC.Namespace }}/{{ .PVC.Name }}`)에 따라 자동으로 만들어짐
- CloudNativePG의 PVC 이름은 `<클러스터명>-1` 형태, Redis를 StatefulSet으로 띄우면 `data-redis-0` 형태
- 폴더는 PVC가 처음 만들어질 때 생기므로 미리 만들 필요 없음

## 남은 결정 사항

- [x] DB와 Redis: 앱과 같은 프로젝트·환경 네임스페이스에 둠
- [x] 프로젝트 데이터 경로: `/data/dev/<project>/k8s/<namespace>/<pvc>`
- [x] 기본 StorageClass: 두지 않음
- [x] 공용 데이터 경로: `/data/dev/infra/k8s` (StorageClass `local-path-infra`, Jenkins 도입 시 생성)
- [x] 네임스페이스 규칙: 기본 도구는 각자 기본 네임스페이스, 인프라 스택은 `infra`, 프로젝트는 `<프로젝트명>-<프로필>`
- [ ] "기본은 default"가 각 도구의 기본 네임스페이스라는 뜻이 맞는지
- [ ] gitops에 `infra/` 최상위 디렉토리 추가, Envoy Gateway를 컨트롤러와 Gateway 리소스로 분리하는 구성 확정
- [ ] Jenkins를 k8s에 올릴 계획이 있는지
- [ ] 구현 시 local-path-provisioner 버전의 `storageClassConfigs` 지원 여부 확인

---

# 추가1

- default가 k8s 통합으로 쓰려했는데 니가 별로라매
- infra는 프로젝트 구분없이 모두 쓰이는 도구들 (젠킨스 등등). 물론 젠킨스는 쓰지 않고 github actions 쓸거임. 예시임)
- local-path가 너무 복잡해 이해 못하겠어
- ns 계획, gitops 계획, 서버 디렉토리 계획도 너무 섞여서 복잡하고 local-path, storageClass까지 들어오니까 이해가 안되고 복잡해 
- 내가 하려는거는 그냥 이거임
  - 프로젝트/환경별로 네임스페이스를 구분
  - gitops에는 프로젝트별로 구분 (gitops는 프로젝트 내에서 환경별 overlays 가능하니까)
  - 여러 프로젝트에 모두 사용되는 도구는 infrastructure 네임스페이스로 사용

---

# 추가1 분석

> 추가 1 확인

## 답변

- 한 번에 너무 많은 것을 섞어 설명했으므로 계획을 줄이고 세 가지를 따로 정리함

## default 네임스페이스

- 추천하지 않은 것은 k8s의 `default`라는 이름의 네임스페이스에 실제로 넣는 것이었음. 클러스터 전체용 도구를 따로 묶는다는 생각 자체가 별로라는 뜻은 아니었음
- 클러스터 전체용 도구(ArgoCD, Envoy Gateway 등)는 설치하면 각자 자기 네임스페이스(`argocd`, `envoy-gateway-system` 등)가 자동으로 생김

## local-path 쉽게 설명

| 용어 | 뜻 |
| --- | --- |
| PVC | 앱이 내는 "디스크 10GB 주세요"라는 요청서 |
| local-path | 요청서를 받으면 서버에 폴더를 하나 만들어 주는 도구 |
| StorageClass | "어느 폴더 아래에 만들지" 정해둔 규칙 |
| pathPattern | 새로 만드는 폴더의 이름을 짓는 규칙 |

- 복잡해진 이유: `/data/dev/choimory-dev/` 아래에 넣으려고 규칙(StorageClass)을 프로젝트마다 하나씩 만들기로 했기 때문
- 간단하게 바꾸는 것을 추천
  - 규칙은 하나만 두고 기준 폴더는 `/data/dev/k8s`
  - 그 아래에 네임스페이스 이름으로 폴더가 자동으로 생김
  - 네임스페이스 이름에 프로젝트 이름이 들어가 있어 폴더만 봐도 어느 프로젝트 것인지 알 수 있음
  - 프로젝트를 추가할 때 설정을 바꿀 일이 없음
  - 대신 k8s 데이터는 `/data/dev/choimory-dev/`가 아니라 `/data/dev/k8s/` 아래에 모임

## 세 가지 계획 따로 보기

### 네임스페이스

| 무엇 | 네임스페이스 |
| --- | --- |
| 클러스터 도구 (ArgoCD, Envoy Gateway 등) | 설치할 때 생기는 그대로 (신경 쓰지 않음) |
| 공용 도구 (모든 프로젝트가 같이 쓰는 것, 지금은 cloudflared) | `infra` |
| 프로젝트 | `choimory-dev-stg`, `choimory-dev-prod` |

### gitops 디렉토리

```
choimory-gitops/
├── bootstrap/      ← 처음 한 번 ArgoCD 설치
├── platform/       ← 클러스터 도구
├── infra/          ← 공용 도구 (infra 네임스페이스)
└── projects/
    └── choimory-dev/
        ├── member-api/
        ├── front/
        └── data/   ← PostgreSQL, Redis
```

- 디렉토리 하나가 네임스페이스 표의 한 줄과 그대로 대응

### 서버 디렉토리

```
/data/dev/k8s/
├── choimory-dev-stg/     ← stg의 PostgreSQL, Redis 데이터
├── choimory-dev-prod/    ← prod의 PostgreSQL, Redis 데이터
└── infra/                ← 공용 도구 데이터 (생기면)
```

- plan3 앞부분의 프로젝트별 StorageClass와 `/data/dev/infra` 구성은 취소하고, 규칙 하나와 기준 폴더 `/data/dev/k8s` 하나로 가는 제안
- k8s 데이터를 꼭 `/data/dev/choimory-dev/` 아래에 두고 싶다면 프로젝트마다 규칙을 하나씩 두는 방식을 다시 정리

---

# 추가1 재분석

> 추가1 다시 확인

## 답변

- 원하는 구조는 세 줄로 정리되며 그대로 하면 됨. 앞에서 예외와 선택지를 너무 많이 붙여 복잡해졌음

## 원하는 구조

| 원칙 | 네임스페이스 | gitops 디렉토리 |
| --- | --- | --- |
| 프로젝트와 환경별로 구분 | `choimory-dev-stg`, `choimory-dev-prod` | `projects/choimory-dev/` (안에서 `overlays/stg`, `overlays/prod`로 환경 구분) |
| 여러 프로젝트가 같이 쓰는 도구 | `infrastructure` | `infrastructure/` |

```
choimory-gitops/
├── bootstrap/              ← 처음 한 번 ArgoCD 설치
├── infrastructure/         ← 공용 도구 → infrastructure 네임스페이스
│   ├── argocd/
│   ├── envoy-gateway/
│   ├── cloudflared/
│   ├── sealed-secrets/
│   ├── cloudnative-pg/
│   ├── local-path-provisioner/
│   └── metrics-server/
└── projects/
    └── choimory-dev/       ← choimory-dev-stg, choimory-dev-prod 네임스페이스
        ├── member-api/  (base, overlays/stg, overlays/prod)
        ├── front/       (base, overlays/stg, overlays/prod)
        └── data/        (PostgreSQL, Redis / base, overlays/stg, overlays/prod)
```

- `default` 대신 `infrastructure`가 "k8s 통합" 역할을 함
- ArgoCD, Envoy Gateway 같은 공용 도구도 모두 `infrastructure` 네임스페이스에 넣음
  - 설치할 때 네임스페이스만 `infrastructure`로 지정
  - 대가는 명령어에 네임스페이스를 직접 붙여야 하는 정도(예: `kubeseal`에 `--controller-namespace infrastructure`)
  - 집 서버 한 대에서는 이 편이 이해하기 쉬움

## k8s 데이터 저장 위치

- local-path는 "k8s 데이터는 `/data/dev/k8s/<네임스페이스>/` 아래에 자동으로 쌓인다"는 설정 하나만 기억하면 됨

```
/data/dev/k8s/
├── choimory-dev-stg/       ← stg의 PostgreSQL, Redis
├── choimory-dev-prod/      ← prod의 PostgreSQL, Redis
└── infrastructure/         ← 공용 도구 데이터 (생기면)
```

- 네임스페이스 구분이 그대로 폴더 구분이 됨
- 프로젝트별 StorageClass와 `/data/dev/infra` 구성은 취소

## 이번에 확정할 것

- [ ] 공용 네임스페이스와 디렉토리 이름을 `infrastructure`로 할지 (plan3 앞부분에서는 `infra`)
- [ ] k8s 데이터를 `/data/dev/k8s/<네임스페이스>/`에 둘지

---

# 추가2

- infra는 그냥 설명하느라 대충쓴거고 infrastructure로 ㄱㄱ
- k8s 데이터 저장위치
  - 기본: /data/dev/choimory/k8s/<네임스페이스>
  - 프로젝트별: /data/dev/프로젝트명/k8s/<네임스페이스>

---

# 추가2 분석

> 추가2 확인

## 답변

- 두 가지 모두 가능. 공용 네임스페이스와 디렉토리 이름은 `infrastructure`로 확정

## 저장 위치 구성

| 데이터 | 저장 위치 |
| --- | --- |
| 기본 (infrastructure 등 프로젝트가 아닌 것) | `/data/dev/choimory/k8s/<네임스페이스>/` |
| 프로젝트 (예: choimory-dev) | `/data/dev/choimory-dev/k8s/choimory-dev-stg/`, `/data/dev/choimory-dev/k8s/choimory-dev-prod/` |

```
/data/dev/
├── choimory/k8s/
│   └── infrastructure/          ← 공용 도구 데이터
└── choimory-dev/k8s/
    ├── choimory-dev-stg/        ← stg의 PostgreSQL, Redis
    └── choimory-dev-prod/       ← prod의 PostgreSQL, Redis
```

## 필요한 설정

- 프로젝트마다 상위 폴더가 다르므로, local-path 설정에 "이 프로젝트는 이 폴더"라는 항목을 하나씩 적어야 함
  - 기본 항목: `/data/dev/choimory/k8s`c
  - choimory-dev 항목: `/data/dev/choimory-dev/k8s`
- choimory-dev의 PostgreSQL과 Redis에는 "choimory-dev 항목을 써라"라고 한 줄씩 지정
- 그 아래 `<네임스페이스>/` 폴더는 자동으로 만들어짐
- 프로젝트를 새로 추가할 때는 설정에 항목 하나만 추가
- 프로젝트 쪽에 지정을 빠뜨리면 데이터가 기본 폴더(`/data/dev/choimory/k8s/`)에 쌓임. 사라지지는 않고 위치만 틀려짐

## 확인 사항

- [x] 공용 네임스페이스와 gitops 디렉토리 이름: `infrastructure`
- [x] 프로젝트 데이터: `/data/dev/<프로젝트명>/k8s/<네임스페이스>/`
- [ ] 기본 폴더 이름이 `choimory`가 맞는지 (`choimory-dev` 오타가 아니라 공용 폴더로 따로 두려는 것인지)


---

# 공용 폴더와 프로젝트 폴더 확정

> choimory는 공용폴더용, choimory-dev는 프로젝트 중 하나임. 이후 project-b가 생기면 /data/dev/choimory, choimory-dev, project-b 이렇게 되겠지

## 답변

- 맞음. 해당 구조로 확정

```
/data/dev/
├── choimory/k8s/              ← 공용 (기본 항목)
│   └── infrastructure/
├── choimory-dev/k8s/          ← 프로젝트 choimory-dev
│   ├── choimory-dev-stg/
│   └── choimory-dev-prod/
└── project-b/k8s/             ← 나중에 추가
    ├── project-b-stg/
    └── project-b-prod/
```

## 프로젝트 추가 시 데이터 쪽 할 일

1. local-path 설정에 `project-b` → `/data/dev/project-b/k8s` 항목 하나 추가
2. project-b의 DB와 Redis에 그 항목을 쓰도록 지정

## 폴더 규칙

- 공용은 `choimory`, 프로젝트는 각 프로젝트명
- 그 아래는 항상 `k8s/<네임스페이스>/`
- [x] 기본 폴더 이름: `choimory` (공용 폴더)

---

# 결정 현황 확인

> 다 정해진건가?

## 답변

- 큰 구조는 모두 정해졌으나, plan1 최종 스펙의 미정 항목 중 아직 확정하지 않은 것이 남아 있음
- 확정 사항과 남은 항목의 전체 목록은 [정해진 것과 정해야 할 것 요약](#정해진-것과-정해야-할-것-요약) 참고

## 아직 확정하지 않은 것

| 항목 | 지금 상태 / 추천 |
| --- | --- |
| CNI | 미정. Cilium 추천(Gateway는 Envoy를 쓰므로 Cilium의 Gateway 기능은 끔) |
| DB 구현 | CloudNativePG로 이야기를 진행했지만 명시적으로 확정하지 않음. 백업 저장소 R2도 마찬가지 |
| Redis 구성 | 공식 이미지와 StatefulSet 추천. prod는 AOF 켬, stg는 끔 |
| 비밀값 관리 | Sealed Secrets로 가정만 함 |
| GHCR 공개 여부 | 앱 레포의 public/private 여부에 따름. private이면 pull secret 필요. 아직 확인하지 못함 |
| 자원 설정 | kubelet `systemReserved` 6~8GB, stg는 가볍게 하는 안. 제안만 함 |
| 구현 순서 | plan1의 1~8단계는 "제안, 미확정" 상태 |

## 구현 때 확인하거나 미룬 것

- 구현 때 확인
  - local-path-provisioner의 `storageClassConfigs` 지원 여부
  - 앱 레포 git remote 정리 (`choimory-io-*` → `choimory-dev-*`)
  - gitops README 제목(`choimory-dev-manifests`)을 레포 이름과 맞추기
- 구현 시작 때 함께 할 일: member-api `application-stg.yml` 작성, calendar를 TASK로 변경
- 나중으로 미룸: gRPC 설계, 프로젝트별 모니터링, dev 서버
- 노션 5차 페이지에는 plan1 최종 스펙까지만 반영되어 있고, plan2와 plan3의 결정 사항은 아직 반영하지 않음

---

# local-path와 StorageClass 구성

> local-path랑 storageClass인가 그거는 어떻게 할거야

## 답변

- 결정된 경로를 local-path 설정으로 옮긴 계획. 실제 작성은 구현 시작 때 진행

## 전체 그림

```
local-path-provisioner (infrastructure 네임스페이스에 1개)
│
├── StorageClass "local-path"               ← 기본값. /data/dev/choimory/k8s/<네임스페이스>/<pvc>
└── StorageClass "local-path-choimory-dev"  ← /data/dev/choimory-dev/k8s/<네임스페이스>/<pvc>
```

- 프로그램(provisioner)은 하나만 설치
- 저장 규칙(StorageClass)을 공용 하나, 프로젝트마다 하나씩 둠
- 앱의 PVC는 어느 규칙을 쓸지만 고르면 되고, 폴더는 자동으로 만들어짐

## provisioner 설정 (ConfigMap)

- "어느 규칙이 어느 폴더를 쓰는지"를 적는 곳

```json
{
  "nodePathMap": [
    { "node": "DEFAULT_PATH_FOR_NON_LISTED_NODES", "paths": ["/data/dev/choimory/k8s"] }
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

- 맨 위 `nodePathMap`이 기본 폴더(공용)
- `storageClassConfigs` 아래가 프로젝트별 폴더. project-b가 생기면 여기에 한 덩어리 추가

## StorageClass 두 개

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-path
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"   # 지정 안 하면 여기로
provisioner: rancher.io/local-path
parameters:
  pathPattern: "{{ .PVC.Namespace }}/{{ .PVC.Name }}"
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Retain
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-path-choimory-dev
provisioner: rancher.io/local-path
parameters:
  pathPattern: "{{ .PVC.Namespace }}/{{ .PVC.Name }}"
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Retain
```

- `pathPattern`이 폴더 이름 규칙. `<네임스페이스>/<pvc이름>` 형태로 만듦
- `reclaimPolicy: Retain` 추천
  - 기본값(`Delete`)이면 PVC를 지울 때 DB 데이터 폴더도 같이 지워짐
  - `Retain`이면 실수로 PVC를 지워도 폴더가 남음
  - 대신 정말 필요 없어진 폴더는 직접 지워야 함

## 프로젝트에서 쓰는 법

- DB와 Redis에 규칙 이름을 한 줄 적음

```yaml
# projects/choimory-dev/data/base — CloudNativePG
spec:
  storage:
    storageClass: local-path-choimory-dev
    size: 10Gi

# Redis StatefulSet
volumeClaimTemplates:
  - spec:
      storageClassName: local-path-choimory-dev
```

- 결과

```
/data/dev/choimory-dev/k8s/choimory-dev-prod/<pg-cluster>-1/
/data/dev/choimory-dev/k8s/choimory-dev-prod/data-redis-0/
```

## gitops 위치

```
infrastructure/local-path-provisioner/
├── kustomization.yaml     ← 공식 설치 파일 + 네임스페이스를 infrastructure로 변경
├── config.json            ← provisioner 설정
└── storageclasses.yaml    ← StorageClass들
```

## project-b를 추가할 때

1. `config.json`에 `local-path-project-b` 한 덩어리 추가
2. `storageclasses.yaml`에 StorageClass 하나 추가
3. project-b의 DB와 Redis에 `local-path-project-b` 지정

## 구현 때 확인할 것

- `storageClassConfigs`는 비교적 최근 버전에 들어간 기능으로 알고 있음. 설치 버전의 지원 여부를 먼저 확인
- 지원하지 않으면 provisioner를 프로젝트마다 하나씩 띄우는 방식으로 변경(프로젝트별 설정 파일과 provisioner 이름을 따로 둠). 결과 폴더 구조는 동일
- [ ] `reclaimPolicy: Retain`으로 갈지

---

# 정해진 것과 정해야 할 것 요약

> 더 정해야할거랑 정해진거 한번 요약

## 답변

- 지금까지(plan1~plan3) 확정된 것과 아직 정해야 할 것을 한 곳에 정리

## 확정 사항 요약

| 영역 | 결정 |
| --- | --- |
| 서버 | Beelink EQR6 (Ryzen 7 7735HS, 32GB), 노드 1대. Docker Compose 앱과 함께 운영 |
| Kubernetes | kubeadm |
| CI / CD / 이미지 저장소 | GitHub Actions / ArgoCD (pull 방식) / GHCR |
| 매니페스트 | 앱은 Kustomize, ArgoCD 최초 설치만 Helm. 이후 ArgoCD가 전체 관리 |
| 외부 노출 | Cloudflare Tunnel (사용자 트래픽), Tailscale (관리 접속). 포트포워딩 없음 |
| Gateway | Envoy Gateway. API 게이트웨이로 쓰고 서비스는 경로로 구분 (`api.choimory.dev/member/...`) |
| 환경과 브랜치 | stg(`stg` 브랜치), prod(`main` 브랜치). dev 서버는 나중에. 로컬은 docker-compose `local` 프로필 |
| 배포 | 자동 배포. 사람이 확인하는 지점은 앱 레포의 `stg → main` PR |
| 도메인 | prod `choimory.dev` / `api.choimory.dev`, staging `staging.choimory.dev` / `staging-api.choimory.dev` |
| front 빌드 | 환경별 이미지 빌드 (`stg-<sha>`, `prod-<sha>`). Actions가 gitops overlay의 태그를 커밋 |
| 이름 | 내부(overlay, 네임스페이스, 태그, 브랜치)는 `stg`, 도메인만 `staging` |
| 네임스페이스 | 공용 도구는 `infrastructure`, 프로젝트는 `<프로젝트명>-<프로필>` (`choimory-dev-stg`, `choimory-dev-prod`) |
| gitops 디렉토리 | `bootstrap/`, `infrastructure/`, `projects/<프로젝트명>/` (`member-api`, `front`, `data`, 각각 base와 overlays) |
| gitops 브랜치 | `main` 하나. 환경은 overlay 디렉토리로 구분 |
| DB와 Redis | 프로젝트별·환경별로 하나씩, 앱과 같은 네임스페이스에 둠 |
| 데이터 경로 | 공용 `/data/dev/choimory/k8s/<네임스페이스>/`, 프로젝트 `/data/dev/<프로젝트명>/k8s/<네임스페이스>/` |
| local-path | provisioner 하나에 StorageClass를 공용 `local-path`(기본값), 프로젝트별 `local-path-<프로젝트명>`으로 둠. `pathPattern`은 `{{ .PVC.Namespace }}/{{ .PVC.Name }}` |
| member-api | `application-stg.yml` 추가 (추천 로그 레벨, CORS `https://staging.choimory.dev`), `application.yml` group에 `stg: stg, flyway` 추가. 구현 시작 때 작성 |
| gRPC | 내부 전용 |

## 결정 필요 사항

| 항목 | 추천 |
| --- | --- |
| `reclaimPolicy` | `Retain`. PVC를 지워도 데이터 폴더가 남음 |
| CNI | Cilium. Gateway와 Ingress 기능은 끄고 사용 |
| DB 구현 | CloudNativePG, 백업은 Cloudflare R2. stg는 백업 없이 |
| Redis 구성 | 공식 이미지를 StatefulSet으로. prod는 AOF 켬, stg는 끔 |
| 비밀값 관리 | Sealed Secrets |
| GHCR 공개 여부 | 앱 레포의 public/private 확인 필요. private이면 pull secret 필요 |
| 자원 설정 | kubelet `systemReserved` 6~8GB, 모든 Pod에 requests와 limits 지정 |
| 구현 순서 | plan1의 1~8단계 제안 확정 필요 |

## 구현 시 확인 사항

- local-path-provisioner의 `storageClassConfigs` 지원 여부
- 앱 레포 git remote 정리 (`choimory-io-*` → `choimory-dev-*`)
- gitops README 제목을 레포 이름(`choimory-gitops`)과 맞추기

## 보류 사항

- gRPC 설계
- 프로젝트별 모니터링
- dev 서버

---

# 추가3

- reclaimPolicy: reclaimPolicy는 도커 볼륨 관련되서 따로 저장하겠다는건가?
- CNI: CNI는 Gateway API 중에 Envoy gateway 쓰기로 했는데?
- DB구현: 백업 얘긴가? DB 백업은 추후 고려
- Redis 구성: Redis 공식 이미지를 StetefulSet으로 파드 띄운다는거지? AOF는 뭐지
- 비밀값 관리: Sealed secret 오케이
- GHCR 공개여부: 모든 프로젝트 리포는 public임
- 자원설정은 서버 자원 생각해서 적절하게 정해줘. 참고로 현재 Homepage에서는 0% CPU, 램 23.3 GiB 남음, 스토리지 404 GB 남음
- 구현순서는 작업시작전에 최종정리

---

# 추가3 분석

> 추가3 확인

## 답변

- 추가3의 8개 항목을 각각 설명하고 추천안을 정리함

## reclaimPolicy

- 따로 저장한다는 뜻이 아니라, PVC를 지웠을 때 데이터 폴더를 같이 지울지 정하는 설정

| 설정 | PVC를 지우면 | Docker로 치면 |
| --- | --- | --- |
| `Delete` (기본값) | 데이터 폴더도 삭제 | `docker compose down -v` |
| `Retain` | 데이터 폴더는 남음 | `docker compose down` |

- `Retain` 추천. 매니페스트를 잘못 고쳐 ArgoCD가 PVC를 지워도 DB 데이터가 남음

## CNI

- CNI와 Gateway는 다른 것이며 둘 다 필요

| | 하는 일 | Docker로 치면 |
| --- | --- | --- |
| CNI | Pod에 IP를 주고 Pod끼리 통신하게 함 | Docker의 bridge 네트워크 |
| Gateway (Envoy) | 바깥 요청을 받아 서비스로 전달 | 앞단의 리버스 프록시 (nginx 같은) |

- kubeadm에는 CNI가 들어 있지 않음. 설치하지 않으면 노드가 `NotReady` 상태로 남고 Pod가 뜨지 않음

| 후보 | 특징 |
| --- | --- |
| Flannel | 가장 단순. NetworkPolicy(네임스페이스 간 접근 차단) 미지원 |
| Calico | NetworkPolicy 지원. 무난함 |
| Cilium | NetworkPolicy 지원, 트래픽을 들여다보는 Hubble 등 기능 많음. 메모리 약 300~500MB |

- Cilium 추천. Gateway와 Ingress 기능은 끄고 사용
- NetworkPolicy가 있어야 "stg Pod가 prod DB에 접근하지 못하게" 같은 차단 가능

## DB 구현

- 백업 이야기가 아니라 PostgreSQL을 k8s에서 어떤 방식으로 띄울지에 대한 것

| 방식 | 내용 |
| --- | --- |
| CloudNativePG | `Cluster` 리소스 하나로 Pod, PVC, 접속 Secret, Service를 자동 생성. 나중에 백업을 붙일 때도 설정 몇 줄 |
| 공식 이미지 + StatefulSet | Redis와 같은 방식. Secret, Service 등을 직접 작성 |

- CloudNativePG 추천. member-api가 쓸 DB 접속 Secret을 자동으로 만들어 주는 것이 큼
- 백업은 추후 고려
  - 그 전까지 DB 데이터는 서버 디스크 한 곳에만 있음
  - prod에 실제 데이터가 쌓이기 시작하면 최소한 `pg_dump` 정도는 붙이는 것을 권장

## Redis 구성

- 공식 `redis` 이미지를 StatefulSet으로 Pod 1개 띄우고 PVC를 붙이는 것이 맞음
- AOF: Redis가 쓰기 명령을 디스크 파일에 계속 기록해 두는 기능
  - 없으면: 메모리에만 데이터가 있어 Pod가 재시작되면 Refresh Token이 전부 사라지고 모든 사용자가 로그아웃됨
  - 있으면: 재시작 시 파일을 다시 읽어 복구. 1초 단위 기록(`everysec`)이면 성능 영향은 거의 없음
- 추천: prod는 켜고, stg는 꺼도 됨

## 비밀값 관리

- Sealed Secrets로 확정

## GHCR 공개 여부

- 레포가 public이면 pull secret 없이 이미지를 받을 수 있음
- 다만 GHCR 패키지의 공개 여부는 레포와 따로 설정될 수 있음. 처음 push한 패키지가 private으로 만들어질 수 있으므로, 첫 빌드 후 패키지 설정에서 public인지 확인하는 것을 구현 체크리스트에 넣음

## 자원 설정

- 현재 RAM 23.3GiB가 남아 있으므로 OS와 Docker Compose 앱이 약 9GB를 쓰는 상태. Immich ML, Jellyfin 트랜스코딩 피크를 고려해 여유를 둠

### kubelet 예약

| 설정 | 값 | 의미 |
| --- | --- | --- |
| `systemReserved` | memory 10Gi, cpu 2 | OS와 Docker Compose 앱 몫 |
| `kubeReserved` | memory 1Gi, cpu 500m | kubelet과 containerd 몫 |
| `evictionHard` | `memory.available<500Mi` | 메모리가 이만큼 남으면 Pod를 정리해 서버를 보호 |
| k8s가 쓸 수 있는 양 | 약 20Gi | |

### Pod별 값

| 대상 | prod (요청 / 상한) | stg (요청 / 상한) |
| --- | --- | --- |
| member-api | 512Mi / 1Gi | 384Mi / 768Mi |
| front | 128Mi / 256Mi | 128Mi / 256Mi |
| PostgreSQL | 256Mi / 1Gi | 128Mi / 512Mi |
| Redis | 64Mi / 256Mi (`maxmemory 200mb`) | 32Mi / 128Mi |
| choimory-dev 합계 (상한 기준) | 약 2.5Gi | 약 1.6Gi |

- CPU: 요청값만 주고 상한은 두지 않음. CPU가 놀고 있을 때도 상한에 걸려 느려지는 것을 막음. 요청은 앱 100~200m, DB 100m 정도
- JVM: member-api는 `-XX:MaxRAMPercentage=75`로 메모리 상한에 맞춰 힙 크기 설정
- LimitRange: 프로젝트 네임스페이스마다 기본값을 두어, 값을 적지 않은 Pod에도 기본 상한이 걸리게 함

### 전체 예상

| 영역 | 메모리 |
| --- | --- |
| k8s 기반 (control plane, Cilium, ArgoCD, Envoy 등) | 약 3~4Gi |
| choimory-dev | 약 4Gi |
| 합계 (20Gi 중) | 약 7~8Gi |

- 프로젝트를 2~3개 더 추가해도 들어감

### 디스크

- PVC 크기: prod PostgreSQL 10Gi, stg 5Gi, Redis 각 1Gi
- local-path는 PVC 크기를 실제로 제한하지 않음. 크기는 표시용이고 실제 사용량은 남은 디스크(404GB) 안에서 늘어남

## 구현 순서

- 작업 시작 전에 최종 정리

## 추가3 결정 현황

| 항목 | 상태 |
| --- | --- |
| Sealed Secrets | 확정 |
| GHCR | public, pull secret 없음 (첫 push 후 패키지 공개 여부 확인) |
| DB 백업 | 추후 |
| 구현 순서 | 작업 시작 전에 정리 |
| reclaimPolicy | 확인 필요 (추천: `Retain`) |
| CNI | 확인 필요 (추천: Cilium) |
| DB 구현 | 확인 필요 (추천: CloudNativePG) |
| Redis AOF | 확인 필요 (추천: prod는 켜고 stg는 끔) |
| 자원 설정 | 확인 필요 (위 값) |

---

# 최종결정

- reclaimPolicy: Retain
- CNI: Cilium
- DB: CloudNativePG
- Redis: stg aof off, prod aof on
- 자원: 최소한으로해줘 자원 많이남았다고 다 먹지 말고 

---

# 최종결정 분석

> 최종결정 확인

## 답변

- reclaimPolicy, CNI, DB, Redis는 그대로 확정하고, 자원은 "최소한으로"에 맞춰 값을 다시 낮춤

## 확정

| 항목 | 결정 |
| --- | --- |
| reclaimPolicy | `Retain` |
| CNI | Cilium (Gateway와 Ingress 기능 끔) |
| DB | CloudNativePG (백업은 추후) |
| Redis | 공식 이미지를 StatefulSet으로. prod는 AOF 켬, stg는 끔 |

## 자원: 최소한으로 다시 잡은 값

- 원칙: 실제로 쓸 만큼만 요청하고, 상한으로 최대치를 묶음. 남은 RAM이 많아도 k8s가 상한 이상은 가져가지 못함

### Pod별 값 (요청 / 상한)

| 대상 | prod | stg | 이전 제안 대비 |
| --- | --- | --- | --- |
| member-api | 384Mi / 768Mi | 256Mi / 512Mi | 약 ⅔ |
| front | 128Mi / 256Mi | 96Mi / 192Mi | 비슷 |
| PostgreSQL | 128Mi / 512Mi (`shared_buffers` 128MB) | 64Mi / 256Mi (`shared_buffers` 32MB) | 절반 |
| Redis | 32Mi / 128Mi (`maxmemory 100mb`) | 16Mi / 64Mi (`maxmemory 32mb`) | 절반 |
| 합계 (상한 기준) | 약 1.6Gi | 약 1.0Gi | 4.1Gi → 2.6Gi |

- CPU는 요청값만 작게(앱 100m, DB와 Redis 50m) 주고 상한은 두지 않음. 지금 CPU는 거의 0%라 순간적으로 몰아서 써도 문제없음
- member-api는 `-XX:MaxRAMPercentage=75`로 prod 힙 약 576MB, stg 약 384MB. 부족하면 그때 올림
- 레플리카는 전부 1개

### 네임스페이스 상한 (ResourceQuota)

| 네임스페이스 | 메모리 상한 합계 |
| --- | --- |
| `choimory-dev-prod` | 2Gi |
| `choimory-dev-stg` | 1.5Gi |

- 프로젝트 하나가 넘을 수 없는 울타리. Pod 값을 실수로 크게 적어도 이 이상은 못 가져감
- LimitRange 기본값: 값을 적지 않은 Pod에 요청 64Mi, 상한 256Mi

### 공용 도구를 가볍게

| 도구 | 줄이는 방법 |
| --- | --- |
| ArgoCD | Dex(SSO)와 notifications를 끄고 모든 구성 요소를 레플리카 1개로 |
| Cilium | Hubble을 끄고 operator를 레플리카 1개로 |
| Envoy Gateway | 프록시 1개만, 요청 64Mi |
| CloudNativePG 오퍼레이터, Sealed Secrets, metrics-server, local-path | 기본값 (각각 수십 MB 수준) |

### kubelet 예약

| 설정 | 값 |
| --- | --- |
| `systemReserved` | memory 10Gi, cpu 2 |
| `kubeReserved` | memory 1Gi, cpu 500m |
| `evictionHard` | `memory.available<500Mi` |

- 예약값은 Docker Compose 앱을 보호하는 장치이므로 그대로 둠

### 예상 사용량

| 영역 | 평소 | 최대 (상한 기준) |
| --- | --- | --- |
| k8s 기반 (control plane, Cilium, ArgoCD, Envoy 등) | 약 2~2.5Gi | 약 3Gi |
| choimory-dev (stg + prod) | 약 1.5Gi | 약 2.6Gi |
| 합계 | 약 3.5~4Gi | 약 5.6Gi |

- 남은 23GiB 중 평소 4GiB 정도, 최악의 경우에도 6GiB 안쪽만 사용

## 남은 것

- [x] reclaimPolicy: `Retain`
- [x] CNI: Cilium
- [x] DB: CloudNativePG
- [x] Redis: prod AOF 켬, stg AOF 끔
- [ ] 자원 설정: 위 값으로 확정할지
- [ ] 구현 순서: 작업 시작 전 최종 정리
