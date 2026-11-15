# 목차

- [개요](#개요)
- [남은 작업](#남은-작업)
  - [답변](#답변)
  - [1. 지금 정리할 것 (직접)](#1-지금-정리할-것-직접)
  - [2. 7단계 전에 정할 것](#2-7단계-전에-정할-것)
  - [3. 7단계 전 준비 (직접)](#3-7단계-전-준비-직접)
  - [4. 7단계 1차 확인](#4-7단계-1차-확인)
  - [5. 2차 외부 공개](#5-2차-외부-공개)
  - [6. 범위를 정할 것 (선택)](#6-범위를-정할-것-선택)
  - [7. 답변 대기 중인 문서 항목](#7-답변-대기-중인-문서-항목)
  - [아직 검증하지 못한 것](#아직-검증하지-못한-것)
- [추가1](#추가1)
- [추가1 분석](#추가1-분석)
  - [답변](#답변-1)
  - [1. 디렉토리 구성 변경](#1-디렉토리-구성-변경)
    - [바뀌는 구조](#바뀌는-구조)
    - [Application 이름](#application-이름)
    - [데이터 안전성](#데이터-안전성)
    - [같이 고칠 것](#같이-고칠-것)
    - [확인할 것](#확인할-것)
  - [2. Actions 빌드 수동 실행](#2-actions-빌드-수동-실행)
    - [바꾸는 내용](#바꾸는-내용)
    - [같이 막아야 할 것](#같이-막아야-할-것)
    - [정할 것](#정할-것)
  - [3. 7단계 진행 순서](#3-7단계-진행-순서)
  - [4. 그 외](#4-그-외)
  - [결정 필요 사항](#결정-필요-사항)
- [추가2](#추가2)
- [추가2 분석](#추가2-분석)
  - [답변](#답변-2)
  - [디렉토리 이름과 Pod 이름](#디렉토리-이름과-pod-이름)
  - [테스트만 자동 (B)](#테스트만-자동-b)
  - [시작하면 진행할 순서](#시작하면-진행할-순서)
- [구조 변경 반영 확인과 deploy.yml 수정안](#구조-변경-반영-확인과-deployyml-수정안)
  - [답변](#답변-3)
  - [확인 결과](#확인-결과)
  - [다음: 앱 레포 deploy.yml 수정 (5번)](#다음-앱-레포-deployyml-수정-5번)
- [Deploy Key 이름](#deploy-key-이름)
  - [답변](#답변-4)
  - [키 생성 (Mac)](#키-생성-mac)
  - [등록 위치](#등록-위치)
  - [등록 후 삭제](#등록-후-삭제)
- [Environments 만드는 위치](#environments-만드는-위치)
  - [답변](#답변-5)
  - [만드는 순서](#만드는-순서)
  - [주의](#주의)
- [Environment secrets와 variables](#environment-secrets와-variables)
  - [답변](#답변-6)
  - [등록할 값](#등록할-값)
  - [variable에 넣는 이유](#variable에-넣는-이유)
- [민감한 값의 위치](#민감한-값의-위치)
  - [답변](#답변-7)
  - [variable과 secret의 차이](#variable과-secret의-차이)
  - [이 프로젝트에서 값이 놓이는 곳](#이-프로젝트에서-값이-놓이는-곳)
  - [NEXT_PUBLIC_ 주의](#next_public_-주의)
- [모두 secret에 넣는 경우](#모두-secret에-넣는-경우)
  - [답변](#답변-8)
  - [secret에 넣으면 생기는 일](#secret에-넣으면-생기는-일)
  - [기준](#기준)
- [Deploy Key를 Environment secret에 넣는 경우](#deploy-key를-environment-secret에-넣는-경우)
  - [답변](#답변-9)
  - [Environment secret에 넣으려면](#environment-secret에-넣으려면)
  - [얻는 것](#얻는-것)
  - [레포 Secret을 추천하는 이유](#레포-secret을-추천하는-이유)
  - [정리](#정리)
- [GitHub secret, variable 레벨 정리](#github-secret-variable-레벨-정리)
  - [답변](#답변-10)
  - [기준](#기준-1)
  - [GitHub에 넣는 값은 빌드할 때 쓰는 값만](#github에-넣는-값은-빌드할-때-쓰는-값만)
- [SSH 키 사용처](#ssh-키-사용처)
  - [답변](#답변-11)
  - [사용되는 곳](#사용되는-곳)
  - [전체 흐름에서의 위치](#전체-흐름에서의-위치)
  - [왜 필요한가](#왜-필요한가)
  - [사용되지 않는 곳](#사용되지-않는-곳)
- [Deploy Key의 인증 대상](#deploy-key의-인증-대상)
  - [답변](#답변-12)
  - [계정 SSH 키와 비교](#계정-ssh-키와-비교)
  - [GitHub가 이 키로 판단하는 것](#github가-이-키로-판단하는-것)
  - [커밋 작성자](#커밋-작성자)
  - [이렇게 하는 이유](#이렇게-하는-이유)
- [API 도메인을 서비스별 서브도메인으로 변경](#api-도메인을-서비스별-서브도메인으로-변경)
  - [답변](#답변-13)
  - [바뀌는 도메인](#바뀌는-도메인)
  - [문제없는 것](#문제없는-것)
  - [고칠 것](#고칠-것)
  - [front Environment variable 값 변경](#front-environment-variable-값-변경)
  - [비용](#비용)
  - [진행 방식](#진행-방식)
- [나중에 API 게이트웨이로 전환](#나중에-api-게이트웨이로-전환)
  - [답변](#답변-14)
  - [전환할 때 바뀌는 것](#전환할-때-바뀌는-것)
  - [미리 알아둘 점](#미리-알아둘-점)
- [경로 방식에서 서비스 이름이 겹치는 문제](#경로-방식에서-서비스-이름이-겹치는-문제)
  - [답변](#답변-15)
  - [겹치는 이유](#겹치는-이유)
  - [서브도메인 방식](#서브도메인-방식)
  - [게이트웨이에서 겹침을 피하는 방법](#게이트웨이에서-겹침을-피하는-방법)
- [다음 작업: API 도메인 변경과 7단계 진행](#다음-작업-api-도메인-변경과-7단계-진행)
  - [답변](#답변-16)
  - [1. API 도메인 변경 (Claude)](#1-api-도메인-변경-claude)
  - [2. gitops 커밋과 main 반영 (직접)](#2-gitops-커밋과-main-반영-직접)
  - [3. 앱 레포 커밋과 머지 (직접)](#3-앱-레포-커밋과-머지-직접)
  - [4. stg 배포 실행 (직접)](#4-stg-배포-실행-직접)
  - [5. GHCR 패키지 public 전환 (직접)](#5-ghcr-패키지-public-전환-직접)
  - [6. stg 1차 확인](#6-stg-1차-확인)
  - [7. prod 배포와 확인](#7-prod-배포와-확인)

---

# 개요

- task6에서 6단계(SealedSecret)를 완료함
- 이 문서는 6단계 이후 남은 작업을 정리함
- 참고 문서: [plan8.md](./plan8.md)(이전 남은 작업 정리), [task6.md](./task6.md)(6단계 완료)

---

# 남은 작업

> 이제 뭐뭐남았지

## 답변

- 6단계까지 끝났고, 남은 것은 7단계(실제 배포 확인)와 2차 외부 공개
- 그 외에 정리할 것, 정해야 할 것, 범위를 정할 것이 있음
- 바로 다음 할 일은 2번, 디렉토리 이름 변경 여부를 정하는 것

## 1. 지금 정리할 것 (직접)

- 서버의 `~/sealed`, Mac의 `~/Downloads/sealed` 삭제
- `~/sealed-secrets-key-backup.yaml`을 서버 밖 안전한 곳에 보관
- ArgoCD 화면 로그인 후 `argocd-initial-admin-secret` 삭제 (아직 안 했다면)

## 2. 7단계 전에 정할 것

- 디렉토리 이름 변경 여부 (`front`, `member-api`, `data` → `choimory-dev-front`, `choimory-dev-member-api`, `database`)
  - 앱 레포를 `stg`에 머지하기 전에 정해야 함
  - 하기로 하면 gitops의 ApplicationSet과 앱 레포 2개의 `deploy.yml` 경로를 같이 바꿈
  - 검토 내용은 [plan8.md 디렉토리 이름 변경](./plan8.md#디렉토리-이름-변경) 참고

## 3. 7단계 전 준비 (직접)

1. Deploy Key 등록
   - gitops 레포에 쓰기 권한이 있는 Deploy Key 추가
   - 같은 개인키를 앱 레포 2개의 Secret `GITOPS_DEPLOY_KEY`로 등록
2. front의 GitHub Environments 변수 등록 (stg, prod)
3. 앱 레포를 `stg`에 머지
   - Actions가 이미지를 빌드하고, gitops의 이미지 태그를 실제 값으로 커밋함
4. GHCR 패키지 2개를 public으로 전환
   - 첫 빌드가 끝난 뒤에 진행. 비공개로 두면 `ImagePullBackOff`가 계속됨

## 4. 7단계 1차 확인

- stg 먼저 확인
  - Pod 정상 여부, 재시작 횟수
  - actuator(8081) liveness, readiness 응답
  - front, member-api 헬스체크
  - 회원가입, 로그인, 토큰 재발급 동작. Redis에 토큰이 저장되는지
  - ArgoCD가 Actions의 태그 커밋을 받아 자동으로 배포하는지
- 그다음 prod를 같은 방식으로 진행 (앱 레포 `main` 머지)

## 5. 2차 외부 공개

- cloudflared를 클러스터 안에 배포하고 Cloudflare Tunnel로 도메인 4개를 연결
- 1차 확인이 끝난 뒤 새 plan 문서에서 계획함

## 6. 범위를 정할 것 (선택)

- etcd와 `/etc/kubernetes/pki` 백업. 데이터가 쌓이기 전에 하는 것을 추천
- PostgreSQL 백업 (CloudNativePG 백업 설정)
- Uptime Kuma, Homepage 연동
- 배포 실패 알림
- HPA 테스트

## 7. 답변 대기 중인 문서 항목

- task4의 clone 경로를 실제 경로로 고칠지
- plan3의 RAM 값을 26Gi로 고칠지
- `plan1.md:37`의 노션 URL 처리
- 노션 TIL에 서버 호스트명을 그대로 둘지

## 아직 검증하지 못한 것

- Mac에서 Tailscale로 `kubectl` 접속
- Actions부터 ArgoCD 자동 배포까지의 흐름, actuator 응답. 이 두 가지는 7단계에서 확인됨

---

# 추가1

- projects/choimory-dev 하위 디렉토리 구성 변경
  - data안에 있는 postgres, redis를 각각의 디렉토리로 분리
  - choimory-dev/argocd, postgres, redis, choimory-dev-front, choimory-dev-member-api...
- github actions 빌드를 수동으로 변경
  - stg, prod push 후 github의 actions 탭 들어가서 직접 run workflow 같은거 눌러야 빌드 실행되는식으로
- 7단계 진행
  - deploy key 등록 등 내가 해야하는거 있으면 작업 중간중간 차례되면 알려주면 처리해줌
- 그 외
  - task4 clone 경로, plan3의 ram값 수정
  - plan1 노션 url 유지
  - 노션 til 서버 호스트명 유지
  - mac에서 kubectl 명령어 칠 생각없고, 앞으로도 서버 ssh 원격접속해서 진행할거임

---

# 추가1 분석

> plan9 추가1 섹션 확인

## 답변

- 추가1의 항목별 분석과 진행 계획을 정리함
- 아직 파일은 수정하지 않음

## 1. 디렉토리 구성 변경

### 바뀌는 구조

```
projects/choimory-dev/
├── argocd/                   # 그대로
├── namespaces/               # 그대로
├── postgres/                 # data에서 분리
├── redis/                    # data에서 분리 (redis SealedSecret도 같이 이동)
├── choimory-dev-front/       # front에서 이름 변경
└── choimory-dev-member-api/  # member-api에서 이름 변경
```

### Application 이름

- ApplicationSet 요소에 `dir`을 추가해 디렉토리 이름과 Application 이름을 분리하는 것을 추천
- 분리하지 않으면 `choimory-dev-stg-choimory-dev-front`처럼 이름이 겹침

| 디렉토리 | Application 이름 |
|---|---|
| `choimory-dev-front` | `choimory-dev-{env}-front` (유지) |
| `choimory-dev-member-api` | `choimory-dev-{env}-member-api` (유지) |
| `postgres` | `choimory-dev-{env}-postgres` (신규) |
| `redis` | `choimory-dev-{env}-redis` (신규) |
| `data` | `choimory-dev-{env}-data` 삭제 |

### 데이터 안전성

- `data`를 나누면 ArgoCD 입장에서는 `*-data` 앱이 삭제되고 `*-postgres`, `*-redis` 앱이 새로 생김. 그래서 DB가 지워지지 않는지 확인함
- `choimory-dev-protected` ApplicationSet에 `preserveResourcesOnDeletion: true`가 있어, `*-data` 앱이 삭제되어도 Cluster, StatefulSet, PVC는 클러스터에 남음
- 리소스 이름은 그대로임 (`choimory-dev-postgres`, `choimory-dev-redis`, Secret 이름 등)
  - 새 앱은 기존 리소스를 그대로 넘겨받고 관리 표시만 바꾸므로, Pod가 재생성되지 않을 것으로 봄
- SealedSecret은 이름과 네임스페이스로만 풀리므로 파일 위치를 옮겨도 다시 만들 필요 없음
- 주의: 옛 앱과 새 앱이 같은 리소스를 동시에 관리하면 서로 충돌함. 그래서 디렉토리 이동과 ApplicationSet 변경을 한 커밋에 넣어야 함
- 데이터를 다루는 작업이라 반영 전에 PostgreSQL 덤프를 추천

### 같이 고칠 것

- ApplicationSet 2개
- 매니페스트 주석의 경로
- 앱 레포 2개의 `deploy.yml` 경로 (gitops 레포에서 경로를 참조하는 곳은 주석 말고는 없음)

### 확인할 것

- 요청한 이름대로면 `postgres`, `redis`에는 접두어가 없고 `choimory-dev-front`, `choimory-dev-member-api`에는 접두어가 있음. 의도한 것이 맞으면 그대로 진행

## 2. Actions 빌드 수동 실행

### 바꾸는 내용

- 트리거를 `push` 대신 `workflow_dispatch`로 변경
- Actions 탭에서 "Run workflow"를 누르고 브랜치(`stg`, `main`)를 고르는 방식

### 같이 막아야 할 것

- 브랜치 제한
  - 지금 PROFILE은 "`main`이면 prod, 아니면 stg"로 정함
  - 그래서 수동 실행에서 `dev` 같은 다른 브랜치를 고르면 그 코드가 stg로 배포됨
  - `stg`와 `main`이 아니면 실패하도록 조건을 추가함
- 버튼이 보이는 조건
  - "Run workflow" 버튼은 워크플로 파일이 기본 브랜치(`main`)에 있어야 나타남
  - 지금 `deploy.yml`은 앱 레포의 `dev`에만 있어 `main`까지 머지해야 버튼이 보임
  - 수동 실행으로 바뀌면 `main`에 머지해도 prod 배포가 자동으로 시작되지 않으므로 이 순서가 안전해짐

### 정할 것

- 수동으로 바꾸면 테스트도 실행할 때만 돌아감
  - A: 둘 다 수동. 테스트와 빌드 모두 수동 실행에서만 돎
  - B: 테스트만 자동 (추천). 테스트는 push 때 자동으로 돌고, 빌드와 배포만 수동. 머지 전에 테스트 실패를 바로 알 수 있음

## 3. 7단계 진행 순서

- 직접 해야 할 일은 차례가 되면 알려줌

| 순서 | 작업 | 담당 |
|---|---|---|
| 1 | PostgreSQL 덤프 (서버) | 직접 |
| 2 | gitops 구조 변경 작성 | Claude |
| 3 | gitops 커밋, `main` 반영 | 직접 |
| 4 | 서버에서 앱 전환 확인 (`*-data` 삭제, `*-postgres`, `*-redis` Healthy, Pod 재시작 없음) | 직접 실행, 결과 확인은 Claude |
| 5 | 앱 레포 2개 `deploy.yml` 수정 (경로, 수동 트리거) | Claude |
| 6 | Deploy Key 등록, front Environments 변수 등록 | 직접 |
| 7 | 앱 레포 `stg`, `main` 머지 (`main`은 버튼 표시용) | 직접 |
| 8 | Actions에서 `stg` 실행, GHCR public 전환 | 직접 |
| 9 | stg 1차 확인 | 명령어는 Claude, 실행은 직접 |
| 10 | Actions에서 `main` 실행, prod 확인 | 직접 실행, 확인은 같이 |

## 4. 그 외

- task4 clone 경로, plan3 RAM 값: 기존 문서를 고치는 것이라 요청한 두 곳만 수정
- plan1 노션 URL, 노션 TIL 호스트명: 그대로 둠
- Mac에서 kubectl 접속: "아직 검증하지 못한 것"에서 제외하고, 명령어는 모두 서버 기준으로 안내

## 결정 필요 사항

1. `postgres`, `redis`를 접두어 없는 이름으로 진행할지
2. 테스트를 A(둘 다 수동)로 할지, B(테스트만 자동, 추천)로 할지
3. 위 순서로 시작해도 될지
   - 시작하면 문서 2곳(task4, plan3)을 먼저 고침
   - 그다음 1번(PostgreSQL 덤프) 명령어를 안내함

---

# 추가2

- 디렉토리명은 projects/choimory-dev/postgres, redis인데 파드명은 choimory-dev-postgres, redis
- 테스트만 자동으로

---

# 추가2 분석

> 추가2 섹션 확인

## 답변

- 결정 필요 사항 1, 2번이 정해졌고, 남은 것은 3번(시작 여부)

## 디렉토리 이름과 Pod 이름

- 디렉토리는 `projects/choimory-dev/postgres`, `projects/choimory-dev/redis`로 만듦
- Pod 이름은 이미 `choimory-dev-postgres-1`, `choimory-dev-redis-0`임
  - 리소스 이름은 매니페스트의 `metadata.name`으로 정해지고 디렉토리 이름과 상관없으므로 고칠 것이 없음
  - 이름이 바뀌지 않으므로 새 앱이 기존 리소스를 그대로 넘겨받음
- Application 이름은 `choimory-dev-{env}-postgres`, `choimory-dev-{env}-redis`

## 테스트만 자동 (B)

- `deploy.yml` 한 파일 안에서 실행 방식을 나눔

| 실행 방식 | test job | deploy job |
|---|---|---|
| push (`dev`, `stg`, `main`) | 자동 실행 | 실행 안 함 |
| Actions 탭에서 Run workflow | 실행 | test 통과 후 실행 |

- deploy job 조건
  - 수동 실행일 때만 돎 (`github.event_name == 'workflow_dispatch'`)
  - 브랜치가 `stg`나 `main`일 때만 돎. 그 외 브랜치를 고르면 실패시켜 `dev` 코드가 stg로 배포되는 것을 막음
- push 테스트 대상은 `dev`, `stg`, `main` 세 브랜치를 추천

## 시작하면 진행할 순서

1. 문서 2곳 수정
   - task4: clone 경로를 `/data/dev/choimory/repo/choimory-gitops`로 변경
   - plan3: RAM 기준을 26Gi로 변경
2. PostgreSQL 덤프 명령어 안내 (서버에서 직접 실행)
3. 덤프 완료 후 gitops 구조 변경 작성. 작성 후 변경 내용을 설명하고, 커밋은 직접 진행

---

# 구조 변경 반영 확인과 deploy.yml 수정안

> ```
> choimory@choimory-server:/data/storage/media$   kubectl -n infrastructure get applications | grep choimory-dev
> choimory-dev                   Synced        Healthy
> choimory-dev-prod-front        Synced        Degraded
> choimory-dev-prod-member-api   Synced        Degraded
> choimory-dev-prod-namespaces   Synced        Healthy
> choimory-dev-prod-postgres     Synced        Healthy
> choimory-dev-prod-redis        Synced        Healthy
> choimory-dev-stg-front         Synced        Degraded
> choimory-dev-stg-member-api    Synced        Degraded
> choimory-dev-stg-namespaces    Synced        Healthy
> choimory-dev-stg-postgres      Synced        Healthy
> choimory-dev-stg-redis         Synced        Healthy
> choimory@choimory-server:/data/storage/media$   kubectl -n choimory-dev-stg get pods,pvc
> NAME                                           READY   STATUS             RESTARTS   AGE
> pod/choimory-dev-front-85469c8fb5-klvlp        0/1     ImagePullBackOff   0          4d23h
> pod/choimory-dev-member-api-69cd5894dd-vcmbc   0/1     ImagePullBackOff   0          4d23h
> pod/choimory-dev-postgres-1                    1/1     Running            0          4d23h
> pod/choimory-dev-redis-0                       1/1     Running            0          4d23h
>
> NAME                                              STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS              VOLUMEATTRIBUTESCLASS   AGE
> persistentvolumeclaim/choimory-dev-postgres-1     Bound    pvc-<STG_POSTGRES_PV_ID>   5Gi        RWO            local-path-choimory-dev   <unset>                 4d23h
> persistentvolumeclaim/data-choimory-dev-redis-0   Bound    pvc-<STG_REDIS_PV_ID>   1Gi        RWO            local-path-choimory-dev   <unset>                 4d23h
> choimory@choimory-server:/data/storage/media$   kubectl -n choimory-dev-prod get pods,pvc
> NAME                                          READY   STATUS             RESTARTS   AGE
> pod/choimory-dev-front-56889d449f-x6z6t       0/1     ImagePullBackOff   0          4d23h
> pod/choimory-dev-member-api-dd677d978-pqfk6   0/1     ImagePullBackOff   0          4d23h
> pod/choimory-dev-postgres-1                   1/1     Running            0          4d23h
> pod/choimory-dev-redis-0                      1/1     Running            0          4d23h
>
> NAME                                              STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS              VOLUMEATTRIBUTESCLASS   AGE
> persistentvolumeclaim/choimory-dev-postgres-1     Bound    pvc-<PROD_POSTGRES_PV_ID>   10Gi       RWO            local-path-choimory-dev   <unset>                 4d23h
> persistentvolumeclaim/data-choimory-dev-redis-0   Bound    pvc-<PROD_REDIS_PV_ID>   1Gi        RWO            local-path-choimory-dev   <unset>                 4d23h
> choimory@choimory-server:/data/storage/media$   kubectl -n choimory-dev-stg get
> You must specify the type of resource to get. Use "kubectl api-resources" for a complete list of supported resources.
>
> error: Required resource not specified.
> Use "kubectl explain <resource>" for a detailed description of that resource (e.g. kubectl explain pods).
> See 'kubectl get -h' for help and examples
> choimory@choimory-server:/data/storage/media$   cluster.postgresql.cnpg.io,statefulset,sealedsecret
> cluster.postgresql.cnpg.io,statefulset,sealedsecret: command not found
> choimory@choimory-server:/data/storage/media$   kubectl -n choimory-dev-prod get
> You must specify the type of resource to get. Use "kubectl api-resources" for a complete list of supported resources.
>
> error: Required resource not specified.
> Use "kubectl explain <resource>" for a detailed description of that resource (e.g. kubectl explain pods).
> See 'kubectl get -h' for help and examples
> choimory@choimory-server:/data/storage/media$   cluster.postgresql.cnpg.io,statefulset,sealedsecret
> cluster.postgresql.cnpg.io,statefulset,sealedsecret: command not found
> choimory@choimory-server:/data/storage/media$
> ```

## 답변

- 구조 변경이 클러스터에 정상 반영됨. 정상 기준 네 가지를 모두 충족하고, DB와 Redis는 재생성되지 않음
- 작업 기록은 [task7.md](./task7.md) 참고

## 확인 결과

| 정상 기준 | 결과 |
|---|---|
| `*-data` 앱 삭제 | 목록에서 사라짐 |
| `*-postgres`, `*-redis` 앱 | stg, prod 4개 모두 `Synced`, `Healthy` |
| postgres, redis Pod | AGE 4d23h 그대로, RESTARTS 0. 재생성되지 않음 |
| PVC | 4개 모두 `Bound`. 볼륨 이름(`pvc-...`)과 AGE가 그대로라 같은 PVC를 계속 사용 |

- front, member-api는 여전히 `ImagePullBackOff`이며, 이미지 태그가 임시값이라 예상한 상태
- 마지막 두 명령어는 붙여넣을 때 줄이 나뉘어 실행되지 않음. 위 결과로 확인은 충분하며, 보고 싶으면 한 줄로 실행

```bash
kubectl -n choimory-dev-stg get cluster.postgresql.cnpg.io,statefulset,sealedsecret
kubectl -n choimory-dev-prod get cluster.postgresql.cnpg.io,statefulset,sealedsecret
```

## 다음: 앱 레포 deploy.yml 수정 (5번)

- 대상: `choimory-dev-front`, `choimory-dev-member-api`의 `.github/workflows/deploy.yml`

| 항목 | 변경 |
|---|---|
| gitops 경로 | `projects/choimory-dev/{front,member-api}/overlays`를 `projects/choimory-dev/choimory-dev-{front,member-api}/overlays`로 변경. 주석 1줄, 명령 2줄씩 |
| 트리거 | `push`(`dev`, `stg`, `main`)와 `workflow_dispatch` |
| test job | push와 수동 실행 모두에서 돎 |
| deploy job | 수동 실행이면서 브랜치가 `stg`나 `main`일 때만 돎 |
| 브랜치 확인 | 수동 실행에서 다른 브랜치를 고르면 deploy job 첫 단계에서 실패시키고 이유를 출력 |
| 상단 주석 | "자동 배포"를 "수동 실행으로 배포"로 수정 |

- `concurrency`(같은 브랜치 연속 실행 시 이전 실행 취소)는 push 테스트에도 적용됨. 테스트끼리 서로 취소하는 것은 문제없어 그대로 둠
- 작성 후 YAML 문법을 검사하고, 커밋은 하지 않음

---

# Deploy Key 이름

> deploy key 이름 choimory-dev도 붙여서

## 답변

- 키 주석, Deploy key 제목, 키 파일 이름에 `choimory-dev-`를 붙임
- Secret 이름 `GITOPS_DEPLOY_KEY`는 워크플로가 이 이름으로 읽으므로 그대로 둠. 레포마다 따로 저장되는 값이라 이름이 겹쳐도 문제없음
- task7의 키 이름도 같이 변경함

## 키 생성 (Mac)

```bash
ssh-keygen -t ed25519 -C "choimory-dev-member-api-actions" -f ./choimory-dev-member-api-deploy -N ""
ssh-keygen -t ed25519 -C "choimory-dev-front-actions" -f ./choimory-dev-front-deploy -N ""
```

## 등록 위치

| 파일 | 등록 위치 |
|---|---|
| `choimory-dev-member-api-deploy.pub` | `choimory-gitops` → Settings → Deploy keys → Add. 제목 `choimory-dev-member-api-actions`, Allow write access 체크 |
| `choimory-dev-member-api-deploy` | `choimory-dev-member-api` → Settings → Secrets and variables → Actions → New repository secret. 이름 `GITOPS_DEPLOY_KEY` |
| `choimory-dev-front-deploy.pub` | `choimory-gitops` → Deploy keys. 제목 `choimory-dev-front-actions`, Allow write access 체크 |
| `choimory-dev-front-deploy` | `choimory-dev-front` → Actions Secret. 이름 `GITOPS_DEPLOY_KEY` |

## 등록 후 삭제

```bash
rm ./choimory-dev-member-api-deploy ./choimory-dev-member-api-deploy.pub ./choimory-dev-front-deploy ./choimory-dev-front-deploy.pub
```

---

# Environments 만드는 위치

> env 설정에서 stg, prod 환경 만드는게 어디임?

## 답변

- `choimory-dev-front` 레포 → Settings → 왼쪽 메뉴 "Code and automation" 그룹의 Environments
- Secrets and variables 메뉴가 아님

## 만드는 순서

1. New environment 클릭
2. Name에 `stg` 입력 → Configure environment
3. 설정 화면 아래의 Environment variables → Add environment variable
   - Name: `NEXT_PUBLIC_MEMBER_API_BASE_URL`
   - Value: `https://staging-api.choimory.dev/member`
4. 같은 방식으로 `prod` 생성
   - Value: `https://api.choimory.dev/member`

## 주의

- Deployment protection rules(승인자 지정 등)와 Environment secrets는 건드리지 않음
- Settings → Secrets and variables → Actions → Variables 탭에 넣으면 레포 전체 변수가 되어 stg와 prod를 나눌 수 없음. 반드시 Environment 안에 넣음
- Environments 기능은 public 레포에서는 무료, private 레포에서는 유료 플랜이 필요함

---

# Environment secrets와 variables

> profile 생성했고 env secret, env var가 있는데 뭘 해야하지?

## 답변

- Environment variables에만 넣고, Environment secrets는 비워둠

## 등록할 값

| Environment | Name | Value |
|---|---|---|
| `stg` | `NEXT_PUBLIC_MEMBER_API_BASE_URL` | `https://staging-api.choimory.dev/member` |
| `prod` | `NEXT_PUBLIC_MEMBER_API_BASE_URL` | `https://api.choimory.dev/member` |

## variable에 넣는 이유

- 워크플로가 `vars.NEXT_PUBLIC_MEMBER_API_BASE_URL`로 읽음. secret에 넣으면 `vars.`로 읽히지 않아 비어 있다고 판단하고 빌드 전에 실패함
- API 주소는 비밀이 아님. `NEXT_PUBLIC_` 값은 빌드할 때 브라우저용 코드에 그대로 들어가 누구나 볼 수 있음
- Deploy Key 개인키(`GITOPS_DEPLOY_KEY`)는 Environment가 아니라 레포 전체 Secret(Settings → Secrets and variables → Actions)에 넣음
  - member-api 워크플로는 environment를 쓰지 않아 Environment secret을 읽지 못함
  - 두 레포를 같은 방식으로 맞추기 위해 레포 Secret에 넣음

---

# 민감한 값의 위치

> 민감한 값도 env var에 등록함?

## 답변

- 아님. 민감한 값은 secret에 넣고, variable에는 공개되어도 괜찮은 값만 넣음

## variable과 secret의 차이

| 구분 | variable | secret |
|---|---|---|
| 설정 화면 | 값이 그대로 보임 | 등록 후 다시 볼 수 없음 |
| Actions 로그 | 그대로 출력됨 | `***`로 가려짐 |
| 워크플로에서 읽는 방법 | `vars.이름` | `secrets.이름` |
| 넣는 값 | API 주소, 환경 이름처럼 공개되어도 되는 값 | 비밀번호, 토큰, 개인키 |

## 이 프로젝트에서 값이 놓이는 곳

| 값 | 위치 |
|---|---|
| `NEXT_PUBLIC_MEMBER_API_BASE_URL` | front의 Environment variable. 브라우저 코드에 들어가는 공개 값 |
| `GITOPS_DEPLOY_KEY` (Deploy Key 개인키) | 앱 레포 2개의 레포 Secret |
| JWT 서명 키, Redis 비밀번호 | GitHub에 없음. 클러스터의 SealedSecret (6단계) |
| DB 접속 정보 | GitHub에 없음. CloudNativePG가 클러스터 안에서 만든 Secret |

- 앱 실행에 쓰는 비밀값은 GitHub에 두지 않고 클러스터에서 넣는 구조라, 지금 GitHub Environment secret에 넣을 값은 없음

## NEXT_PUBLIC_ 주의

- `NEXT_PUBLIC_`으로 시작하는 값은 secret에 넣어도 빌드 시 브라우저 코드에 들어가 누구나 볼 수 있음
- 그래서 front에는 `NEXT_PUBLIC_` 이름으로 비밀값을 절대 넣지 않음

---

# 모두 secret에 넣는 경우

> 그럼 그냥 다 secret에 넣어도 되는거 아님?

## 답변

- 넣을 수는 있지만, 이 값은 variable로 두는 것을 추천. 얻는 것 없이 불편만 생김

## secret에 넣으면 생기는 일

- 워크플로 수정 필요: 지금은 `vars.`로 읽으므로 `secrets.`로 바꿔야 함. 그대로 두면 빌드 전에 실패함
- 값 확인 불가: 등록 후 다시 볼 수 없어 주소를 잘못 넣어도 확인이 안 되고, 다시 등록해야 함
- 로그 가독성 저하: GitHub는 secret 값과 같은 문자열을 로그에서 모두 `***`로 바꿈. 빌드나 테스트 로그의 API 주소가 가려져 문제 원인을 찾기 어려움
- 보안상 이득 없음: 빌드 시 브라우저 코드에 들어가 누구나 볼 수 있으므로 GitHub에서만 숨겨도 의미 없음

## 기준

- 공개되면 안 되는 값은 secret, 공개되어도 되는 값은 variable
- 헷갈리면 secret에 넣어도 동작에는 문제없지만 위 불편을 감수해야 함
- secret으로 통일하려면 front `deploy.yml`의 `vars.`를 `secrets.`로 바꾸면 됨 (두 줄)

---

# Deploy Key를 Environment secret에 넣는 경우

> 그럼 gitops deploy key도 env의 secret에 넣으면 안됨?

## 답변

- 가능하지만, 지금 구성에서는 얻는 것이 적어 레포 Secret을 추천

## Environment secret에 넣으려면

- member-api 워크플로 수정: Environment secret은 job에 `environment:`를 지정해야 읽힘. front deploy job에는 이미 있지만 member-api에는 없어 한 줄 추가 필요
- 등록 횟수 증가: 같은 키를 `stg`, `prod` 두 Environment에 각각 넣어야 함. 레포 2개 기준 2곳에서 4곳으로 늘어남

## 얻는 것

- 키를 읽을 수 있는 job이 줄어듦. Environment를 지정한 deploy job만 읽고, push 때 도는 test job은 읽지 못함
- 브랜치 제한을 GitHub 설정으로 걸 수 있음. Deployment branches 설정에서 `stg` 환경은 `stg` 브랜치, `prod` 환경은 `main` 브랜치만 쓰도록 강제
- prod 배포 승인 가능. Required reviewers로 prod 배포 전에 승인을 거치게 할 수 있음

## 레포 Secret을 추천하는 이유

- 브랜치 제한은 이미 워크플로의 `Check Branch` 단계가 함
- 배포가 수동 실행이라 Run workflow를 누르는 것 자체가 승인 역할을 함
- 같은 키를 두 환경에 넣으면 환경을 나눈 의미가 없음. stg 키도 gitops 레포 전체(prod overlay 포함)에 쓸 수 있기 때문. 제대로 나누려면 환경별로 키를 따로 만들어야 하는데, 혼자 쓰는 프로젝트에는 과함
- 레포 Secret도 포크된 레포의 PR에는 전달되지 않으며, 레포 쓰기 권한이 있는 본인만 쓸 수 있음

## 정리

| 방식 | 등록 위치 | 워크플로 변경 | 추천 |
|---|---|---|---|
| 레포 Secret (현재 계획) | 레포당 1곳 | 없음 | 지금 추천 |
| Environment secret | 레포당 2곳 (`stg`, `prod`) | member-api에 `environment:` 추가 | 나중에 협업하거나 prod 승인 절차가 필요할 때 |

---

# GitHub secret, variable 레벨 정리

> 헷갈려서 정리하자면 repo 레벨의 secret, var 등록이 있고, env 레벨의 secret, var 등록이 있는데 ssh key는 repo secret, 프로젝트별 환경변수값은 env 레벨의 var나 secret에 저장한다는거지?

## 답변

- 맞음. 다만 앱이 실행될 때 쓰는 비밀값은 GitHub 어디에도 넣지 않음

## 기준

| | secret (비밀값) | variable (공개값) |
|---|---|---|
| repo 레벨 (모든 환경 공통) | `GITOPS_DEPLOY_KEY` (SSH 개인키) | 지금은 없음 |
| env 레벨 (stg, prod별로 다른 값) | 지금은 없음 | `NEXT_PUBLIC_MEMBER_API_BASE_URL` |

- repo와 env 구분: 값이 stg와 prod에서 같으면 repo, 다르면 env. SSH 키는 두 환경에서 같은 키를 쓰므로 repo
- secret과 variable 구분: 노출되면 안 되면 secret, 공개되어도 되면 variable

## GitHub에 넣는 값은 빌드할 때 쓰는 값만

- GitHub Secrets와 Variables는 Actions가 빌드하고 배포할 때 쓰는 값
- 앱이 서버에서 실행될 때 쓰는 비밀값은 클러스터에 있음

| 값 | 쓰는 시점 | 위치 |
|---|---|---|
| `GITOPS_DEPLOY_KEY` | 배포 (gitops 커밋) | GitHub repo secret |
| `NEXT_PUBLIC_MEMBER_API_BASE_URL` | 빌드 (front 번들에 포함) | GitHub env variable |
| JWT 서명 키, Redis 비밀번호 | 실행 | 클러스터 SealedSecret |
| DB 접속 정보 | 실행 | 클러스터 Secret (CloudNativePG가 생성) |
| `SPRING_PROFILES_ACTIVE` 같은 일반 설정 | 실행 | gitops overlay 매니페스트 |

- 프로젝트별 환경변수값이 빌드 때 필요한 값이면 env 레벨에, 실행 때 필요한 값이면 gitops 쪽에 둠

---

# SSH 키 사용처

> ssh키는 근데 어디에 사용되는거야?

## 답변

- 앱 레포의 Actions가 gitops 레포에 이미지 태그를 커밋하고 push할 때만 사용됨

## 사용되는 곳

- deploy job의 마지막 두 단계
  1. Checkout GitOps: `actions/checkout`이 `ssh-key: ${{ secrets.GITOPS_DEPLOY_KEY }}`로 `choimory-gitops` 레포를 내려받음
  2. Update GitOps Image Tag: overlay의 이미지 태그를 바꾸고 `git commit` 후 `git push origin HEAD:main`. 1번에서 설정된 같은 키로 인증함

## 전체 흐름에서의 위치

```
Run workflow (수동)
  → test
  → 이미지 빌드
  → GHCR에 push                  ← GITHUB_TOKEN 사용
  → gitops 레포 clone, 태그 수정
  → gitops 레포에 push            ← SSH 키 사용 (여기만)
  → ArgoCD가 gitops main 변경 감지 → 클러스터에 배포
```

## 왜 필요한가

- Actions 기본 `GITHUB_TOKEN`은 워크플로가 도는 레포에만 쓸 수 있음
- 그래서 `choimory-dev-front`의 Actions는 다른 레포인 `choimory-gitops`에 push할 수 없음
- gitops 레포에 쓰기 권한이 있는 Deploy Key를 등록하고, 개인키를 앱 레포 Secret에 넣어 Actions가 쓰게 함

## 사용되지 않는 곳

| 대상 | 인증 방식 |
|---|---|
| GHCR 이미지 push | `GITHUB_TOKEN` (`packages: write` 권한) |
| ArgoCD가 gitops 레포 읽기 | 인증 없음 (public 레포를 https로 읽음) |
| 클러스터가 이미지 받기 | 인증 없음 (GHCR 패키지를 public으로 전환 예정) |
| 홈서버 SSH 접속 | 관계없음. 이 키는 서버에 저장하지 않음 |

---

# Deploy Key의 인증 대상

> github에서 나를 인식하기 위한 보안 용도로 사용하는건가?

## 답변

- 아님. Deploy Key는 사람 계정이 아니라 `choimory-gitops` 레포 하나에 붙은 출입 열쇠

## 계정 SSH 키와 비교

| 구분 | 계정 SSH 키 | Deploy Key (이번 키) |
|---|---|---|
| 등록 위치 | 개인 Settings → SSH and GPG keys | `choimory-gitops` → Settings → Deploy keys |
| 인증하는 대상 | 사용자 본인 (choimory 계정) | 이 개인키를 가진 쪽 |
| 접근 범위 | 본인이 권한 있는 모든 레포 | `choimory-gitops` 한 곳만 |
| 용도 | Mac에서 본인이 git push할 때 | Actions가 gitops에 push할 때 |

## GitHub가 이 키로 판단하는 것

- "이 사용자가 누구인가"가 아니라 "이 레포에 쓰기를 허락받은 키인가"만 확인함
- 키를 가진 Actions 러너는 `choimory-gitops`에 push할 수 있지만, 다른 레포에는 아무것도 할 수 없음

## 커밋 작성자

- 커밋 작성자도 본인이 아님
- 워크플로에서 `git config user.name "github-actions[bot]"`으로 설정하므로 gitops 커밋 기록에 `github-actions[bot]`으로 남음

## 이렇게 하는 이유

- 개인 계정 키나 토큰을 Actions에 넣으면, 유출 시 본인의 모든 레포가 위험해짐
- Deploy Key는 유출돼도 피해가 `choimory-gitops` 한 곳으로 한정됨
- 유출 시 GitHub에서 해당 Deploy Key를 지우고 새로 만들면 됨

---

# API 도메인을 서비스별 서브도메인으로 변경

> api 주소말인데
>
> member-api: member-api.choimory.dev
> board-api: board-api.choimory.dev 이런식으로 할거임
>
> stg면 staging-member-api.choimory.dev 이런식으로 할거고

## 답변

- 서비스별 서브도메인 방식으로 변경 가능. 고칠 곳은 gitops 3개 파일과 front 주석 1줄
- plan2에서 정한 경로 방식(`api.choimory.dev/member/...`)을 변경하는 결정

## 바뀌는 도메인

| 서비스 | stg | prod |
|---|---|---|
| member-api (기존) | `staging-api.choimory.dev/member` | `api.choimory.dev/member` |
| member-api (변경) | `staging-member-api.choimory.dev` | `member-api.choimory.dev` |
| board-api (추후) | `staging-board-api.choimory.dev` | `board-api.choimory.dev` |

## 문제없는 것

- SSL: 모두 한 단계 서브도메인이라 Cloudflare Universal SSL(`*.choimory.dev`)로 처리됨. `staging.member-api...`가 아니라 하이픈으로 잇는 것이 맞음
- Gateway: `shared-gateway`는 hostname 제한 없이 HTTPRoute에서 도메인을 정하므로 수정 없음
- CORS: member-api 허용 Origin은 front 도메인(`staging.choimory.dev`, `choimory.dev`)이라 변경 없음
- 쿠키: `member-api.choimory.dev`와 `choimory.dev`는 같은 사이트라 `SameSite=Lax` 그대로 동작
- front 서버 쪽 호출: 클러스터 내부 주소(`http://choimory-dev-member-api:8080`)를 쓰므로 영향 없음

## 고칠 것

| 위치 | 변경 |
|---|---|
| gitops `choimory-dev-member-api/base/httproute.yaml` | `/member` 경로 매칭과 접두어를 떼는 URLRewrite 삭제. 도메인 전체를 member-api로 전달 (앱 경로가 이미 `/v1/member/...`) |
| gitops `overlays/stg/kustomization.yaml` | hostname을 `staging-member-api.choimory.dev`로 변경 |
| gitops `overlays/prod/kustomization.yaml` | hostname을 `member-api.choimory.dev`로 변경 |
| front `deploy.yml` 97줄 | 주석의 주소 예시 수정 |

## front Environment variable 값 변경

- `NEXT_PUBLIC_MEMBER_API_BASE_URL` 값에서 끝의 `/member`가 빠짐

| Environment | 값 |
|---|---|
| `stg` | `https://staging-member-api.choimory.dev` |
| `prod` | `https://member-api.choimory.dev` |

## 비용

- 경로 방식은 API 추가 시 HTTPRoute 규칙만 추가하면 됐음
- 서브도메인 방식은 API마다 도메인이 생겨 Cloudflare DNS와 Tunnel 설정도 늘어남
- 2차 Tunnel 작업 때 `*.choimory.dev` 와일드카드 하나로 연결하면 API를 추가해도 Cloudflare 설정 변경이 필요 없음. 그때 결정

## 진행 방식

- gitops 수정은 앱 레포 머지 전에 함께 진행 추천. 이미지가 없어 member-api가 떠 있지 않으므로 영향 없음
- 수정 후 overlay를 빌드해 검증하고, 커밋은 직접 진행

---

# 나중에 API 게이트웨이로 전환

> 이렇게 쓰다가, 나중에 api gateway 추가하면 api.choimory.dev/member, board 이런식으로 바꿀거임

## 답변

- 지금 서브도메인으로 시작해도 나중에 API 게이트웨이로 옮길 때 막히는 것은 없음

## 전환할 때 바뀌는 것

| 대상 | 변경 |
|---|---|
| front Environment variable | `https://member-api.choimory.dev` → `https://api.choimory.dev/member` |
| front 이미지 | 값이 빌드 시 번들에 고정되므로 stg, prod 모두 다시 빌드 |
| gitops HTTPRoute | `api.choimory.dev`는 API 게이트웨이로 연결. member-api는 게이트웨이 뒤의 내부 서비스 |
| Cloudflare | 와일드카드로 연결해 두면 변경 없음 |
| member-api, CORS, 쿠키 | 변경 없음 |

## 미리 알아둘 점

- 경로 겹침: member-api 경로가 이미 `/v1/member/...`라 게이트웨이에서 `/member`를 그대로 넘기면 `api.choimory.dev/member/v1/member/join`이 됨. 게이트웨이가 접두어를 떼고 전달하면 됨
- 전환 기간에는 `member-api.choimory.dev`도 남겨두고, front를 새 주소로 다시 배포한 뒤 정리하면 끊기지 않음
- 지금 지우는 `/member` 경로 매칭 설정은 git 기록에 남아 나중에 참고 가능

---

# 경로 방식에서 서비스 이름이 겹치는 문제

> 내가 왜 바꾸려하냐면 member-api를 예로들면
>
> GET api.choimory.dev/member/v1/member/1
> GET api.choimory.dev/member/v1/admin/member/1
>
> 이런식으로 member가 두번 들어가거나 해야하지 않아?

## 답변

- 맞음. 지금 계획의 경로 방식이면 실제로 그렇게 됨

## 겹치는 이유

- member-api 안의 경로가 이미 `/v1/member/...`, `/v1/admin/member/...`처럼 리소스 이름을 포함함
- 게이트웨이가 서비스를 구분하려고 `/member`를 앞에 붙이면, 서비스 이름과 리소스 이름이 같아 두 번 나옴

```
api.choimory.dev/member/v1/member/1
                 ^^^^^^    ^^^^^^
                 서비스 구분  앱의 리소스 경로
```

## 서브도메인 방식

```
member-api.choimory.dev/v1/member/1
member-api.choimory.dev/v1/admin/member/1
board-api.choimory.dev/v1/board/1
```

- 도메인이 서비스를 구분하고, 경로는 앱 경로 그대로라 겹치지 않음

## 게이트웨이에서 겹침을 피하는 방법

- `api.choimory.dev` 하나로 합칠 때 서비스 접두어를 붙이지 않고 앱 경로 자체로 라우팅

| 요청 경로 | 전달 대상 |
|---|---|
| `api.choimory.dev/v1/member/**` | member-api |
| `api.choimory.dev/v1/admin/member/**` | member-api |
| `api.choimory.dev/v1/agreement/**` | member-api |
| `api.choimory.dev/v1/board/**` | board-api |

- 장점: URL이 `api.choimory.dev/v1/member/1`로 깔끔하고 앱 코드 변경 없음
- 지킬 것
  - 게이트웨이가 서비스별 경로를 알아야 함. 예: member-api는 `member`, `admin/member`, `agreement` 세 규칙
  - 서비스끼리 경로가 겹치면 안 됨. 예: 두 서비스가 모두 `/v1/admin/...` 아래에 다른 리소스를 두면 규칙을 세분화해야 함
- 나중에 게이트웨이를 붙일 때 이 방식이면 겹침 없이 합칠 수 있음. 그때 결정

---

# 다음 작업: API 도메인 변경과 7단계 진행

> secret, var 모두 확인 됐어

## 답변

- 6번(Deploy Key, front Environments 변수 등록)이 끝나 남은 작업 순서를 정리함
- 등록 결과는 [task7.md Deploy Key와 Environments 변수 등록 완료](./task7.md#deploy-key와-environments-변수-등록-완료) 참고

## 1. API 도메인 변경 (Claude)

| 위치 | 변경 |
|---|---|
| gitops `choimory-dev-member-api/base/httproute.yaml` | `/member` 경로 매칭과 접두어를 떼는 URLRewrite 삭제. 도메인 전체를 member-api 8080으로 전달. 상단 주석 수정 |
| gitops `choimory-dev-member-api/overlays/stg/kustomization.yaml` | hostname `staging-api.choimory.dev` → `staging-member-api.choimory.dev` |
| gitops `choimory-dev-member-api/overlays/prod/kustomization.yaml` | hostname `api.choimory.dev` → `member-api.choimory.dev` |
| front `.github/workflows/deploy.yml` 97줄 | 주석의 주소 예시를 새 도메인으로 수정 |

- 검증: `kubectl kustomize`로 stg, prod overlay를 빌드해 HTTPRoute의 hostname과 규칙 확인
- member-api가 아직 떠 있지 않아 반영해도 영향 없음

## 2. gitops 커밋과 main 반영 (직접)

- 반영 후 ArgoCD가 HTTPRoute만 갱신함. 확인 명령어는 반영 후 안내

## 3. 앱 레포 커밋과 머지 (직접)

- 커밋할 것
  - front: `deploy.yml` 수정 (수동 실행, gitops 경로, 주석). `dev`에 push 안 된 커밋 1개가 함께 올라감
  - member-api: `deploy.yml` 수정 (수동 실행, gitops 경로). `dev`에 push 안 된 커밋 3개가 함께 올라감
- `stg`와 `main`에 머지
  - `deploy.yml`이 기본 브랜치(`main`)에 있어야 Run workflow 버튼이 보임
  - 수동 실행이라 `main`에 머지해도 prod 배포는 시작되지 않음
- push하면 test job만 자동으로 돎. 여기서 테스트가 통과하는지 먼저 확인

## 4. stg 배포 실행 (직접)

- 두 레포 Actions 탭 → deploy → Run workflow → 브랜치 `stg` 선택
- 확인할 것
  - test, deploy job 성공
  - GHCR에 `stg-<sha>` 태그 이미지 생성
  - gitops `main`에 `github-actions[bot]`의 `deploy: ...` 커밋 생성
- front와 member-api를 동시에 실행해도 gitops push 충돌은 워크플로가 rebase 후 재시도함

## 5. GHCR 패키지 public 전환 (직접)

- 첫 빌드 후 `choimory-dev-member-api`, `choimory-dev-front` 패키지를 public으로 변경
- 매니페스트에 `imagePullSecrets`가 없어 비공개면 계속 `ImagePullBackOff`

## 6. stg 1차 확인

- 명령어는 Claude가 안내하고 서버에서 직접 실행
- 확인 항목
  - Pod `Running`, 재시작 횟수
  - member-api actuator(8081) liveness, readiness
  - `port-forward`와 `curl`로 front, member-api 헬스체크
  - 회원가입, 로그인, 토큰 재발급. Redis에 토큰 저장 여부
  - ArgoCD가 Actions의 태그 커밋을 받아 자동 배포하는지

## 7. prod 배포와 확인

- Actions에서 브랜치 `main`으로 Run workflow
- stg와 같은 방식으로 확인
