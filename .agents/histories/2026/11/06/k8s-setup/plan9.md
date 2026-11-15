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
