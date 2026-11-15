# 목차

- [개요](#개요)
- [작업 시작과 문서 수정](#작업-시작과-문서-수정)
  - [답변](#답변)
  - [문서 수정 내용](#문서-수정-내용)
  - [PostgreSQL 덤프 명령어 (서버)](#postgresql-덤프-명령어-서버)
- [덤프 대상 확인](#덤프-대상-확인)
  - [답변](#답변-1)
  - [덤프하려던 이유](#덤프하려던-이유)
  - [지금은 DB가 비어 있음](#지금은-db가-비어-있음)
  - [추천](#추천)
- [gitops 구조 변경](#gitops-구조-변경)
  - [답변](#답변-2)
  - [디렉토리](#디렉토리)
  - [변경 파일](#변경-파일)
  - [검증 결과](#검증-결과)
  - [커밋과 main 반영 (직접)](#커밋과-main-반영-직접)
  - [반영 후 확인 명령어 (서버)](#반영-후-확인-명령어-서버)
  - [정상 기준](#정상-기준)
- [main 반영 확인](#main-반영-확인)
  - [답변](#답변-3)
  - [커밋 확인 결과](#커밋-확인-결과)
  - [확인 명령어 (서버)](#확인-명령어-서버)
  - [정상 기준](#정상-기준-1)
  - [앱 목록이 바뀌지 않을 때 (서버)](#앱-목록이-바뀌지-않을-때-서버)
- [앱 레포 deploy.yml 수정](#앱-레포-deployyml-수정)
  - [답변](#답변-4)
  - [변경 내용 (두 레포 동일)](#변경-내용-두-레포-동일)
  - [계획과 달라진 점: concurrency](#계획과-달라진-점-concurrency)
  - [검증](#검증)
  - [직접 할 일: Deploy Key 등록](#직접-할-일-deploy-key-등록)
  - [직접 할 일: front Environments 변수 등록](#직접-할-일-front-environments-변수-등록)
- [Deploy Key와 Environments 변수 등록 완료](#deploy-key와-environments-변수-등록-완료)
  - [답변](#답변-5)
  - [등록 결과](#등록-결과)
  - [이전 안내와 달라진 점](#이전-안내와-달라진-점)
  - [다음 진행](#다음-진행)
- [API 도메인 변경](#api-도메인-변경)
  - [답변](#답변-6)
  - [변경 내용](#변경-내용)
  - [검증](#검증-1)
  - [커밋과 반영 (직접)](#커밋과-반영-직접)
  - [gitops 반영 후 확인 명령어 (서버)](#gitops-반영-후-확인-명령어-서버)

---

# 개요

- [plan9.md](./plan9.md) 추가1, 추가2에서 정한 내용에 따라 7단계 전 작업을 시작함
- 이 문서는 문서 정정(task4, plan3), PostgreSQL 덤프 여부 결정, gitops 디렉토리 구성 변경을 기록함
- 이전 작업은 [task6.md](./task6.md)(6단계 완료) 참고

---

# 작업 시작과 문서 수정

> 시작

## 답변

- plan9 추가2 분석의 순서대로 문서 2곳(task4, plan3)을 먼저 수정함
- 그다음 PostgreSQL 덤프 명령어를 안내함

## 문서 수정 내용

| 문서 | 변경 |
|---|---|
| task4 | clone 명령어를 실제로 실행한 경로로 변경: `cd /data/dev/choimory/repo` 후 `git clone`, `cp` 경로도 맞춤 |
| plan3 | 서버 RAM 32GB를 26Gi로 변경 |
| | 사용량 계산을 "26Gi 중 23.3GiB 남음, 약 3Gi 사용"으로 변경 |
| | k8s가 쓸 수 있는 양 20Gi를 약 14.5Gi로 변경 (plan7 계산 기준) |
| | 합계 기준을 "14.5Gi 중"으로 변경 |
| | 프로젝트 추가 여유를 "2~3개"에서 "1~2개"로 변경 |
| | 정정했다는 메모와 task4, plan7 링크 추가 |

- 실제 RAM 값은 task4의 `free -h` 결과(26Gi), 정정 기준은 plan7의 "RAM 확인 결과에 따른 처리"를 따름

## PostgreSQL 덤프 명령어 (서버)

- DB 구조를 바꾸지는 않지만, 데이터 앱의 관리 주체가 바뀌므로 만약을 위해 덤프를 받아 두려 함
- DB 이름은 `member`

```bash
mkdir -p ~/backup && chmod 700 ~/backup

for env in stg prod; do
  kubectl -n choimory-dev-$env exec choimory-dev-postgres-1 -c postgres -- \
    pg_dump -U postgres -d member -Fc > ~/backup/choimory-dev-$env-member-$(date +%Y%m%d).dump
done

chmod 600 ~/backup/*.dump
ls -l ~/backup
head -c 5 ~/backup/choimory-dev-stg-member-*.dump; echo
head -c 5 ~/backup/choimory-dev-prod-member-*.dump; echo
```

- 파일 크기가 0보다 크고 `head` 결과가 `PGDMP`이면 정상
- Redis는 로그인 토큰뿐이고 PVC도 그대로 남으므로 백업하지 않음
- 이 명령어는 다음 대화에서 실행하지 않기로 함

---

# 덤프 대상 확인

> 덤프는 기존 db 덤프뜨는건가?

## 답변

- k8s 클러스터에서 돌고 있는 PostgreSQL을 덤프하는 것
- stg와 prod 네임스페이스의 `choimory-dev-postgres-1`에 있는 `member` DB가 대상이며, Docker Compose 쪽 DB나 다른 DB는 건드리지 않음

## 덤프하려던 이유

- 이번 작업은 `data` 앱을 지우고 `postgres`, `redis` 앱으로 나누는 것
- `preserveResourcesOnDeletion: true`라 DB는 지워지지 않아야 하지만, DB를 관리하는 앱이 바뀌는 작업이라 만약을 대비하려 함

## 지금은 DB가 비어 있음

- member-api는 지금까지 `ImagePullBackOff`라 한 번도 실행되지 않음
- 그래서 `member` DB에는 CloudNativePG가 처음 만든 빈 DB만 있고, 테이블이나 데이터는 없을 것으로 봄
- 최악의 경우 DB가 지워져도 CloudNativePG가 빈 DB를 다시 만들면 되는 상황

## 추천

- 덤프는 건너뛰는 것을 추천
- PostgreSQL 백업은 7단계에서 실제 데이터가 쌓이기 시작할 때 챙김 (plan9의 "범위를 정할 것" 항목)
- 구조 변경 후에는 Pod 재시작 횟수와 PVC가 그대로인지 확인해 DB 유지 여부를 검증함

---

# gitops 구조 변경

> 필요없어 아무 데이터도 기록 안했어

## 답변

- 덤프는 건너뛰고 gitops 구조 변경을 작성함
- 커밋은 하지 않음

## 디렉토리

```
projects/choimory-dev/
├── argocd/
├── namespaces/
├── postgres/                 # data에서 분리 (Cluster)
├── redis/                    # data에서 분리 (Service, StatefulSet, SealedSecret)
├── choimory-dev-front/       # front에서 이름 변경
└── choimory-dev-member-api/  # member-api에서 이름 변경
```

- 이동은 `git mv`로 하여 파일 이력이 유지됨
- `data` 디렉토리는 삭제됨

## 변경 파일

| 파일 | 변경 |
|---|---|
| `postgres/base`, `redis/base`의 `kustomization.yaml` | 새로 작성 |
| `postgres/overlays/{stg,prod}` | 기존 data overlay에서 PostgreSQL patch만 남김 |
| `redis/overlays/{stg,prod}` | Redis patch와 SealedSecret을 옮김 |
| `applicationset-apps.yaml` | 요소에 `dir` 추가, 경로를 `{{ .dir }}`로 변경 |
| `applicationset-protected.yaml` | `data`를 `postgres`, `redis` 두 요소로 나누고 `dir` 추가 |
| `choimory-dev-member-api/base/deployment.yaml` | 주석 경로 수정 (`data/base/redis.yaml` → `redis/base/redis.yaml`) |
| `infrastructure/apps/cloudnative-pg.yaml` | 주석 수정 (data 디렉토리 → postgres 디렉토리) |

- Application 이름
  - `*-front`, `*-member-api`, `*-namespaces`: 유지
  - `*-postgres`, `*-redis`: 신규
  - `*-data`: 삭제
- README에는 옛 디렉토리 이름이 언급되지 않아 수정할 것이 없음

## 검증 결과

- `kubectl kustomize`로 overlay 10개를 모두 빌드했으며 정상
- 변경 전(HEAD)과 변경 후의 빌드 결과를 비교함
  - front, member-api: stg, prod 모두 완전히 같음
  - 이전 `data` 결과와 새 `postgres` + `redis`를 합친 결과: stg, prod 모두 같음
- 클러스터에 적용되는 리소스 내용은 바뀌지 않고, 어느 Application이 관리하느냐만 바뀜

## 커밋과 main 반영 (직접)

- 디렉토리 이동과 ApplicationSet 변경은 반드시 한 커밋에 넣음
  - 나눠서 반영하면 옛 앱과 새 앱이 같은 리소스를 동시에 관리하게 됨
- 문서 변경(plan3, task4)과 추적되지 않은 plan9, task6을 같이 커밋할지는 직접 정함
- 앱 레포 `deploy.yml`은 아직 옛 경로(`projects/choimory-dev/front/...`)를 가리킴
  - 앱 레포를 머지하기 전까지 Actions가 돌지 않으므로 지금은 문제없음
  - gitops 반영 확인이 끝나면 바로 수정함

## 반영 후 확인 명령어 (서버)

- push 후 1~3분 뒤 실행

```bash
kubectl -n infrastructure get applications | grep choimory-dev
kubectl -n choimory-dev-stg get pods,pvc
kubectl -n choimory-dev-prod get pods,pvc
kubectl -n choimory-dev-stg get cluster.postgresql.cnpg.io,statefulset,sealedsecret
kubectl -n choimory-dev-prod get cluster.postgresql.cnpg.io,statefulset,sealedsecret
```

## 정상 기준

- `*-data` 앱이 목록에서 사라짐
- `*-postgres`, `*-redis` 앱이 `Synced`, `Healthy`
- postgres, redis Pod의 AGE가 그대로(4d21h 이상)이고 RESTARTS가 0. 재생성되지 않았다는 뜻
- PVC가 그대로 `Bound`

---

# main 반영 확인

> gitops 메인브랜치 push 했음

## 답변

- `main` 반영을 확인함
- 매니페스트 변경과 문서 변경이 두 커밋으로 나뉘었지만, 지켜야 했던 조건은 충족함

## 커밋 확인 결과

| 커밋 | 내용 |
|---|---|
| `b408be0` | 디렉토리 이동, postgres·redis 분리, ApplicationSet 2개, 주석 수정 |
| `86dd5a4` | 문서 6개 (plan1, plan3, plan9, task4, task6, task7) |
| `5171616` | release/1.0.3을 `main`에 머지 |

- 디렉토리 이동과 ApplicationSet 변경이 같은 커밋(`b408be0`)에 있어, 옛 앱과 새 앱이 같은 리소스를 동시에 관리하는 상황은 생기지 않음
- `origin/main`의 `projects/choimory-dev` 아래 디렉토리: `argocd`, `choimory-dev-front`, `choimory-dev-member-api`, `namespaces`, `postgres`, `redis`
- `dev`와 `main`의 매니페스트 내용이 같음

## 확인 명령어 (서버)

- push 후 1~3분 뒤 실행

```bash
kubectl -n infrastructure get applications | grep choimory-dev
kubectl -n choimory-dev-stg get pods,pvc
kubectl -n choimory-dev-prod get pods,pvc
kubectl -n choimory-dev-stg get cluster.postgresql.cnpg.io,statefulset,sealedsecret
kubectl -n choimory-dev-prod get cluster.postgresql.cnpg.io,statefulset,sealedsecret
```

## 정상 기준

- `*-data` 앱이 목록에서 사라짐
- `*-postgres`, `*-redis` 앱이 `Synced`, `Healthy`
- postgres, redis Pod의 AGE가 그대로(4d21h 이상)이고 RESTARTS가 0
- PVC가 그대로 `Bound`

## 앱 목록이 바뀌지 않을 때 (서버)

- ApplicationSet을 갖고 있는 `choimory-dev` 앱이 새 커밋을 아직 못 가져온 것일 수 있음
- 아래 명령어로 바로 다시 읽게 함

```bash
kubectl -n infrastructure annotate application choimory-dev argocd.argoproj.io/refresh=hard --overwrite
```

---

# 앱 레포 deploy.yml 수정

> 시작

## 답변

- gitops 구조 변경이 클러스터에 정상 반영된 것을 확인한 뒤 진행함. 확인 결과는 [plan9.md 구조 변경 반영 확인과 deploy.yml 수정안](./plan9.md#구조-변경-반영-확인과-deployyml-수정안) 참고
  - `*-data` 삭제, `*-postgres`, `*-redis` `Healthy`, postgres와 redis Pod 재생성 없음, PVC 유지
- 앱 레포 2개(`choimory-dev-front`, `choimory-dev-member-api`)의 `.github/workflows/deploy.yml`을 수정함
- 커밋은 하지 않음

## 변경 내용 (두 레포 동일)

| 항목 | 변경 |
|---|---|
| 트리거 | `push`(`dev`, `stg`, `main`)와 `workflow_dispatch` |
| deploy job | `if: github.event_name == 'workflow_dispatch'`. push 때는 테스트만 돎 |
| 브랜치 확인 | deploy job 첫 단계 `Check Branch`. `stg`, `main`이 아니면 "stg 또는 main 브랜치에서만 배포할 수 있습니다"를 출력하고 실패 |
| gitops 경로 | `choimory-dev-front`, `choimory-dev-member-api`로 변경 (주석 1줄, 명령 2줄) |
| 상단 주석 | 수동 배포 방식으로 수정 |

## 계획과 달라진 점: concurrency

- 계획에서는 그대로 두려 했으나, 작성 중 문제를 발견함
  - stg 수동 배포가 도는 중에 stg에 push하면 push 테스트가 배포 실행을 취소함
  - 이미지만 올라가고 gitops 태그 커밋은 안 된 상태로 끊길 수 있음
- 그룹을 `deploy-<브랜치>-<실행 방식>`(`deploy-${{ github.ref_name }}-${{ github.event_name }}`)으로 나눔
  - push 테스트끼리, 수동 배포끼리만 서로 취소함

## 검증

- Ruby YAML 파서로 두 파일을 읽음. 트리거, `if`, 첫 단계가 의도대로 들어감
  - YAML 1.1 규칙 때문에 `on` 키가 `true`로 읽히는 점을 감안해 확인함
- actionlint(`rhysd/actionlint:1.7.7`, Docker)로 두 파일 모두 오류 없음
- 실제 Actions 실행은 머지 후 확인

## 직접 할 일: Deploy Key 등록

- plan5에서 정한 대로 앱 레포별로 키를 1개씩 만듦
- Mac에서 키 생성

```bash
ssh-keygen -t ed25519 -C "choimory-dev-member-api-actions" -f ./choimory-dev-member-api-deploy -N ""
ssh-keygen -t ed25519 -C "choimory-dev-front-actions" -f ./choimory-dev-front-deploy -N ""
```

| 파일 | 등록 위치 |
|---|---|
| `choimory-dev-member-api-deploy.pub` | `choimory-gitops` → Settings → Deploy keys → Add. 제목 `choimory-dev-member-api-actions`, Allow write access 체크 |
| `choimory-dev-member-api-deploy` | `choimory-dev-member-api` → Settings → Secrets and variables → Actions → New repository secret. 이름 `GITOPS_DEPLOY_KEY` |
| `choimory-dev-front-deploy.pub` | `choimory-gitops` → Deploy keys. 제목 `choimory-dev-front-actions`, Allow write access 체크 |
| `choimory-dev-front-deploy` | `choimory-dev-front` → Actions Secret. 이름 `GITOPS_DEPLOY_KEY` |

- 등록 후 Mac의 키 파일 4개 삭제

```bash
rm ./choimory-dev-member-api-deploy ./choimory-dev-member-api-deploy.pub ./choimory-dev-front-deploy ./choimory-dev-front-deploy.pub
```

## 직접 할 일: front Environments 변수 등록

- `choimory-dev-front` → Settings → Environments에서 `stg`, `prod` 환경을 만들고 각각 Variables에 등록

| Environment | 변수 이름 | 값 |
|---|---|---|
| `stg` | `NEXT_PUBLIC_MEMBER_API_BASE_URL` | `https://staging-api.choimory.dev/member` |
| `prod` | `NEXT_PUBLIC_MEMBER_API_BASE_URL` | `https://api.choimory.dev/member` |

- Secrets가 아니라 Variables에 넣음. 워크플로가 `vars.`로 읽음
- 끝나면 7번(앱 레포 커밋과 `stg`, `main` 머지) 순서를 안내함

---

# Deploy Key와 Environments 변수 등록 완료

> secret, var 추가했어
>
> var는 stg: https://staging-member-api.choimory.dev
> prod: https://member-api.choimory.dev

## 답변

- 6번(Deploy Key, front Environments 변수 등록)을 완료함
- `gh`가 설치되어 있지 않아 직접 조회하지 못하고, 체크리스트로 확인을 요청함. 모두 확인됨

## 등록 결과

| 위치 | 등록한 것 |
|---|---|
| `choimory-gitops` → Settings → Deploy keys | `choimory-dev-member-api-actions`, `choimory-dev-front-actions` (Read/write) |
| `choimory-dev-member-api` → Repository secrets | `GITOPS_DEPLOY_KEY` |
| `choimory-dev-front` → Repository secrets | `GITOPS_DEPLOY_KEY` |
| `choimory-dev-front` → Environments → `stg` → Environment variables | `NEXT_PUBLIC_MEMBER_API_BASE_URL` = `https://staging-member-api.choimory.dev` |
| `choimory-dev-front` → Environments → `prod` → Environment variables | `NEXT_PUBLIC_MEMBER_API_BASE_URL` = `https://member-api.choimory.dev` |
| Mac | 키 파일 4개 삭제 |

- 주의사항
  - Deploy key가 Read-only면 Actions의 gitops push가 실패함
  - `GITOPS_DEPLOY_KEY`가 Environment secret에 있으면 member-api 워크플로가 읽지 못함 (member-api deploy job은 environment를 쓰지 않음)

## 이전 안내와 달라진 점

- Deploy Key 이름에 `choimory-dev-`를 붙임. 위 [직접 할 일: Deploy Key 등록](#직접-할-일-deploy-key-등록)도 같은 이름으로 수정함
- API 도메인을 서비스별 서브도메인으로 바꾸기로 하여 변수 값이 바뀜
  - 이전: `https://staging-api.choimory.dev/member`, `https://api.choimory.dev/member`
  - 변경: `https://staging-member-api.choimory.dev`, `https://member-api.choimory.dev`
  - 결정 과정은 [plan9.md API 도메인을 서비스별 서브도메인으로 변경](./plan9.md#api-도메인을-서비스별-서브도메인으로-변경) 참고

## 다음 진행

1. API 도메인 변경 4곳 수정 (Claude)
   - gitops `choimory-dev-member-api/base/httproute.yaml`: `/member` 경로 매칭과 URLRewrite 삭제
   - gitops `overlays/stg`, `overlays/prod`: hostname을 `staging-member-api.choimory.dev`, `member-api.choimory.dev`로 변경
   - front `deploy.yml` 97줄 주석 수정
2. gitops 커밋, `main` 반영 (직접)
3. 앱 레포 2개 커밋, `stg`와 `main` 머지 (직접). `deploy.yml`이 `main`에 있어야 Run workflow 버튼이 보임

---

# API 도메인 변경

> 시작

## 답변

- [plan9.md 다음 작업](./plan9.md#다음-작업-api-도메인-변경과-7단계-진행)의 1번(API 도메인 변경)을 진행함
- 커밋은 하지 않음

## 변경 내용

| 위치 | 변경 |
|---|---|
| gitops `choimory-dev-member-api/base/httproute.yaml` | `/member` 경로 매칭과 URLRewrite 삭제. 경로를 지정하지 않아 모든 경로가 member-api 8080으로 그대로 전달됨. 상단 주석 수정 |
| gitops `overlays/stg/kustomization.yaml` | hostname을 `staging-member-api.choimory.dev`로 변경, 주석 수정 |
| gitops `overlays/prod/kustomization.yaml` | hostname을 `member-api.choimory.dev`로 변경 |
| front `deploy.yml` 97줄 | 주석의 주소 예시를 새 도메인으로 수정 |

## 검증

- `kubectl kustomize`로 stg, prod overlay를 빌드함
- 변경 전(HEAD)과 비교한 차이는 HTTPRoute 두 군데뿐
  - hostname 변경
  - `/member` 매칭과 URLRewrite 삭제
  - Deployment, Service, SealedSecret은 그대로
- gitops와 앱 레포 2개의 매니페스트, 워크플로에 옛 도메인(`staging-api`, `api.choimory.dev`)이 남아 있지 않음
- front `deploy.yml`을 actionlint로 다시 검사했으며 오류 없음

## 커밋과 반영 (직접)

- gitops
  - 커밋할 것: HTTPRoute 관련 3개 파일, 문서 2개(plan9, task7)
  - 커밋 후 `main`에 반영. member-api가 아직 떠 있지 않아 영향 없음
- 앱 레포 2개

| 레포 | 커밋할 것 |
|---|---|
| `choimory-dev-front` | `deploy.yml` (수동 실행, gitops 경로, 주석) |
| `choimory-dev-member-api` | `deploy.yml` (수동 실행, gitops 경로) |

- 커밋 후 `dev` push, `stg` 머지, `main` 머지 순서로 진행
- `dev`에 push 안 된 커밋도 함께 올라감 (front 1개, member-api 3개)
- push하면 test job만 자동으로 돎. 테스트 결과를 먼저 확인

## gitops 반영 후 확인 명령어 (서버)

```bash
kubectl -n choimory-dev-stg get httproute choimory-dev-member-api -o jsonpath='{.spec.hostnames}{"\n"}'
kubectl -n choimory-dev-prod get httproute choimory-dev-member-api -o jsonpath='{.spec.hostnames}{"\n"}'
```

- 각각 `["staging-member-api.choimory.dev"]`, `["member-api.choimory.dev"]`가 나오면 정상
- 다음은 Actions에서 `stg` 브랜치로 Run workflow 실행
