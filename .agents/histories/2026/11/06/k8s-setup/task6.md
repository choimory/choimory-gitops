# 목차

- [개요](#개요)
- [진행 상황 확인](#진행-상황-확인)
  - [답변](#답변)
  - [진행 상황 (plan8 기준)](#진행-상황-plan8-기준)
  - [문제: SealedSecret 커밋이 main에 반영되지 않음](#문제-sealedsecret-커밋이-main에-반영되지-않음)
  - [다음 진행 (plan8 순서)](#다음-진행-plan8-순서)
  - [결정 필요 사항](#결정-필요-사항)
- [6단계 남은 작업](#6단계-남은-작업)
  - [답변](#답변-1)
  - [남은 것](#남은-것)
  - [6단계와 별개인 것](#6단계와-별개인-것)
- [main 반영과 확인 명령어](#main-반영과-확인-명령어)
  - [답변](#답변-2)
  - [브랜치 상태](#브랜치-상태)
  - [확인 명령어 (서버)](#확인-명령어-서버)
  - [완료 기준](#완료-기준)
  - [반영되지 않을 때 (서버)](#반영되지-않을-때-서버)
- [SealedSecret 일부만 반영됨](#sealedsecret-일부만-반영됨)
  - [답변](#답변-3)
  - [확인 결과](#확인-결과)
  - [원인](#원인)
  - [실행할 명령어 (서버)](#실행할-명령어-서버)
- [6단계 완료](#6단계-완료)
  - [답변](#답변-4)
  - [확인 결과](#확인-결과-1)
  - [지금 정리할 것 (직접)](#지금-정리할-것-직접)
  - [다음: 7단계 전 준비](#다음-7단계-전-준비)

---

# 개요

- plan8에서 정리한 남은 작업 중 6단계(SealedSecret) 마무리를 기록함
- SealedSecret 커밋을 `main`에 반영하고, 클러스터 반영 확인을 진행함
- 참고 문서: [plan8.md](./plan8.md)(남은 작업 순서), [task5.md](./task5.md)(5단계, 6단계 작업)

---

# 진행 상황 확인

> 261106 확인

## 답변

- `.agents/histories/2026/11/06/k8s-setup`의 문서와 git 상태를 확인함
- calendar 상태는 `TASK:k8s 구축 작업(k8s-setup)`이고, 가장 최근 문서는 plan8

## 진행 상황 (plan8 기준)

| 단계 | 상태 |
|---|---|
| 1~5단계 (member-api stg 프로필, Actions, gitops 작성, 클러스터 설치, ArgoCD) | 완료 |
| 6단계 SealedSecret | 레포에 작성함. 클러스터 반영 확인이 남음 |
| 7단계 배포 확인 | 시작 전 |
| 2차 외부 공개 (Cloudflare Tunnel) | 시작 전 |

## 문제: SealedSecret 커밋이 main에 반영되지 않음

- SealedSecret 4개, kustomization 4개, task5, plan8은 로컬 `dev`의 `147e434`(feat: 클러스터 설치, secret 적용 등)에 커밋되어 있었음
- `dev`가 `origin/dev`보다 1커밋 앞서 있어 push되지 않은 상태였음
- `origin/main`(`9475281`, release/1.0.1)에는 직전 커밋 `d44a8c5`(파드 이름변경)까지만 반영됨
- ArgoCD의 모든 Application이 `targetRevision: main`을 보므로 클러스터에는 Secret이 없음
  - 그 결과 redis는 계속 `CreateContainerConfigError`로 예상함
- 로컬 `stg`도 `origin/stg`보다 3커밋 앞서 있었고, `origin/stg`는 `dda819e`에 머물러 있었음

## 다음 진행 (plan8 순서)

1. 6단계 마무리
   - `dev` push 후 release로 `main`까지 반영(직접)
   - 1~3분 뒤 서버에서 확인 명령어 실행
   - 완료 기준: Secret 생성, `choimory-dev-redis-0`이 `Running`, `*-data`가 `Healthy`
2. 7단계 전 준비(직접)
   - Deploy Key 등록, front Environments 변수(stg, prod) 등록
   - 앱 레포를 `stg`에 머지한 뒤 GHCR 패키지를 public으로 전환
3. 7단계 1차 확인
   - stg에서 Pod, actuator, 로그인 API, ArgoCD 자동 배포 확인
   - 그다음 prod를 같은 방식으로 진행

## 결정 필요 사항

- 디렉토리 이름 변경 여부(`front`, `member-api`, `data` → `choimory-dev-front`, `choimory-dev-member-api`, `database`)
  - 앱 레포를 `stg`에 머지하기 전에 정해야 Actions 경로를 한 번에 맞출 수 있음
- 답변 대기 중인 문서 항목 4개
  - task4 clone 경로
  - plan3 RAM 값을 26Gi로 정정할지
  - plan1 노션 URL
  - 노션 TIL의 호스트명

---

# 6단계 남은 작업

> 6단계는 gitops 리포를 메인브랜치에 올리는 거만 남은거임?

## 답변

- 레포 작업은 끝났고 `main` 반영만 남음
- 다만 반영 후 서버에서 확인하는 단계까지 해야 6단계가 끝남

## 남은 것

1. `main` 반영(직접)
   - `dev` push 후 release를 거쳐 `main`에 머지
   - 반영할 커밋은 `147e434` 하나(SealedSecret 4개, kustomization 4개, task5, plan8)
2. 반영 확인(서버)
   - push 후 1~3분 뒤 plan8의 확인 명령어 실행
   - 완료 기준
     - stg, prod 모두에 Secret `choimory-dev-redis-secret`, `choimory-dev-member-api-secret` 생성
     - `choimory-dev-redis-0`이 `Running`
     - `choimory-dev-{stg,prod}-data`가 `Healthy`

## 6단계와 별개인 것

- front, member-api의 `ImagePullBackOff`는 6단계가 끝나도 그대로임
  - 이미지 태그가 `*-bootstrap` 임시값이라서이며, 7단계 준비에서 앱 레포를 `stg`에 머지해야 풀림
- plan8의 "중간에 정리할 것"은 6단계 완료 조건은 아니지만, Secret 반영 확인 후 바로 하는 것을 추천
  - 서버의 `~/sealed`, Mac의 `~/Downloads/sealed` 삭제
  - `~/sealed-secrets-key-backup.yaml`을 서버 밖 안전한 곳에 보관

---

# main 반영과 확인 명령어

> 일단 gitops 메인브랜치 push 했음

## 답변

- `git fetch` 후 `origin/main`에 `147e434`가 포함된 것을 확인함

## 브랜치 상태

| 브랜치 | 커밋 |
|---|---|
| `main` | `cae59cf` (Merge branch 'release/1.0.2'), `147e434` 포함 |
| `dev` | `147e434`, `origin/dev`와 같음 |
| `stg` | `147e434`, `origin/stg`와 같음 |

## 확인 명령어 (서버)

- ArgoCD가 바뀐 내용을 가져가는 데 1~3분 걸리므로, 그 뒤에 실행

```bash
kubectl get sealedsecrets -n choimory-dev-stg
kubectl get sealedsecrets -n choimory-dev-prod
kubectl get secret -n choimory-dev-stg | grep choimory-dev
kubectl get secret -n choimory-dev-prod | grep choimory-dev
kubectl -n choimory-dev-stg get pods
kubectl -n choimory-dev-prod get pods
kubectl -n infrastructure get applications | grep choimory-dev
```

## 완료 기준

- stg, prod 모두에 Secret `choimory-dev-redis-secret`, `choimory-dev-member-api-secret`이 있음
- `choimory-dev-redis-0`이 `Running`
- `choimory-dev-{stg,prod}-data`가 `Healthy`
- front, member-api는 이번에도 `ImagePullBackOff`가 정상(이미지 태그가 임시값)

## 반영되지 않을 때 (서버)

- 3분이 지나도 반영되지 않으면 ArgoCD가 바뀐 커밋을 아직 못 가져온 것일 수 있음
- 아래 명령어로 root를 바로 다시 읽게 함

```bash
kubectl -n infrastructure annotate application root argocd.argoproj.io/refresh=hard --overwrite
```

---

# SealedSecret 일부만 반영됨

> ```
> choimory@choimory-server:~$ kubectl get sealedsecrets -n choimory-dev-stg
> No resources found in choimory-dev-stg namespace.
> choimory@choimory-server:~$ kubectl get sealedsecrets -n choimory-dev-prod
> NAME                             STATUS   SYNCED   AGE
> choimory-dev-member-api-secret            True     80s
> choimory@choimory-server:~$ kubectl get secret -n choimory-dev-stg | grep choimory-dev
> choimory-dev-postgres-app           kubernetes.io/basic-auth   11     4d21h
> choimory-dev-postgres-ca            Opaque                     2      4d21h
> choimory-dev-postgres-replication   kubernetes.io/tls          2      4d21h
> choimory-dev-postgres-server        kubernetes.io/tls          2      4d21h
> choimory@choimory-server:~$ kubectl get secret -n choimory-dev-prod | grep choimory-dev
> choimory-dev-member-api-secret      Opaque                     1      87s
> choimory-dev-postgres-app           kubernetes.io/basic-auth   11     4d21h
> choimory-dev-postgres-ca            Opaque                     2      4d21h
> choimory-dev-postgres-replication   kubernetes.io/tls          2      4d21h
> choimory-dev-postgres-server        kubernetes.io/tls          2      4d21h
> choimory@choimory-server:~$ kubectl -n choimory-dev-stg get pods
> NAME                                       READY   STATUS                       RESTARTS   AGE
> choimory-dev-front-85469c8fb5-klvlp        0/1     ImagePullBackOff             0          4d21h
> choimory-dev-member-api-69cd5894dd-vcmbc   0/1     ImagePullBackOff             0          4d21h
> choimory-dev-postgres-1                    1/1     Running                      0          4d21h
> choimory-dev-redis-0                       0/1     CreateContainerConfigError   0          4d21h
> choimory@choimory-server:~$ kubectl -n choimory-dev-prod get pods
> NAME                                      READY   STATUS                       RESTARTS   AGE
> choimory-dev-front-56889d449f-x6z6t       0/1     ImagePullBackOff             0          4d21h
> choimory-dev-member-api-dd677d978-pqfk6   0/1     ImagePullBackOff             0          4d21h
> choimory-dev-postgres-1                   1/1     Running                      0          4d21h
> choimory-dev-redis-0                      0/1     CreateContainerConfigError   0          4d21h
> choimory@choimory-server:~$ kubectl -n infrastructure get applications | grep choimory-dev
> choimory-dev                   Synced        Healthy
> choimory-dev-prod-data         Synced        Progressing
> choimory-dev-prod-front        Synced        Degraded
> choimory-dev-prod-member-api   Synced        Degraded
> choimory-dev-prod-namespaces   Synced        Healthy
> choimory-dev-stg-data          Synced        Progressing
> choimory-dev-stg-front         Synced        Degraded
> choimory-dev-stg-member-api    Synced        Degraded
> choimory-dev-stg-namespaces    Synced        Healthy
> ```

## 답변

- 레포 설정은 문제없음
- prod member-api Secret만 먼저 반영됐고, 나머지 세 앱은 ArgoCD가 아직 새 커밋을 가져오지 않은 것으로 판단함

## 확인 결과

| 대상 | 결과 |
|---|---|
| prod member-api SealedSecret, Secret | 생성됨 (80초 전) |
| stg member-api SealedSecret | 없음 |
| stg, prod redis SealedSecret | 없음 |
| redis Pod | 둘 다 `CreateContainerConfigError` (Secret이 없어서) |
| `*-data` | `Progressing` (redis가 안 떠서) |
| front, member-api | `ImagePullBackOff` (임시 태그라 예상대로) |

## 원인

- 레포의 SealedSecret 4개를 확인했으며 서로 다른 점이 없음
  - 이름, 네임스페이스, kustomization `resources` 등록 방식이 4개 모두 같음
  - 두 ApplicationSet(`choimory-dev-apps`, `choimory-dev-protected`) 모두 `main`을 봄
- 설정 문제가 아니라, ArgoCD가 앱마다 다른 시점에 Git을 다시 읽기 때문으로 판단함
  - prod member-api만 먼저 새 커밋을 읽었고, 나머지 세 앱은 아직 읽지 않음
- 나머지 앱이 `Synced`로 보이는 것도 같은 이유로, 새 커밋이 아닌 예전 커밋과 비교한 결과임
- 이 판단은 레포 파일을 보고 한 것이며, 클러스터에서 직접 확인하지는 않음

## 실행할 명령어 (서버)

- 기다리기보다 세 앱이 바로 다시 읽도록 함

```bash
for app in choimory-dev-stg-member-api choimory-dev-stg-data choimory-dev-prod-data; do
  kubectl -n infrastructure annotate application "$app" argocd.argoproj.io/refresh=hard --overwrite
done
```

- 30초~1분 뒤 확인

```bash
kubectl get sealedsecrets -n choimory-dev-stg
kubectl get sealedsecrets -n choimory-dev-prod
kubectl -n choimory-dev-stg get pods
kubectl -n choimory-dev-prod get pods
kubectl -n infrastructure get applications | grep choimory-dev
```

- Secret이 생기면 kubelet이 다시 시도하면서 redis Pod가 저절로 `Running`이 되므로 재시작할 필요 없음
- 그래도 SealedSecret이 생기지 않으면 아래 결과로 원인을 확인

```bash
kubectl -n infrastructure get application choimory-dev-stg-data -o jsonpath='{.status.sync.revision}{"\n"}{.status.conditions}{"\n"}'
```

---

# 6단계 완료

> ```
> choimory@choimory-server:~$ for app in choimory-dev-stg-member-api choimory-dev-stg-data choimory-dev-prod-data; do
>   kubectl -n infrastructure annotate application "$app" argocd.argoproj.io/refresh=hard --overwrite
> done
> application.argoproj.io/choimory-dev-stg-member-api annotated
> application.argoproj.io/choimory-dev-stg-data annotated
> application.argoproj.io/choimory-dev-prod-data annotated
> choimory@choimory-server:~$ kubectl get sealedsecrets -n choimory-dev-stg
> NAME                             STATUS   SYNCED   AGE
> choimory-dev-member-api-secret            True     76s
> choimory-dev-redis-secret                 True     2m23s
> choimory@choimory-server:~$ kubectl get sealedsecrets -n choimory-dev-prod
> NAME                             STATUS   SYNCED   AGE
> choimory-dev-member-api-secret            True     3m51s
> choimory-dev-redis-secret                 True     2m3s
> choimory@choimory-server:~$ kubectl -n choimory-dev-stg get pods
> NAME                                       READY   STATUS             RESTARTS   AGE
> choimory-dev-front-85469c8fb5-klvlp        0/1     ImagePullBackOff   0          4d21h
> choimory-dev-member-api-69cd5894dd-vcmbc   0/1     ImagePullBackOff   0          4d21h
> choimory-dev-postgres-1                    1/1     Running            0          4d21h
> choimory-dev-redis-0                       1/1     Running            0          4d21h
> choimory@choimory-server:~$ kubectl -n choimory-dev-prod get pods
> NAME                                      READY   STATUS             RESTARTS   AGE
> choimory-dev-front-56889d449f-x6z6t       0/1     ImagePullBackOff   0          4d21h
> choimory-dev-member-api-dd677d978-pqfk6   0/1     ImagePullBackOff   0          4d21h
> choimory-dev-postgres-1                   1/1     Running            0          4d21h
> choimory-dev-redis-0                      1/1     Running            0          4d21h
> choimory@choimory-server:~$ kubectl -n infrastructure get applications | grep choimory-dev
> choimory-dev                   Synced        Healthy
> choimory-dev-prod-data         Synced        Healthy
> choimory-dev-prod-front        Synced        Degraded
> choimory-dev-prod-member-api   Synced        Degraded
> choimory-dev-prod-namespaces   Synced        Healthy
> choimory-dev-stg-data          Synced        Healthy
> choimory-dev-stg-front         Synced        Degraded
> choimory-dev-stg-member-api    Synced        Degraded
> choimory-dev-stg-namespaces    Synced        Healthy
> ```

## 답변

- 6단계 완료 기준 세 가지를 모두 충족하여 6단계를 완료함

## 확인 결과

| 완료 기준 | 결과 |
|---|---|
| stg, prod에 Secret `choimory-dev-redis-secret`, `choimory-dev-member-api-secret` 생성 | 4개 모두 생성, `SYNCED True` |
| `choimory-dev-redis-0`이 `Running` | stg, prod 모두 `1/1 Running`, 재시작 0회 |
| `choimory-dev-{stg,prod}-data`가 `Healthy` | 둘 다 `Healthy` |

- front, member-api는 여전히 `ImagePullBackOff`, `Degraded`
  - 이미지 태그가 `*-bootstrap` 임시값이라 예상한 상태이며, 7단계 준비에서 풀림
- stg redis Secret은 생성된 지 2분 23초로, refresh 전에 이미 만들어지고 있었음
  - ArgoCD가 이미 새 커밋을 가져오던 중이었던 것으로 보이며, 앞서 판단한 대로 앱마다 반영 시점이 달랐던 것이 원인

## 지금 정리할 것 (직접)

- 서버의 `~/sealed`, Mac의 `~/Downloads/sealed` 삭제
- `~/sealed-secrets-key-backup.yaml`을 서버 밖 안전한 곳(비밀번호 관리자 등)에 보관. 레포에는 넣지 않음
- 아직 안 했다면 ArgoCD 화면 로그인 후 `argocd-initial-admin-secret` 삭제

## 다음: 7단계 전 준비

- 앱 레포를 `stg`에 머지하기 전에 디렉토리 이름 변경 여부를 먼저 정해야 함
  - 머지한 뒤에 바꾸면 앱 레포 `deploy.yml`의 경로를 다시 고쳐야 함
- 선택지
  1. 이름 변경을 지금 할 경우: gitops의 ApplicationSet과 앱 레포 2개의 `deploy.yml` 경로를 같이 바꾸고, gitops를 먼저 `main`에 반영한 뒤 앱 레포를 머지
  2. 이름 변경을 미룰 경우: Deploy Key 등록, front Environments 변수 등록, 앱 레포 `stg` 머지, GHCR public 전환 순서로 바로 진행
