# 목차

- [개요](#개요)
- [3단계 gitops 구성안](#3단계-gitops-구성안)
  - [답변](#답변)
  - [디렉토리 구성](#디렉토리-구성)
  - [설치 순서 (infrastructure/apps)](#설치-순서-infrastructureapps)
  - [기존 계획에서 고칠 것](#기존-계획에서-고칠-것)
  - [주요 설계](#주요-설계)
  - [이번 범위에서 제외](#이번-범위에서-제외)
  - [결정 필요 사항](#결정-필요-사항)
- [actuator 추가와 Deploy Key 등록 시점](#actuator-추가와-deploy-key-등록-시점)
  - [답변](#답변-1)
  - [Deploy Key 등록 시점](#deploy-key-등록-시점)
  - [actuator 추가안](#actuator-추가안)
  - [이렇게 잡은 이유](#이렇게-잡은-이유)
  - [결정 필요 사항](#결정-필요-사항-1)
- [추가1](#추가1)
- [data 앱 prune 설명](#data-앱-prune-설명)
  - [답변](#답변-2)
  - [prune이란](#prune이란)
  - [data 앱만 끄자는 이유](#data-앱만-끄자는-이유)
  - [끄면 생기는 비용](#끄면-생기는-비용)
  - [한계](#한계)
  - [결정 필요 사항](#결정-필요-사항-2)

---

# 개요

- 구현 순서 3단계(gitops 작성)의 구성안과 이에 필요한 member-api actuator 추가안을 정리함
- 인증 방식과 2단계 분석은 [plan5.md](./plan5.md), 구현 순서는 [plan4.md](./plan4.md) 참고

---

# 3단계 gitops 구성안

> 3단계 gitops 구성안 보여줘

## 답변

- 3단계 구성안을 정리함. 파일은 아직 만들지 않음
- local-path-provisioner v0.0.37 기준 `storageClassConfigs`는 지원되지만, plan3의 설정은 그대로 쓰면 안 되어 수정함

## 디렉토리 구성

```text
choimory-gitops/
├── bootstrap/
│   └── root.yaml                 # 처음 한 번만 apply. infrastructure/apps를 바라봄
├── infrastructure/
│   ├── apps/                     # ArgoCD Application 모음 (sync wave로 순서 지정)
│   ├── namespace.yaml
│   ├── argocd/values.yaml        # Helm 최초 설치와 ArgoCD 자체 관리에 공용
│   ├── sealed-secrets/values.yaml
│   ├── local-path-provisioner/   # kustomization, config.json, storageclasses.yaml
│   ├── metrics-server/values.yaml
│   ├── cloudnative-pg/values.yaml
│   ├── envoy-gateway/values.yaml
│   └── gateway/                  # GatewayClass, EnvoyProxy, 공용 Gateway
└── projects/choimory-dev/
    ├── argocd/                   # AppProject, ApplicationSet
    ├── namespaces/               # base + overlays/{stg,prod}: Namespace, Quota, LimitRange, NetworkPolicy
    ├── data/                     # base + overlays/{stg,prod}: CloudNativePG Cluster, Redis
    ├── member-api/               # base + overlays/{stg,prod}: Deployment, Service, HTTPRoute
    └── front/                    # base + overlays/{stg,prod}: Deployment, Service, HTTPRoute
```

## 설치 순서 (infrastructure/apps)

| wave | 대상 |
| --- | --- |
| 0 | `infrastructure` 네임스페이스, ArgoCD 자체 관리 |
| 1 | Sealed Secrets |
| 2 | local-path-provisioner, StorageClass |
| 3 | metrics-server |
| 4 | CloudNativePG 오퍼레이터 |
| 5 | Envoy Gateway 컨트롤러 |
| 6 | 공용 Gateway |
| 7 | `choimory-dev` (AppProject, ApplicationSet) |

## 기존 계획에서 고칠 것

- local-path 설정: `storageClassConfigs`를 쓰면 최상위 `nodePathMap`은 `[]`여야 함. 공용 `local-path`와 `local-path-choimory-dev`를 모두 `storageClassConfigs` 안에 넣음
- `pathPattern`: 렌더링 결과가 `{{ .PVC.Namespace }}/{{ .PVC.Name }}/`로 시작해야 해서 끝에 `/`를 붙임. 폴더 구조는 그대로
- Gateway 리스너: `*.choimory.dev`는 `choimory.dev` 자체에 매칭되지 않음. 리스너에는 hostname을 두지 않고 각 HTTPRoute에서 도메인을 지정
- sync wave: ArgoCD는 기본적으로 자식 Application의 Health를 기다리지 않음. 순서가 지켜지도록 `argocd/values.yaml`에 Application health 설정을 넣음

## 주요 설계

- 라우팅: `api.choimory.dev/member`와 `staging-api.choimory.dev/member`는 `URLRewrite`로 접두어를 떼고 `member-api:8080`에 전달. front는 서버 측 주소로 `http://member-api:8080`을 씀
- DB: 클러스터 이름은 `choimory-dev-pg`이고, member-api는 자동 생성되는 `choimory-dev-pg-app` Secret을 참조
- Quota: prod 3Gi, stg 2Gi. Pod별 값은 plan3 그대로
- NetworkPolicy: 같은 네임스페이스와 `infrastructure`에서 오는 요청만 허용. 그래서 stg와 prod 사이는 차단됨
- ArgoCD 대상: 레포 URL은 HTTPS, `targetRevision`은 `main`. 지금 작업 브랜치는 `dev`이므로 작성 후 `main` 머지 필요

## 이번 범위에서 제외

- SealedSecret 본문(6단계), cloudflared(2차), kubeadm·Cilium 설정(4단계)은 제외
- 첫 동기화 직후에는 member-api가 Secret 대기 상태, 앱 이미지는 첫 CI 전까지 pull 실패 상태가 정상

## 결정 필요 사항

- [ ] probe: member-api에 actuator가 없어 지금은 앱 수정 없이 TCP probe로 갈지 (추천). 실제 준비 상태를 정확히 보려면 나중에 actuator 추가
- [ ] data 앱의 `prune`: data 앱만 끌지 (추천). 매니페스트를 실수로 지워도 ArgoCD가 DB를 삭제하지 않음. 앱과 나머지는 켬
- [ ] 이 구성으로 `dev` 작업 트리에 작성할지. 차트 버전은 작성할 때 확인해서 고정하고 `kubectl kustomize`로 렌더링만 검증

---

# actuator 추가와 Deploy Key 등록 시점

> 1. member-api에 spring actuator 추가
> 2. front, api 리포에 ssh부터 추가해야하지 않아?

## 답변

- Deploy Key는 첫 `stg` 머지 전에만 끝나 있으면 됨
- actuator는 의존성 외에 설정과 Security도 고쳐야 하므로 수정안을 정리함. 파일은 아직 수정하지 않음

## Deploy Key 등록 시점

- 순서상 먼저 할 필요는 없음. 3단계 gitops 작성은 키를 쓰지 않고, 워크플로는 `stg`, `main`에 올라가야 돌기 때문
- 서로 의존하지 않으므로 지금 해둬도 됨

| 순서 | 작업 | 키 필요 |
| --- | --- | --- |
| 1 | gitops 작성, `main`에 push | 아니요 |
| 2 | Deploy Key 2개 등록, front Environments 변수 등록 | 이 단계에서 등록 |
| 3 | 앱 레포 `stg` 머지 → 워크플로 실행 | 예 |

## actuator 추가안

- `/actuator/health`도 지금 Security 설정으로는 401이 되므로 세 곳을 고쳐야 함

### member-api/build.gradle

```diff
     implementation 'org.springframework.boot:spring-boot-starter-web'
+    implementation 'org.springframework.boot:spring-boot-starter-actuator'
```

### application.yml

- `server`, `spring`과 같은 최상위에 추가

```yaml
management:
    # 앱 포트(8080)와 분리해 Gateway를 통해 외부로 노출되지 않게 한다.
    server:
        port: 8081
    endpoints:
        web:
            exposure:
                include: health
    endpoint:
        health:
            probes:
                enabled: true
    health:
        # Elasticsearch를 쓰지 않아 health가 DOWN으로 집계되는 것을 막는다.
        elasticsearch:
            enabled: false
```

### SecurityConfig.java

```diff
+                        .requestMatchers(EndpointRequest.to("health"))
+                        .permitAll()
                         .anyRequest()
                         .authenticated())
```

## 이렇게 잡은 이유

- 포트 분리 (8081): HTTPRoute는 8080 Service만 바라보므로 `api.choimory.dev/member/actuator/...`로는 접근되지 않음
- health만 공개: 다른 actuator 엔드포인트는 노출하지 않음
- Elasticsearch: 소스에서 쓰는 곳이 없는데 의존성만 있어, 끄지 않으면 전체 health가 DOWN이 될 수 있음. 쓰지 않는다면 starter 자체를 빼는 것이 더 깔끔하지만 범위 밖이라 건드리지 않음
- readiness 기준: 기본값대로 앱 상태만 봄. DB가 잠깐 끊겼다고 Pod를 트래픽에서 빼도 나아지는 것이 없기 때문
- gitops의 Deployment에는 포트 8081의 `/actuator/health/liveness`, `/actuator/health/readiness`를 probe로 넣고, 기동 시간을 고려해 `startupProbe`도 추가

## 결정 필요 사항

- [x] probe: TCP probe 대신 actuator 추가
- [ ] actuator 수정안으로 작성할지. 작성 후 `./gradlew test`로 검증
- [ ] data 앱만 `prune`을 끌지 (추천)
- [ ] actuator 다음에 3단계 구성안대로 `dev` 작업 트리에 작성할지

---

# 추가1

- actuator 수정안으로 작성
- data앱 prune 설명 필요
- actuator 다음 gitops dev 브랜치에 구성안대로 작성

---

# data 앱 prune 설명

> rpune 설명 해달라니까

## 답변

- 설명 시점 기준으로 actuator 추가는 끝남(테스트, `spotlessCheck` 통과, 커밋 전). gitops 파일은 아직 만들지 않음

## prune이란

- Git에서 사라진 리소스를 클러스터에서도 지울지 정하는 ArgoCD 자동 동기화 옵션
- Git에 있는 내용을 고치거나 새로 만드는 동작과는 상관없고 삭제만 다룸

| Git에서 `redis-statefulset.yaml`을 지우면 | 결과 |
| --- | --- |
| `prune: true` | ArgoCD가 클러스터의 Redis도 삭제 |
| `prune: false` | Redis는 그대로 돌고, ArgoCD에 `OutOfSync`로만 표시 (직접 지워야 함) |

## data 앱만 끄자는 이유

- 앱(member-api, front)은 지워져도 이미지로 다시 띄우면 되지만, data 앱의 CloudNativePG `Cluster`와 Redis에는 실제 데이터가 달려 있음
- 예: 리팩토링 중 `kustomization.yaml`의 `resources`에서 `cluster.yaml` 한 줄을 실수로 빼고 push한 경우

| | `prune: true` | `prune: false` |
| --- | --- | --- |
| DB Pod | 삭제되어 서비스 중단 | 계속 동작 |
| PVC | `Cluster`와 함께 삭제 | 유지 |
| 디스크 폴더 | `Retain` 덕분에 남음 | 유지 |
| 복구 | 폴더가 남아 있어도 PV와 PVC를 수동으로 다시 연결해야 함 | 줄을 되돌리면 끝 |

- `Retain`은 데이터 파일을 지켜 줄 뿐, 서비스 중단과 수동 복구까지 막아 주지는 못함
- DB 백업도 아직 없어 이 폴더가 유일한 사본임

## 끄면 생기는 비용

- 정말 삭제하려는 data 리소스는 Git에서 지운 뒤 ArgoCD UI나 `kubectl`로 한 번 더 직접 지워야 함
- data 리소스를 지울 일은 드물어 부담은 작음

## 한계

- `prune: false`는 Git에서 리소스가 빠진 경우만 막음
- ArgoCD Application 자체가 삭제될 때의 연쇄 삭제는 별개라서, 구현할 때 ApplicationSet에 `preserveResourcesOnDeletion: true`를 넣어 같이 막는 것을 추천

## 결정 필요 사항

- [ ] data 앱을 `prune: false`로 할지 (추천)