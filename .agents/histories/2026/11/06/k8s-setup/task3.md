# 목차

- [개요](#개요)
- [member-api actuator 추가](#member-api-actuator-추가)
  - [답변](#답변)
  - [변경 파일](#변경-파일)
  - [검증 결과](#검증-결과)
- [3단계 gitops 작성](#3단계-gitops-작성)
  - [답변](#답변-1)
  - [작성한 구조](#작성한-구조)
  - [고정한 버전](#고정한-버전)
  - [구성안과 달라진 점](#구성안과-달라진-점)
  - [주요 설정](#주요-설정)
  - [작성 중 수정한 것](#작성-중-수정한-것)
  - [검증 결과](#검증-결과-1)
  - [직접 해야 할 일](#직접-해야-할-일)

---

# 개요

- 구현 순서 3단계(gitops 작성)와 그에 필요한 member-api actuator 추가를 기록함
- 구성안과 결정 과정은 [plan6.md](./plan6.md), 2단계 작업은 [task2.md](./task2.md) 참고
- gitops 파일은 `dev` 작업 트리에 작성했으며 커밋하지 않음

---

# member-api actuator 추가

> plan6 추가1 확인

## 답변

- plan6 추가1에 따라 actuator를 member-api에 추가함
- 커밋은 하지 않음

## 변경 파일

| 파일 | 변경 |
| --- | --- |
| `member-api/build.gradle` | `spring-boot-starter-actuator` 추가 |
| `member-api/src/main/resources/application.yml` | 관리 포트 8081, `health`만 노출, probe 활성화, Elasticsearch health 끔 |
| `member-api/src/main/java/dev/choimory/member/api/config/SecurityConfig.java` | `EndpointRequest.to("health")` 인증 없이 허용 |

## 검증 결과

| 항목 | 결과 |
| --- | --- |
| `./gradlew test` | 통과 |
| `spotlessCheck` | 통과 |
| 8081 포트 실제 응답 | 미검증. 앱을 띄워야 확인 가능 |

---

# 3단계 gitops 작성

> prune false로 하고 작업 계속 진행해줘

## 답변

- data 앱을 `prune: false`로 확정하고 3단계 gitops를 작성함 (파일 45개)
- SealedSecret 본문(6단계), cloudflared(2차), kubeadm·Cilium 설정(4단계)은 범위에서 제외

## 작성한 구조

```text
bootstrap/root.yaml
infrastructure/
├── apps/                     # namespace, argocd, sealed-secrets, local-path-provisioner, metrics-server,
│                             # cloudnative-pg, envoy-gateway, gateway, choimory-dev
├── argocd/values.yaml
├── sealed-secrets/values.yaml
├── local-path-provisioner/   # kustomization.yaml, storageclasses.yaml
├── metrics-server/values.yaml
├── cloudnative-pg/values.yaml
├── envoy-gateway/values.yaml
└── gateway/                  # kustomization.yaml, gateway.yaml
projects/choimory-dev/
├── argocd/                   # appproject, applicationset-apps, applicationset-protected
├── namespaces/               # base + overlays/{stg,prod}
├── data/                     # base + overlays/{stg,prod}
├── member-api/               # base + overlays/{stg,prod}
└── front/                    # base + overlays/{stg,prod}
```

## 고정한 버전

| 도구 | 버전 |
| --- | --- |
| ArgoCD 차트 | 10.17.0 (앱 v3.6.1) |
| Sealed Secrets 차트 | 2.22.0 (저장소 `https://bitnami.github.io/sealed-secrets`) |
| local-path-provisioner | v0.0.37 |
| metrics-server 차트 | 3.14.0 |
| CloudNativePG 차트 | 0.29.1 |
| Envoy Gateway 차트 | v1.10.0 |
| Redis 이미지 | 8.10.3-alpine |

## 구성안과 달라진 점

- ApplicationSet 2개로 분리: `namespaces`와 `data`는 보호용(`prune: false`, `preserveResourcesOnDeletion: true`). Namespace가 지워져도 DB가 함께 사라지기 때문. 앱(member-api, front)은 `prune: true`
- `choimory-dev` Application도 `prune: false`: ApplicationSet이 prune으로 지워지는 경로도 막음
- `namespace.yaml` 위치: ArgoCD가 읽도록 `infrastructure/`에서 `infrastructure/apps/`로 옮김
- local-path 설정 방식: `config.json`을 별도 파일로 두지 않고 `kustomization.yaml`의 패치로 넣음
- Envoy Service 이름 고정 제외: cloudflared가 쓸 일이라 2차 작업으로 미룸
- metrics-server: kubeadm 기본 kubelet 인증서가 자체 서명이라 `--kubelet-insecure-tls` 사용

## 주요 설정

- ArgoCD는 `infrastructure` 네임스페이스, 대상 브랜치는 `main`. sync wave가 지켜지도록 Application health 설정 추가
- local-path: 최상위 `nodePathMap`은 `[]`, `local-path`(기본값, `/data/dev/choimory/k8s`)와 `local-path-choimory-dev`(`/data/dev/choimory-dev/k8s`)를 `storageClassConfigs`에 지정. `pathPattern`은 끝에 `/` 포함, `reclaimPolicy: Retain`
- Gateway: `shared-gateway`, 리스너에 hostname 없음, `gateway-access: "true"` 라벨이 있는 네임스페이스의 HTTPRoute만 허용, Envoy Service는 ClusterIP
- Quota: prod 3Gi, stg 2Gi. LimitRange 기본 요청 64Mi, 상한 256Mi
- NetworkPolicy: 같은 네임스페이스와 `infrastructure`에서 오는 요청만 허용
- PostgreSQL: 클러스터 `choimory-dev-pg`, database와 owner는 `member`, `enablePDB: false`
- Redis: stg는 AOF와 스냅샷 끔, prod는 AOF `everysec`
- member-api: 8081 포트의 `/actuator/health/liveness`, `/actuator/health/readiness`로 probe. Secret은 `choimory-dev-pg-app`(자동 생성), `redis-secret`, `member-api-secret`(6단계)
- front: TCP probe, 서버 측 API 주소는 `http://member-api:8080`
- 이미지 태그 초깃값은 `stg-bootstrap`, `prod-bootstrap`. 첫 CI 전까지 pull 실패가 정상

## 작성 중 수정한 것

- 공식 배포 파일의 `local-path` StorageClass와 새로 만든 같은 이름의 리소스가 충돌해 렌더링 실패. 공식 리소스에 패치로 설정을 덧입히는 방식으로 변경

## 검증 결과

| 항목 | 결과 |
| --- | --- |
| overlay 8개, 공용 kustomize 2개 `kubectl kustomize` 렌더링 | 성공 |
| local-path 렌더링 결과 (네임스페이스, StorageClass, config) | 확인 |
| Application, ApplicationSet, Helm values 파일 | YAML 문법만 확인 |
| Helm values 렌더링 | 미검증. 로컬에 `helm` 없음 |
| 실제 클러스터 동기화 | 미검증. 4~5단계에서 확인 |

## 직접 해야 할 일

- member-api actuator 변경을 `dev`에 커밋
- gitops의 `bootstrap/`, `infrastructure/`, `projects/`를 커밋해 `main`에 머지하고 push. ArgoCD와 Actions는 `main`만 봄
- Deploy Key 2개와 front Environments 변수를 등록한 뒤 앱 레포를 `stg`로 머지
