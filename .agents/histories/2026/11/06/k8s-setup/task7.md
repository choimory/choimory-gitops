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
