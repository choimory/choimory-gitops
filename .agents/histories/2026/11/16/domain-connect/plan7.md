# 목차

- [개요](#개요)
  - [설정](#설정)
- [설정 분석](#설정-분석)
  - [답변](#답변)
  - [요구사항 정리](#요구사항-정리)
  - [확인한 사실](#확인한-사실)
  - [설계안](#설계안)
    - [PostgreSQL](#postgresql)
    - [Redis](#redis)
  - [진행 순서 (안)](#진행-순서-안)
  - [다운타임](#다운타임)
  - [결정 필요 사항](#결정-필요-사항)
- [작업사항 확인](#작업사항-확인)
  - [단계별 상태](#단계별-상태)
- [남은 작업](#남은-작업)
- [5단계 접속 확인](#5단계-접속-확인)
- [DB 접속정보 정리](#db-접속정보-정리)
  - [현재 접속 정보 (stg, prod 공통)](#현재-접속-정보-stg-prod-공통)
  - [비밀번호 확인 명령 (서버)](#비밀번호-확인-명령-서버)
  - [로그인 확인 (Mac)](#로그인-확인-mac)
- [비밀번호 확인 결과](#비밀번호-확인-결과)
- [Secret 값이 원문인 이유](#secret-값이-원문인-이유)
- [비밀번호 생성 방식](#비밀번호-생성-방식)
- [계정 재생성 결정](#계정-재생성-결정)
  - [다음 진행](#다음-진행)
- [계정 재생성 계획과 SealedSecret 과정](#계정-재생성-계획과-sealedsecret-과정)
  - [SealedSecret 흐름](#sealedsecret-흐름)
  - [핵심 개념](#핵심-개념)
  - [진행 제안](#진행-제안)
  - [확인 필요](#확인-필요)
- [일반 Secret과 SealedSecret 차이](#일반-secret과-sealedsecret-차이)
  - [1. 일반 Secret 방식](#1-일반-secret-방식)
  - [2. SealedSecret의 차이](#2-sealedsecret의-차이)
  - [3. deployment.yaml 작성](#3-deploymentyaml-작성)
- [SealedSecret 이해 확인](#sealedsecret-이해-확인)
  - [명령은 한 줄로 이어서 실행](#명령은-한-줄로-이어서-실행)
  - [암호화는 kubeseal이 함](#암호화는-kubeseal이-함)
  - [정리](#정리)
- [SealedSecret 요약](#sealedsecret-요약)
  - [주의할 점](#주의할-점)
- [deployment와 SealedSecret 연결](#deployment와-sealedsecret-연결)
  - [연결 구조](#연결-구조)
  - [맞춰야 하는 것](#맞춰야-하는-것)
- [choimory-dev-postgres-app 위치](#choimory-dev-postgres-app-위치)
  - [member-api가 참조하는 Secret](#member-api가-참조하는-secret)
  - [CNPG가 만드는 방식](#cnpg가-만드는-방식)
- [PostgreSQL만 예외](#postgresql만-예외)
  - [plan7 이후](#plan7-이후)
- [계정 재생성 진행 계획](#계정-재생성-진행-계획)
  - [전체 순서](#전체-순서)
  - [4번에서 바뀌는 파일](#4번에서-바뀌는-파일)
  - [5번 주의](#5번-주의)
  - [1번에서 정할 값 (stg, prod 각각)](#1번에서-정할-값-stg-prod-각각)
- [계정 재생성 최종 계획](#계정-재생성-최종-계획)
  - [계획 밖에서 챙길 것](#계획-밖에서-챙길-것)
  - [다음](#다음)
- [추가1](#추가1)
- [앱 계정과 내 계정](#앱-계정과-내-계정)
  - [나누는 이유](#나누는-이유)
- [추가1 분석 (첫 번째)](#추가1-분석-첫-번째)
- [추가1 분석 (두 번째)](#추가1-분석-두-번째)
- [추가1 분석 (세 번째)](#추가1-분석-세-번째)
  - [Secret 구성 (안)](#secret-구성-안)
- [추가1 결정과 앱 저장소 확인](#추가1-결정과-앱-저장소-확인)
  - [앱 저장소를 봐야 하는 이유](#앱-저장소를-봐야-하는-이유)
  - [선택지](#선택지)
- [member-api 저장소 위치](#member-api-저장소-위치)
- [member-api Redis, DB 설정 확인](#member-api-redis-db-설정-확인)
  - [확인 결과](#확인-결과)
  - [member-api deployment.yaml 변경 (안)](#member-api-deploymentyaml-변경-안)
  - [다음](#다음-1)

---

# 개요

- postgres, redis 설정과 계정 설정하기
- 이를 바탕으로 db 새로 만들고 sealed secret도 새로 작성하기
- 앱 재배포해야하면 재배포하기

## 설정

- redis, postgres 모두 별도 계정 추가
- redis가 현재 no username 설정인거 같은데 stg, prod 모두 username required 였으면 좋겠음
- 계정명과 비밀번호는 내가 직접 적고싶은데 명령어 알려주면 내가 거기 바꿔서 새로 만들면 될듯

---

# 설정 분석

> - redis, postgres 모두 별도 계정 추가
> - redis가 현재 no username 설정인거 같은데 stg, prod 모두 username required 였으면 좋겠음
> - 계정명과 비밀번호는 내가 직접 적고싶은데 명령어 알려주면 내가 거기 바꿔서 새로 만들면 될듯

## 답변

- 영향 범위 확인을 위해 member-api의 DB, Redis 접속 방식(`choimory-dev-member-api/base/deployment.yaml`)과 CNPG `1.30` 문서를 함께 확인

## 요구사항 정리

| 요구 | 해석 |
|---|---|
| postgres, redis 계정 설정 | 앱용 계정과 내 계정(별도 계정)을 내가 정한 이름과 비밀번호로 |
| DB 새로 만들기 | PostgreSQL `Cluster`를 내가 정한 DB 이름, 앱 계정으로 다시 만들기 |
| SealedSecret 새로 작성 | 계정, 비밀번호를 서버에서 직접 입력해 암호화 |
| Redis username 필수 | stg, prod 모두 이름 없는 기본 사용자(`default`)를 끄고 이름 있는 계정으로만 접속 |
| 앱 재배포 | member-api 접속 설정이 바뀌므로 필요 |

## 확인한 사실

1. member-api는 PostgreSQL 접속 정보를 CNPG가 만든 `choimory-dev-postgres-app` Secret에서 읽음 (키 `host`, `port`, `dbname`, `user`, `password`)
   - CNPG 문서상 앱 계정 Secret을 직접 제공하면(`initdb.secret`) `-app` Secret은 생성되지 않음
   - member-api env를 새 Secret 기준으로 변경 필요. gitops 수정만으로 되고 이미지 재빌드 불필요
2. 직접 제공하는 Secret은 `basic-auth` 형식(`username`, `password`). `username`은 `owner`와 같아야 함
3. 내 계정(별도 계정)은 `managed.roles`로 추가, `passwordSecret`에 SealedSecret 지정
4. Redis username은 앱 코드 확인 필요
   - member-api env에는 `redis-host`, `redis-port`, `redis-password`만 있음
   - 앱 설정(`spring.data.redis.username`)이 env를 받지 않으면 member-api 저장소 수정과 이미지 재배포(Actions deploy) 필요
   - member-api 저장소는 이 작업 공간에 없어 미확인

## 설계안

### PostgreSQL

| 항목 | 내용 |
|---|---|
| `Cluster` 이름 | `choimory-dev-postgres` 유지 (`-rw`, `-tailnet` Service 이름이 그대로라 다른 곳 영향 없음) |
| DB 이름, 앱 계정 | 직접 정함. `bootstrap.initdb`의 `database`, `owner`, `secret` |
| 앱 계정 비밀번호 | SealedSecret (basic-auth) |
| 내 계정 | `managed.roles`로 추가, SealedSecret (basic-auth) |
| 내 계정 권한 | 예: stg 읽기+쓰기, prod 읽기만(`pg_read_all_data`). DB 권한으로 Read-only 강제 |
| member-api env | host `choimory-dev-postgres-rw`, port `5432`, dbname은 정한 값, user와 password는 새 Secret의 `username`, `password` |

### Redis

| 항목 | 내용 |
|---|---|
| 방식 | Redis ACL. `default` 사용자 끄고 앱 계정과 내 계정을 이름, 비밀번호로 생성 |
| 설정 전달 | ACL 파일(`users.acl`)을 SealedSecret에 넣고 Redis가 `--aclfile`로 읽음 (`requirepass` 제거) |
| Secret | `choimory-dev-redis-secret` 새로 작성. 키 `username`, `password`(member-api용), `users.acl`(Redis용) |
| 내 계정 권한 | 예: stg 전체, prod 읽기 위주 |
| member-api | env에 `redis-username` 추가 + 앱이 사용하도록 설정 (위 확인 필요) |

## 진행 순서 (안)

1. member-api 저장소에서 Redis username 지원 여부 확인. 없으면 앱 수정을 먼저 또는 함께 배포
2. 서버에서 SealedSecret 생성 (환경별). 이름은 화면에 보이게, 비밀번호는 `read -rs`로 숨겨 입력. 명령은 그대로 복사해 쓰도록 제공
3. gitops 작성 (Claude): `postgres.yaml`(initdb, managed.roles), overlay SealedSecret, redis statefulset(ACL), member-api env, README
4. push 후 DB 재생성: `initdb`는 처음 만들 때만 적용되므로 push 후 기존 `Cluster`를 직접 삭제하면 ArgoCD가 새 설정으로 다시 생성
   - 기존 데이터는 사라짐 (현재는 테스트 계정 정도)
   - 스키마는 member-api 재시작 시 Flyway가 다시 생성
5. Redis 재시작: ACL 적용. stg는 원래 데이터 미보존, prod는 AOF로 유지되지만 기존 토큰은 사실상 무효(재로그인)
6. member-api 재배포, 회원가입·로그인 확인, DataGrip 확인

## 다운타임

- 4~6 동안 stg, prod 서비스(로그인 등)가 몇 분 끊김
- stg 먼저 진행해 확인 후 prod 추천

## 결정 필요 사항

1. 기존 DB 데이터: 버리고 새로 시작(추천, 테스트 계정 정도) / 덤프와 복원으로 이전
2. 내 계정 권한: stg 읽기+쓰기, prod 읽기만. Redis도 같은 기준
3. member-api 저장소에서 Redis username 설정(`spring.data.redis.username`) 지원 여부 확인. 안 되면 앱 수정도 이번 범위
4. 진행 순서: stg 먼저, prod 나중
5. Secret 이름: `choimory-dev-postgres-app-auth`(앱 계정), `choimory-dev-postgres-<내계정>-auth`(내 계정), `choimory-dev-redis-secret`(Redis, 키 확장)

---

# 작업사항 확인

> 261116 작업사항 확인

## 답변

- `domain-connect`는 calendar 상태 TASK. 0~5단계 gitops 반영 완료, plan7(계정 재설정) 분석 중, 6단계(마무리) 남음

## 단계별 상태

| 단계 | 내용 | 문서 | 상태 |
|---|---|---|---|
| 0 | Gateway 이름 변경 (`shared-gateway` → `gateway`) | plan1, task1 | 완료 |
| 1 | `choimory.dev` 구매, Cloudflare DNS 연결 | plan2, task2 | 완료 |
| 2 | Cloudflare Tunnel, cloudflared (프록시 이름 `gateway-proxy` 고정, HTTP/2(TCP 443)) | plan3, task3 | 완료 |
| 3 | 브라우저 확인 (CORS, 쿠키) | plan4 | 완료 |
| 4 | ArgoCD 내부 접속 (E2: `infra-gateway`, `*.infra.choimory.dev`, Let's Encrypt, Tailscale Operator) | plan5, task4 | 완료 |
| 5 | PostgreSQL, Redis Tailscale 접속 | plan6, task5 | 커밋 완료 (`c918c88`, `7c23dd9`), push 후 확인 기록 없음 |
| 추가 | PostgreSQL, Redis 계정 재설정 | plan7 | 분석 중 |
| 6 | 마무리 (done, calendar DONE, README) | | 미진행 |

- `.agents/histories/2026/11/16/` 디렉토리 전체가 git untracked 상태

---

# 남은 작업

> 해야할게 뭐지?

## 답변

1. 5단계 push 후 확인 (Service, `ts-` Pod, Mac에서 포트 연결)
2. plan7 결정 사항 5가지 (기존 데이터, 내 계정 권한, member-api Redis username 지원 여부, 진행 순서, Secret 이름)
3. plan7 구현 (SealedSecret 생성, gitops, DB 재생성, Redis 재시작, member-api 재배포, 확인)
4. 마무리 (done1.md, calendar DONE, README, 히스토리 커밋)

- plan7은 member-api의 Redis username 지원 여부가 범위를 결정하므로 먼저 확인 필요

---

# 5단계 접속 확인

> (서버 확인 결과)
>
> ```
> choimory@choimory-server:~$   kube -n choimory-dev-stg get svc choimory-dev-postgres-tailnet choimory-dev-redis-tailnet
>   kube -n choimory-dev-prod get svc choimory-dev-postgres-tailnet choimory-dev-redis-tailnet
>   kube -n choimory-infra get pods | grep ts-
> NAME                            TYPE           CLUSTER-IP      EXTERNAL-IP                                                PORT(S)          AGE
> choimory-dev-postgres-tailnet   LoadBalancer   10.104.27.209   100.x.x.x,choimory-dev-postgres-stg.<TAILNET>.ts.net   5432:31601/TCP   18h
> choimory-dev-redis-tailnet      LoadBalancer   10.100.174.29   100.x.x.x,choimory-dev-redis-stg.<TAILNET>.ts.net    6379:30445/TCP   18h
> NAME                            TYPE           CLUSTER-IP       EXTERNAL-IP                                                  PORT(S)          AGE
> choimory-dev-postgres-tailnet   LoadBalancer   10.104.220.167   100.x.x.x,choimory-dev-postgres-prod.<TAILNET>.ts.net   5432:31186/TCP   18h
> choimory-dev-redis-tailnet      LoadBalancer   10.105.19.225    100.x.x.x,choimory-dev-redis-prod.<TAILNET>.ts.net     6379:31531/TCP   18h
> sealed-secrets-controller-5b56d98858-wbm4r          1/1     Running   0          2d17h
> ts-choimory-dev-postgres-tailnet-pkzhr-0            1/1     Running   0          18h
> ts-choimory-dev-postgres-tailnet-z288q-0            1/1     Running   0          18h
> ts-choimory-dev-redis-tailnet-g7gqq-0               1/1     Running   0          18h
> ts-choimory-dev-redis-tailnet-twqvf-0               1/1     Running   0          18h
> ts-infra-gateway-proxy-9nwld-0                      1/1     Running   0          24h
> ```
>
> (Mac 확인 결과)
>
> ```
> ╰─$   nc -vz choimory-dev-postgres-stg 5432
>   nc -vz choimory-dev-redis-stg 6379
> Connection to choimory-dev-postgres-stg port 5432 [tcp/postgresql] succeeded!
> Connection to choimory-dev-redis-stg port 6379 [tcp/*] succeeded!
> ╰─$   nc -vz choimory-dev-postgres-prod 5432
>   nc -vz choimory-dev-redis-prod 6379
> Connection to choimory-dev-postgres-prod port 5432 [tcp/postgresql] succeeded!
> Connection to choimory-dev-redis-prod port 6379 [tcp/*] succeeded!
> ```

## 답변

- 5단계 접속 확인 완료

| 항목 | 결과 |
|---|---|
| Service 4개 | stg, prod 모두 EXTERNAL-IP `100.x` + `*.<TAILNET>.ts.net` 할당 |
| `ts-` 프록시 Pod | postgres 2, redis 2, `infra-gateway` 1 = 5개 Running |
| Mac 포트 연결 | stg, prod의 PostgreSQL 5432, Redis 6379 모두 성공 |

- 로그인(DataGrip, redis-cli) 확인은 계정 정리 후 진행

---

# DB 접속정보 정리

> 내 생각엔 db 접속정보만 정리하면 될거같은데

## 답변

- 이 시점의 제안: 계정 재생성, Redis ACL, member-api 재배포 없이 기존 계정 접속 정보만 정리 (이후 [계정 재생성 결정](#계정-재생성-결정)에서 계정을 다시 만들기로 변경)

## 현재 접속 정보 (stg, prod 공통)

| 대상 | Host | Port | DB | 사용자 | 비밀번호 위치 |
|---|---|---|---|---|---|
| PostgreSQL | `choimory-dev-postgres-{stg,prod}` | 5432 | `member` | `member` | Secret `choimory-dev-postgres-app`의 `password` (CNPG 자동 생성) |
| Redis | `choimory-dev-redis-{stg,prod}` | 6379 | - | 없음 (`default`) | SealedSecret `choimory-dev-redis-secret`의 `password` |

- PostgreSQL은 SSL `require`. DataGrip에서 SSL 켜고 인증서 검증 없음

## 비밀번호 확인 명령 (서버)

```bash
# PostgreSQL
kube -n choimory-dev-stg  get secret choimory-dev-postgres-app -o jsonpath='{.data.password}' | base64 -d; echo
kube -n choimory-dev-prod get secret choimory-dev-postgres-app -o jsonpath='{.data.password}' | base64 -d; echo
# Redis
kube -n choimory-dev-stg  get secret choimory-dev-redis-secret -o jsonpath='{.data.password}' | base64 -d; echo
kube -n choimory-dev-prod get secret choimory-dev-redis-secret -o jsonpath='{.data.password}' | base64 -d; echo
```

## 로그인 확인 (Mac)

- DataGrip: Host `choimory-dev-postgres-{stg,prod}`, Port `5432`, DB·User `member`, SSL `require`. prod는 Read-only
- Redis: `redis-cli -h choimory-dev-redis-{stg,prod} --askpass ping` → `PONG`

---

# 비밀번호 확인 결과

> (서버 확인 결과, 비밀번호 마스킹)
>
> ```
> choimory@choimory-server:~$   # PostgreSQL
>   kube -n choimory-dev-stg  get secret choimory-dev-postgres-app -o jsonpath='{.data.password}' | base64 -d; echo
>   kube -n choimory-dev-prod get secret choimory-dev-postgres-app -o jsonpath='{.data.password}' | base64 -d; echo
>   # Redis
>   kube -n choimory-dev-stg  get secret choimory-dev-redis-secret -o jsonpath='{.data.password}' | base64 -d; echo
>   kube -n choimory-dev-prod get secret choimory-dev-redis-secret -o jsonpath='{.data.password}' | base64 -d; echo
> ****(64자, stg PostgreSQL)
> ****(64자, prod PostgreSQL)
> ****(64자, stg Redis)
> ****(64자, prod Redis)
> ```

## 답변

- 4개 모두 정상 출력 (순서: stg PG, prod PG, stg Redis, prod Redis)
- 비밀번호 원문이 채팅에 노출됨. tailnet 전용이라 즉시 위험은 낮지만 prod는 교체 권장
- 이후 출력을 붙여 넣을 때는 비밀번호 부분을 지울 것

---

# Secret 값이 원문인 이유

> 저 암호가 암호화내용이 아니라 원문이야?

## 답변

- 원문. DB, Redis에 실제로 쓰이는 비밀번호 그대로

| 위치 | 상태 |
|---|---|
| git의 SealedSecret | 암호화. 클러스터의 sealed-secrets-controller만 복호화 가능 |
| 클러스터 안 Secret | controller가 복호화해 만든 일반 Secret. `data`는 base64 인코딩일 뿐 암호화 아님 |
| `base64 -d` 결과 | 원문 |

- PostgreSQL `-app` Secret은 SealedSecret을 거치지 않고 CNPG가 클러스터 안에서 직접 만든 일반 Secret
- SealedSecret은 git에 안전하게 올리기 위한 것이고, 클러스터 안에서는 RBAC(`get secret` 권한)로 보호

---

# 비밀번호 생성 방식

> 그럼 암호는 임의대로 적었었나 보네?

## 답변

- 사람이 정한 적 없이 둘 다 자동 생성된 랜덤 값

| 대상 | 생성 방식 | 출처 |
|---|---|---|
| Redis | `openssl rand -hex 32`(64자 hex)로 생성해 바로 kubeseal로 암호화. 원래 값은 화면, 파일에 남지 않음 | `.agents/histories/2026/11/06/k8s-setup/task5.md` (Secret 생성 명령) |
| PostgreSQL | `bootstrap.initdb`에 `database`, `owner`만 지정해 CNPG가 랜덤 비밀번호를 생성하고 `choimory-dev-postgres-app`에 저장 | `projects/choimory-dev/postgres/base/postgres.yaml` |

- member-api `jwt-secret`도 Redis와 같은 방식으로 생성

---

# 계정 재생성 결정

> db 계정명이랑 비밀번호 내가 정해서 다시만들거임. 그러니 db 암호 노출상관없음.

## 답변

- 계정명과 비밀번호를 직접 정해 다시 만들기로 결정. [DB 접속정보 정리](#db-접속정보-정리)의 "기존 계정 유지" 제안은 취소하고 [설계안](#설계안)대로 진행
- 기존 비밀번호는 모두 교체되므로 채팅 노출은 무관

## 다음 진행

1. [결정 필요 사항](#결정-필요-사항) 5가지 확정
   - 특히 member-api의 Redis username 지원 여부(`spring.data.redis.username`) 확인. member-api 저장소 확인 필요
2. 서버에서 SealedSecret 생성 (계정명은 직접 입력, 비밀번호는 `read -rs`로 숨겨 입력)
3. gitops 작성, DB 재생성, Redis 재시작, member-api 재배포
4. DataGrip, redis-cli 로그인 확인

---

# 계정 재생성 계획과 SealedSecret 과정

> 내 계획
> 1. db별로 계정과 암호를 정한다
> 2. sealed secret을 새로 만든다 
> 3. 내가 sealed secret 과정을 아직 제대로 이해 못했음. 차근차근 다시해보면서 알수 있는 과정이 될듯 (서버에 k8s secret을 만든다음에 이걸 무슨 과정을 거쳐서 암호화 한다음에 이걸 gitops에 올리는거 같던데)

## 답변

- 서버(클러스터)에 Secret을 먼저 만드는 것이 아님. 서버 셸에서 Secret YAML을 화면에만 만들고 바로 암호화함
- 클러스터에 실제 Secret이 생기는 것은 마지막 단계이며 sealed-secrets-controller가 만듦

## SealedSecret 흐름

```
[서버 셸]
① kubectl create secret ... --dry-run=client -o yaml
   → Secret YAML을 화면에만 생성 (클러스터에는 안 만듦, 값은 base64)
        │ 파이프(|)로 바로 넘김. 파일로 저장하지 않음
② kubeseal --controller-namespace choimory-infra
   → controller의 공개키를 받아 값을 암호화 → SealedSecret YAML 출력
        │ > ~/xxx.yaml
③ 이 파일을 gitops에 커밋, push (암호화 값이라 public 저장소에 올려도 안전)

[클러스터]
④ ArgoCD가 SealedSecret 적용
⑤ sealed-secrets-controller가 개인키로 복호화해 일반 Secret 생성
⑥ Pod(postgres, redis, member-api)가 Secret을 읽음
```

## 핵심 개념

| 개념 | 설명 |
|---|---|
| 공개키, 개인키 | controller가 키 쌍을 가짐. 공개키는 암호화만 가능하고 복호화는 controller의 개인키로만 가능. 그래서 git에 올려도 안전 |
| `--dry-run=client` | 클러스터에 보내지 않고 YAML만 생성. 원문이 클러스터나 파일에 남지 않음 |
| 이름, 네임스페이스 고정 | 암호화할 때 Secret 이름과 네임스페이스가 함께 묶임. stg용은 prod에서 풀리지 않으므로 환경별로 따로 생성 |
| 서버에서 하는 이유 | kubeseal이 공개키를 받으려면 클러스터 접근이 필요. 공개키를 파일로 받아 두면(`kubeseal --fetch-cert`) Mac에서도 가능 |
| 앞에서 본 원문 | ⑤에서 만들어진 일반 Secret을 `get secret`으로 꺼낸 것. 클러스터 안에서는 base64일 뿐 |

## 진행 제안

1. 연습 (가짜 값)
   - `test` 같은 값으로 ①만 실행해 base64 YAML 확인
   - ①②를 이어서 실행해 암호화 결과와 비교
   - 클러스터에는 아무것도 생기지 않음
2. 계정 정하기 (stg, prod 각각, 비밀번호는 채팅에 적지 않음)
   - PostgreSQL: DB 이름, 앱 계정, 내 계정
   - Redis: 앱 계정, 내 계정
3. 실제 생성
   - 계정명은 명령에 직접 적고, 비밀번호는 `read -rs`로 숨겨 입력
   - 결과 파일(암호화 값)은 채팅에 붙여도 됨. 문서에는 마스킹

## 확인 필요

- member-api의 Redis username 지원 여부. 지원하지 않으면 Redis 앱 계정은 `default`를 유지해야 할 수 있음. 2번 전에 member-api 저장소 확인 필요

---

# 일반 Secret과 SealedSecret 차이

> 궁금한거 
>
> 1. kubectl create secret은 나도 해본적 있음. 나는 다만 secret을 암호화해서 써본적은 없음. 서버에서 kubectl create secret 명렁어 쳐서 k8s 클러스터에 secret을 저장해놓으면 이걸 gitops에서 deployment.yaml에 secret key를 ref하면 자동으로 환경변수가 주입되는 방식이였음
> 2. sealed secret은 이과정ㅇ과 뭐가 달라서 암호화가 되는거지? sealed controller에서 암호화처리를 자동으로 해주는건가?
> 3. gitops의 deployment.yaml엔 뭐를 적는거지? 일ㄷ반 secret처럼 key 이름을 ref 하는건가?

## 답변

- 해 본 방식에서 "Secret을 클러스터에 직접 저장"하던 단계만 "암호화해서 git에 올리면 controller가 클러스터에 만들어 주는" 단계로 바뀜. 참조하는 쪽(deployment.yaml)은 그대로

## 1. 일반 Secret 방식

```
서버: kubectl create secret ...  → 클러스터에 Secret 저장
gitops: deployment.yaml에서 secretKeyRef로 참조 → 환경변수 주입
```

- 동작은 문제없지만 Secret이 클러스터에만 있고 git에는 없음
- 클러스터 재구축, 네임스페이스 삭제 시 Secret이 사라지고 값을 기억해 손으로 다시 만들어야 함
- ArgoCD는 이 Secret을 모름. git만으로 클러스터를 재현할 수 없음
- 일반 Secret YAML을 git에 올리면 안 됨. base64는 누구나 풀 수 있고 저장소는 public

## 2. SealedSecret의 차이

- Secret을 git에 올릴 수 있는 형태로 바꾸는 것이 차이

| 역할 | 누가 | 언제 |
|---|---|---|
| 암호화 | 사람이 실행하는 `kubeseal` 명령 (controller의 공개키만 받아 감) | Secret을 만들 때 (서버 셸) |
| 복호화 | sealed-secrets-controller가 자동으로 (개인키는 controller만 가짐) | ArgoCD가 SealedSecret을 적용했을 때 |

- controller가 자동으로 하는 것은 암호화가 아니라 복호화

```
[일반]   kubectl create secret            → 클러스터에 Secret
[Sealed] kubectl create secret --dry-run  → (YAML만 생성) → kubeseal(암호화)
         → git에 SealedSecret → ArgoCD 적용 → controller가 복호화 → 클러스터에 Secret
```

- 여기서 `kubectl create secret`은 클러스터 저장용이 아니라 Secret YAML을 만드는 생성기 역할
- 결과적으로 클러스터에는 일반 방식과 똑같은 일반 Secret이 생김 (그래서 `get secret`으로 원문 확인 가능)

## 3. deployment.yaml 작성

- 일반 Secret과 완전히 같음. SealedSecret이 아니라 controller가 만든 Secret의 이름과 key를 참조

```yaml
- name: redis-password
  valueFrom:
    secretKeyRef:
      name: choimory-dev-redis-secret   # controller가 만든 Secret 이름 (SealedSecret과 같은 이름)
      key: password
```

- Pod는 SealedSecret을 모르고 평소처럼 Secret만 읽음
- gitops에 추가되는 것은 암호화된 SealedSecret 파일 하나 (예: `redis/overlays/stg/choimory-dev-redis-secret.yaml`)이고, overlay의 `resources`에 넣음

---

# SealedSecret 이해 확인

> 내가 이해한게 맞으면 kube create secret 한다음에 kubeseal이란 명령어를 한번 더 치면 seal controller에서 암호화가 되는거고 이걸 gitops에 올리면 된다는건가?

## 답변

- 큰 흐름은 맞음. 두 군데만 보완

## 명령은 한 줄로 이어서 실행

```bash
kubectl create secret generic choimory-dev-redis-secret -n choimory-dev-stg \
  --from-literal=password="..." \
  --dry-run=client -o yaml \
| kubeseal --controller-namespace choimory-infra -o yaml \
> ~/choimory-dev-redis-secret.yaml
```

- `--dry-run=client`라 `kubectl create secret`은 클러스터에 저장하지 않고 Secret YAML만 출력
- 그 출력을 파이프(`|`)로 바로 `kubeseal`에 넘김
- "클러스터에 Secret을 만든 뒤 암호화"하는 두 단계가 아님

## 암호화는 kubeseal이 함

| 단계 | 하는 쪽 |
|---|---|
| 암호화 | 서버 셸에서 실행한 `kubeseal`. controller에게서 공개키만 받아 와 직접 암호화 |
| 복호화 | git에 올린 뒤 ArgoCD가 적용하면 controller가 자동으로 |

## 정리

1. `kubectl create secret --dry-run ... | kubeseal ...` 한 줄로 암호화된 파일 생성
2. 그 파일을 gitops에 올림 (kustomization의 `resources`에 추가)
3. 클러스터 안에서 controller가 복호화해 일반 Secret 생성, 앱은 그 Secret을 읽음

---

# SealedSecret 요약

> 쉽게말해서 kube secret yaml을 kubeseal하면 sealed된 yaml이 되는거고 이 파일을 그냥 gitops에 올리는거네. kube secret yaml은 안만들고 dry-run해서 클러스터에 저장안하는거고

## 답변

- 정확함

```
Secret YAML (dry-run이라 클러스터에 저장 안 함, 파일로도 안 남김)
   │ kubeseal
   ▼
SealedSecret YAML (암호화됨) → gitops에 올림
```

## 주의할 점

- 이름과 네임스페이스는 처음에 정확히 넣기. 암호화할 때 묶이므로 gitops에서 나중에 바꾸면 controller가 복호화하지 못함. stg, prod는 각각 따로 생성
- 파일만 두면 안 되고 overlay의 `kustomization.yaml` `resources`에 등록해야 ArgoCD가 적용

---

# deployment와 SealedSecret 연결

> 그럼 gitops에서는  deployment.yaml이 sealed-secret.yaml을 어케 바라봄?

## 답변

- deployment.yaml은 SealedSecret 파일을 직접 보지 않음. 둘은 이름으로만 이어짐

## 연결 구조

```
redis/overlays/stg/choimory-dev-redis-secret.yaml  (SealedSecret)
  metadata.name: choimory-dev-redis-secret
  namespace:     choimory-dev-stg
        │ controller가 복호화
        ▼
클러스터의 Secret (같은 이름, 같은 네임스페이스)
  name: choimory-dev-redis-secret
        ▲
        │ 이름으로 참조
choimory-dev-member-api/base/deployment.yaml
  secretKeyRef.name: choimory-dev-redis-secret
  secretKeyRef.key:  password
```

## 맞춰야 하는 것

| 항목 | 맞출 곳 |
|---|---|
| 이름 | SealedSecret `metadata.name` = deployment `secretKeyRef.name` |
| key | SealedSecret `encryptedData`의 key = deployment `secretKeyRef.key` |
| 네임스페이스 | SealedSecret과 Pod가 같은 네임스페이스 (Secret은 다른 네임스페이스에서 읽을 수 없음) |

- 파일 경로, 파일 이름은 무관. Redis Secret이 `redis/` 폴더에 있어도 같은 네임스페이스의 member-api가 읽음
- key 이름은 `kubectl create secret`의 `--from-literal=password=...`에서 `password` 부분으로 정해짐

---

# choimory-dev-postgres-app 위치

> member-api의 deployment보니까 secretKeyRef는 일반 secret이랑 차이 없는거 같은데 choimory-dev-postgres-app 이게 어디있는거지? overlays의 파일은 이름이 다른데

## 답변

- `choimory-dev-postgres-app`은 gitops에 파일이 없음. CNPG가 클러스터 안에서 자동으로 만드는 Secret

## member-api가 참조하는 Secret

| Secret | 만드는 주체 | gitops 파일 |
|---|---|---|
| `choimory-dev-postgres-app` | CNPG 오퍼레이터 (`Cluster`를 보고 자동 생성) | 없음 |
| `choimory-dev-redis-secret` | sealed-secrets-controller | `redis/overlays/{stg,prod}/choimory-dev-redis-secret.yaml` |
| `choimory-dev-member-api-secret` (`jwt-secret`) | sealed-secrets-controller | `choimory-dev-member-api/overlays/{stg,prod}/choimory-dev-member-api-secret.yaml` |

- member-api overlays의 파일은 `jwt-secret`용. `postgres/overlays/`에는 `kustomization.yaml`만 있음

## CNPG가 만드는 방식

```
postgres.yaml의 Cluster "choimory-dev-postgres"
  bootstrap.initdb: database member, owner member
        │ CNPG가 DB를 만들면서 비밀번호도 랜덤 생성
        ▼
Secret "{Cluster 이름}-app" = choimory-dev-postgres-app
  key: host, port, dbname, user, password, uri ...
```

- 이름 규칙 `{Cluster 이름}-app`. `postgres.yaml` 맨 위 주석에도 기재

---

# PostgreSQL만 예외

> member-api의 redis 환경변수는 찾아보니까 redis에 저장된 sealed secret 그대로 ref 시켰네. postgresㄱ만 좀 예외인건가 보네?

## 답변

- 맞음. PostgreSQL만 오퍼레이터(CNPG)가 DB, 계정을 관리해 Secret도 CNPG가 만드는 구조

| 대상 | Secret을 만드는 쪽 | member-api 참조 |
|---|---|---|
| Redis | 직접 만든 SealedSecret | 그 이름 그대로 `secretKeyRef` |
| jwt | 직접 만든 SealedSecret | 그 이름 그대로 `secretKeyRef` |
| PostgreSQL | CNPG 자동 생성 (`choimory-dev-postgres-app`) | CNPG가 정한 이름을 `secretKeyRef` |

## plan7 이후

- 계정을 직접 만들면 PostgreSQL도 Redis와 같은 방식

```
SealedSecret (직접 생성, postgres/overlays/{stg,prod}/)
  ├─ Cluster initdb.secret이 참조 → CNPG가 이 계정으로 DB 생성
  └─ member-api secretKeyRef가 참조 → 같은 계정으로 접속
```

- 직접 제공하면 CNPG는 `-app` Secret을 만들지 않음
- 직접 만드는 Secret(`basic-auth`)은 `username`, `password`만 있음. host(`choimory-dev-postgres-rw`), port(`5432`), dbname은 비밀이 아니므로 member-api env에 평문으로 직접 기재

---

# 계정 재생성 진행 계획

> 연습할 필요 없음 이제 계획 완전히 섰음 
>
> 1. db, redis 계정과 같은 환경변수 정보를 정한다 
> 2. 1을 바탕으로 create secret to sealed secret 명령어 생성
> 3. 해당 파일을 gitops에 저장 
>
> 맞잖아

## 답변

- 1~3은 맞음. 다만 Secret이 생기기만 하고 DB와 앱이 쓰지 않으므로 이후 단계 필요

## 전체 순서

| # | 할 일 | 하는 사람 |
|---|---|---|
| 0 | member-api의 Redis username 지원 여부 확인 (미지원이면 앱 수정) | 같이 확인 |
| 1 | 계정, 환경변수 값 정하기 | 사용자 |
| 2 | `create secret --dry-run \| kubeseal` 명령 실행 | 명령은 Claude 제공, 실행은 사용자 (서버) |
| 3 | SealedSecret 파일 gitops 저장, kustomization `resources` 등록 | Claude |
| 4 | 참조하는 쪽 수정 | Claude |
| 5 | push 후 DB 재생성, Redis 재시작, member-api 재시작 | 사용자 (서버) |
| 6 | 회원가입·로그인, DataGrip, redis-cli 확인 | 사용자 |

## 4번에서 바뀌는 파일

- `postgres/base/postgres.yaml`: `initdb`의 `database`, `owner`, `secret`, 내 계정용 `managed.roles`
- `redis/base/statefulset.yaml`, overlay: `--requirepass` 대신 ACL 파일로 username 필수
- `choimory-dev-member-api/base/deployment.yaml`: `-app` Secret 대신 새 Secret 참조, host·port·dbname 평문, Redis username env 추가

## 5번 주의

- `initdb`는 처음 만들 때만 적용되므로 기존 `Cluster`를 지워야 새 계정으로 재생성됨. 기존 DB 데이터는 사라짐
- stg 먼저 확인 후 prod

## 1번에서 정할 값 (stg, prod 각각)

| 대상 | 정할 것 |
|---|---|
| PostgreSQL | DB 이름, 앱 계정, 내 계정 |
| Redis | 앱 계정, 내 계정 |

- 비밀번호는 채팅에 적지 않고 2번에서 `read -rs`로 숨겨 입력

---

# 계정 재생성 최종 계획

> 연습할 필요 없음 이제 계획 완전히 섰음 
>
> 1. db, redis 계정과 같은 환경변수 정보를 정한다 
> 2. 1을 바탕으로 create secret to sealed secret 명령어 생성
> 3. 해당 파일을 gitops에 저장 
> 4. gitops deployment 수정

## 답변

- 순서 맞음. 4번 범위만 Secret을 쓰는 쪽 전부로

1. db, redis 계정과 환경변수 값 정하기
2. `create secret --dry-run | kubeseal` 명령 생성, 실행
3. SealedSecret 파일 gitops 저장 (`resources` 등록 포함)
4. gitops 수정
   - member-api `deployment.yaml`: 새 Secret 참조, host·port·dbname 평문, Redis username
   - `postgres.yaml`: `initdb`가 새 Secret 사용, 내 계정 추가
   - Redis `statefulset.yaml`: username 필수(ACL)

## 계획 밖에서 챙길 것

- 앞: member-api의 Redis username 지원 여부 확인. 미지원이면 4번에 앱 수정 포함
- 뒤: push 후 PostgreSQL `Cluster` 삭제로 재생성(기존 데이터 삭제), Redis, member-api 재시작, 접속 확인

## 다음

- stg, prod 각각 PostgreSQL(DB 이름, 앱 계정, 내 계정), Redis(앱 계정, 내 계정) 이름 정하기 (비밀번호 제외)
- member-api 저장소 경로 확인

---

# 추가1 

- postgres
  - 계정: choimory_dev_member, choimory_dev_blog, choimory_dev_feed.. (도메인별)
  - 비밀번호: 
- redis
  - 계정: choimory-dev (공통)
  - 비밀번호:  
- member 도메인 
  - postgres: member라는 database를 사용
  - redis: database 1 (0은 api-gateway 나오면 사용할수도 있으니 1부터)
- redis database는 도메인별로 다르게 사용할거임 (member:1, blog:2, feed:3..)
- 이 문서의 비밀번호는 secret 처리 끝나고 커밋하기 전에 바로 지우거나 마스킹할거임
- stg, prod 동일하게 할거임
- redis 설정에 username 설정 꺼져있을수 있음 켜야함
- member-api의 redis username은 빈칸으로 뒀을꺼임 아마 

---

# 앱 계정과 내 계정

> 앱계정과 내계정이 뭐임?

## 답변

- plan7 처음의 "redis, postgres 모두 별도 계정 추가"를 앱 계정, 내 계정으로 나눠 부른 것

| | 앱 계정 | 내 계정 |
|---|---|---|
| 쓰는 쪽 | member-api (프로그램) | 사람 (DataGrip, redis-cli) |
| 비밀번호 보관 | Secret에만 있고 앱이 env로 읽음 | 내가 기억하거나 DataGrip에 저장 |
| 권한 | DB 소유자(owner). Flyway가 스키마를 만들어야 해서 전부 | 필요한 만큼. 예: stg 읽기+쓰기, prod 읽기만 |
| 지금 상태 | PostgreSQL `member`, Redis 이름 없음(`default`) | 없음. 앱 계정으로 같이 접속 중 |

## 나누는 이유

1. prod 실수 방지: 읽기 전용 계정이면 실수한 UPDATE, DELETE를 DB가 거부. DataGrip Read-only보다 확실
2. 비밀번호 분리: 한쪽을 바꾸거나 노출돼도 다른 쪽은 그대로 (앱 재시작 불필요)
3. 기록 구분: DB 로그에서 앱과 사람이 한 일이 구분됨

- 혼자 쓰는 토이 프로젝트면 하나로 써도 동작 문제 없음. 추천은 나누기

---

# 추가1 분석 (첫 번째)

> 단축키가 안먹네 확인

## 답변

- 이 시점의 추가1: PostgreSQL, Redis 모두 계정 `choimory` 공통, 비밀번호 `****`(숫자 6자리), member 도메인은 PostgreSQL 스키마 `choimory-dev-member`, Redis database 0

## 짚은 점

1. 비밀번호 평문: 저장소가 public이라 push하면 공개됨. 마스킹 필요, 대화에 나온 값은 실제로 쓰지 않는 것이 안전
2. 비밀번호 강도: 숫자 6자리는 약함. stg, prod, PostgreSQL, Redis가 모두 같으면 하나 노출 시 전부 노출
3. 계정 하나 공통: `initdb.owner`만 지정하면 되고 `managed.roles` 불필요. 대신 prod 읽기 전용은 DataGrip 설정에만 의존
4. DB와 스키마 구분: PostgreSQL은 DB 안에 스키마. 스키마면 member-api Flyway, JPA 설정 변경 필요. 하이픈 이름은 SQL에서 항상 따옴표 필요 → 밑줄 추천
5. Redis database 0은 기본값. username 필수(ACL)는 member-api 지원 여부 확인 필요

---

# 추가1 분석 (두 번째)

> 추가 1 ㅎ다시 확인

## 답변

- 이 시점의 추가1: 계정 `choimory` 마스터 계정으로 앱, 개발자 공용, 비밀번호 `****`, PostgreSQL DB `choimory-dev-member`, Redis member는 0, 도메인별 database 번호, stg·prod 동일, 비밀번호는 커밋 전 마스킹

## 짚은 점

1. DB 이름 하이픈: JDBC, DataGrip은 그대로 가능. SQL에서 DB 이름을 직접 쓸 때는 따옴표 필요
2. 마스터 계정 범위: CNPG `owner`는 해당 DB의 소유자일 뿐 슈퍼유저가 아님. 다른 도메인 DB는 CNPG `Database` 리소스로 추가하며 owner 지정
3. Redis database 번호: 단일 인스턴스라 0~15 사용 가능. ACL 사용자에 전체 권한이면 모든 번호 접근
4. 모든 곳 같은 짧은 비밀번호는 하나 노출 시 전부 노출 (tailnet 전용을 감안한 선택으로 이해)

---

# 추가1 분석 (세 번째)

> 추가1 다시 확인

## 답변

- 계정 구조가 도메인 단위로 바뀜

## 확정 사항

| 항목 | 결정 |
|---|---|
| PostgreSQL 계정 | 도메인별. 이번 `choimory-dev-member`, 이후 `choimory-dev-blog`, `choimory-dev-feed` ... |
| PostgreSQL DB | member 도메인은 `member` (기존과 같은 이름) |
| Redis 계정 | `choimory-dev` 공통 |
| Redis database | 0은 api-gateway용으로 비움. member:1, blog:2, feed:3 ... |
| 환경 | stg, prod 동일 |
| 비밀번호 | Secret 처리 후 커밋 전 삭제 또는 마스킹 |

## 짚은 점

1. PostgreSQL 계정 하이픈: SQL에서 계정을 쓸 때(`GRANT ... TO "choimory-dev-member"`) 따옴표 필요. `choimory_dev_member` 추천
2. DB 이름이 같아도 소유자가 바뀌므로 `initdb`로 재생성 필요. 기존 데이터 삭제
3. Redis username "켜기": Redis에는 on/off 옵션이 따로 없음. 지금은 `--requirepass`로 `default`만 사용. ACL로 `choimory-dev`를 만들고 `default`를 끄면 username 필수
4. member-api Redis username, database: Spring Boot는 OS 환경변수가 `application.yml`보다 우선. deployment에 `SPRING_DATA_REDIS_USERNAME`, `SPRING_DATA_REDIS_DATABASE`를 넣으면 앱 수정 없이 적용될 가능성이 큼. 단, 앱이 Redis 연결을 코드로 직접 만들면 무시됨
5. blog, feed 추가 시: `managed.roles`로 계정, CNPG `Database` 리소스로 DB 추가. 이번에는 member만

## Secret 구성 (안)

| Secret | 형식 | key | 쓰는 곳 |
|---|---|---|---|
| `choimory-dev-postgres-member` (신규) | basic-auth | `username`, `password` | `Cluster` `initdb.secret`, member-api |
| `choimory-dev-redis-secret` (같은 이름으로 재생성) | 일반 | `username`, `password` | Redis(ACL), member-api |

- Redis ACL은 실행 인자로 지정 가능 (예: `--user default off --user choimory-dev on >$(REDIS_PASSWORD) ~* &* +@all`). Secret에 `users.acl` 불필요

---

# 추가1 결정과 앱 저장소 확인

> 1. 언더바로 변경
> 2. 버리고 재생성해도 됨 
> 3. member-api 저장소 경로 이건 ㅁ이해못함 설명좀

## 답변

- PostgreSQL 계정 `choimory_dev_member`로 확정
- 기존 DB 데이터 버리고 재생성 확정

## 앱 저장소를 봐야 하는 이유

| 저장소 | 들어 있는 것 |
|---|---|
| `choimory-gitops` | 배포 설정. member-api를 어떻게 띄울지 (deployment.yaml, env, Secret 참조) |
| member-api 앱 저장소 | Spring 소스 코드. 어떻게 동작할지 (`application.yml`, Redis 연결 코드). GitHub Actions가 이미지(`ghcr.io/choimory/choimory-dev-member-api`)로 빌드 |

- gitops는 env를 넣어 줄 뿐이고, 앱이 그 env를 쓰는지는 앱 코드에 달림
  - Spring Boot 기본 방식(`spring.data.redis.*`)이면 `SPRING_DATA_REDIS_USERNAME`, `SPRING_DATA_REDIS_DATABASE` env로 해결. 앱 수정 불필요
  - `RedisConfig` 등에서 연결을 직접 만들면 env가 무시되어 Redis 로그인 실패. 앱 수정 필요
- Mac의 `IdeaProjects/choimory/` 아래에는 member-api 저장소가 없음

## 선택지

| 방법 | 내용 |
|---|---|
| A. 경로 알려주기 | clone된 경로를 알려주면 설정을 읽어 확인 |
| B. 직접 확인 | `spring.data.redis` 설정, `RedisConnectionFactory`/`LettuceConnectionFactory` 직접 생성 코드 여부 확인 |
| C. 확인 없이 진행 | env를 넣고 stg에 적용해 보고, 실패하면 그때 앱 수정 (prod 영향 없음) |

- 추천 A 또는 B. 앱 수정 필요 여부를 미리 알면 gitops와 앱을 함께 배포해 다운타임이 줄어듦

---

# member-api 저장소 위치

> 아 member-api repo 위치 알려달라는거야?

## 답변

- 맞음. member-api 소스 코드가 Mac 어디에 clone되어 있는지 경로
- 찾기: `find ~ -type d -name "*member-api*" -not -path "*/node_modules/*" 2>/dev/null`
- clone되어 있지 않으면 GitHub 주소, 또는 확인 없이 진행(C)도 가능

---

# member-api Redis, DB 설정 확인

> /Users/choimory/choimory_workspace/IdeaProjects/choimory-dev/choimory-dev-member-api

## 답변

- 앱 코드 수정 불필요. gitops env만 변경

## 확인 결과

| 항목 | 앱 설정 | 결과 |
|---|---|---|
| Redis 연결 방식 | `RedisConnectionFactory` 직접 생성 코드 없음. Spring Boot 자동 설정 | env 그대로 적용 |
| Redis username | `application-redis.yml`에 `username: ${redis-username}` 이미 있음 | 지원함 |
| Redis database | 설정 없음 (기본 0) | env `SPRING_DATA_REDIS_DATABASE=1` 추가로 해결 |
| PostgreSQL | `application-datasource.yml`이 `rdb-host`, `rdb-port`, `rdb-database`, `rdb-user`, `rdb-password`로 URL 조립. 스키마 지정 없음(`public`) | env 값만 변경 |

- gitops `deployment.yaml`에 이미 `redis-username: ""`이 있고 주석에 "ACL을 쓰지 않아 비어 있지만 환경변수 자체는 필요하다"고 되어 있음. 처음부터 ACL을 염두에 둔 구조

## member-api deployment.yaml 변경 (안)

| env | 지금 | 변경 |
|---|---|---|
| `rdb-host` | `-app` Secret `host` | 평문 `choimory-dev-postgres-rw` |
| `rdb-port` | `-app` Secret `port` | 평문 `"5432"` |
| `rdb-database` | `-app` Secret `dbname` | 평문 `member` |
| `rdb-user` | `-app` Secret `user` | `choimory-dev-postgres-member` Secret `username` |
| `rdb-password` | `-app` Secret `password` | `choimory-dev-postgres-member` Secret `password` |
| `redis-username` | `""` | `choimory-dev-redis-secret` Secret `username` |
| `redis-password` | 그대로 | 그대로 (Secret만 재생성) |
| `SPRING_DATA_REDIS_DATABASE` | 없음 | 평문 `"1"` (신규) |

- `SPRING_DATA_REDIS_DATABASE`만 이름 형식이 다름. 앱 yml에 `${redis-database}` 자리가 없어 Spring Boot 표준 env 이름 사용. 통일하려면 나중에 앱 yml에 `database: ${redis-database:0}` 추가 (이번 범위 아님)

## 다음

- SealedSecret 생성 명령 (서버, stg·prod 각각)
  - PostgreSQL `choimory-dev-postgres-member`: basic-auth, `username=choimory_dev_member`
  - Redis `choimory-dev-redis-secret`: `username=choimory-dev`
- 비밀번호는 `read -rs`로 숨겨 입력
