# 목차

- [개요](#개요)
  - [계획](#계획)
- [계획 확인](#계획-확인)
  - [답변](#답변)
  - [1. SealedSecret 생성 명령 (서버)](#1-sealedsecret-생성-명령-서버)
  - [2. gitops에 SealedSecret 추가](#2-gitops에-sealedsecret-추가)
  - [3. 그 외 gitops 수정](#3-그-외-gitops-수정)
  - [4. 반영과 확인 (추가)](#4-반영과-확인-추가)
  - [주의](#주의)
- [member-batch용 DB와 계정](#member-batch용-db와-계정)
  - [batch가 저장하는 데이터](#batch가-저장하는-데이터)
  - [선택지](#선택지)
  - [나중에 추가할 때](#나중에-추가할-때)
- [member DB 스키마 분리](#member-db-스키마-분리)
  - [member-batch 만들 때 챙길 점 (앱)](#member-batch-만들-때-챙길-점-앱)
- [member-api 스키마 확인](#member-api-스키마-확인)
  - [api도 이름 있는 스키마로 옮길지](#api도-이름-있는-스키마로-옮길지)
- [스키마 간 CRUD](#스키마-간-crud)
  - [방법](#방법)
  - [조건: 권한](#조건-권한)
  - [앱에서](#앱에서)
  - [DB가 다르면 불가](#db가-다르면-불가)
- [모듈별 스키마 분리](#모듈별-스키마-분리)
  - [모듈별로 나누면 생기는 문제](#모듈별로-나누면-생기는-문제)
  - [데이터 주인 기준](#데이터-주인-기준)
  - [public을 쓰지 않으려면](#public을-쓰지-않으려면)
  - [마이그레이션 주인](#마이그레이션-주인)
- [batch의 public, batch 스키마 동시 사용](#batch의-public-batch-스키마-동시-사용)
- [JPA 엔티티별 스키마](#jpa-엔티티별-스키마)
  - [기본 스키마 정하기](#기본-스키마-정하기)
  - [Spring Batch 메타데이터는 별개](#spring-batch-메타데이터는-별개)
  - [주의할 점](#주의할-점)
- [최종 스키마 구성 확인](#최종-스키마-구성-확인)
- [~/.cloudflared 정체](#cloudflared-정체)
- [~/.cloudflared 전체 삭제](#cloudflared-전체-삭제)

---

# 개요

- plan7에서 PostgreSQL, Redis 계정 구성을 확정함
  - PostgreSQL: 도메인별 계정. member는 계정 `choimory_dev_member`, DB `member`. 기존 DB는 버리고 재생성
  - Redis: 계정 `choimory-dev` 공통, ACL로 username 필수. member는 database 1
  - stg, prod 동일
- member-api 앱 코드 수정 없이 gitops env만 변경하면 됨 (Redis username 지원 확인)
- 이어서 SealedSecret 생성, gitops 수정(postgres, redis, member-api), DB 재생성과 재시작, 접속 확인을 진행
- 참고 문서: [plan7.md](./plan7.md)(추가1, 추가1 결정과 앱 저장소 확인, member-api Redis, DB 설정 확인)

## 계획 

- kube secret to sealed secret 명령어 생성
- 생성한 sealed secret을 gitops에 추가
- 그 외 수정해야할 gitops 관련 코드 수정 

---

# 계획 확인

> plan8 확인

## 답변

- "계획" 3단계 순서는 맞음. 단계별 구체화와 4번(반영과 확인) 추가

## 1. SealedSecret 생성 명령 (서버)

- stg, prod 같은 값이라 비밀번호는 한 번만 입력하고 반복문으로 두 환경을 함께 생성

```bash
# 비밀번호 입력 (화면과 셸 기록에 남지 않음)
read -rs PG_PW; echo
read -rs REDIS_PW; echo

for env in stg prod; do
  # PostgreSQL: CNPG initdb용 basic-auth
  kube create secret generic choimory-dev-postgres-member -n choimory-dev-$env \
    --type=kubernetes.io/basic-auth \
    --from-literal=username=choimory_dev_member \
    --from-literal=password="$PG_PW" \
    --dry-run=client -o yaml \
  | kubeseal --controller-namespace choimory-infra -o yaml \
  > ~/choimory-dev-postgres-member-$env.yaml

  # Redis: 기존과 같은 이름으로 다시 만들고 username key 추가
  kube create secret generic choimory-dev-redis-secret -n choimory-dev-$env \
    --from-literal=username=choimory-dev \
    --from-literal=password="$REDIS_PW" \
    --dry-run=client -o yaml \
  | kubeseal --controller-namespace choimory-infra -o yaml \
  > ~/choimory-dev-redis-secret-$env.yaml
done

unset PG_PW REDIS_PW
ls -l ~/choimory-dev-*-{stg,prod}.yaml
```

- 결과 파일 4개. 암호화 값이라 채팅에 붙여도 됨
- 실행 전 명령 재확인

## 2. gitops에 SealedSecret 추가

| 파일 | 위치 |
|---|---|
| `choimory-dev-postgres-member.yaml` (신규) | `postgres/overlays/{stg,prod}/`, kustomization `resources`에 등록 |
| `choimory-dev-redis-secret.yaml` (기존 파일 내용 교체) | `redis/overlays/{stg,prod}/` (이미 등록됨) |

## 3. 그 외 gitops 수정

| 파일 | 수정 |
|---|---|
| `postgres/base/postgres.yaml` | `initdb`: `database: member`, `owner: choimory_dev_member`, `secret.name: choimory-dev-postgres-member`. 맨 위 주석(`-app` Secret 설명) 수정 |
| `redis/overlays/{stg,prod}/kustomization.yaml` | args의 `--requirepass $(REDIS_PASSWORD)`를 ACL로 교체: `--user default off --user $(REDIS_USERNAME) on >$(REDIS_PASSWORD) ~* &* +@all` |
| `redis/base/statefulset.yaml` | env `REDIS_USERNAME` 추가 (Secret `username`) |
| `choimory-dev-member-api/base/deployment.yaml` | plan7 정리대로 env 변경 (rdb-* 5개, `redis-username`, `SPRING_DATA_REDIS_DATABASE=1`) |
| `README.md` | Secret 목록, DB 접속 정보 |

- Redis 이미지 `8.10.3`이라 ACL 사용 가능. probe는 TCP 포트 검사라 인증 영향 없음

## 4. 반영과 확인 (추가)

1. push하면 member-api, Redis는 설정 변경으로 자동 재시작
2. PostgreSQL은 `initdb`가 처음 만들 때만 적용되므로 `Cluster`를 직접 삭제 → ArgoCD가 새 설정으로 재생성
3. 접속 확인: 회원가입·로그인, DataGrip(`choimory_dev_member`), `redis-cli --user choimory-dev`

## 주의

- base 파일 수정이라 stg, prod에 동시에 반영됨
- push부터 `Cluster` 재생성까지 두 환경의 member-api가 새 계정으로 접속하지 못해 재시작 반복 (몇 분)
- 데이터를 버리기로 했으므로 한 번에 진행하고 stg, prod `Cluster`를 이어서 재생성 추천

---

# member-batch용 DB와 계정

> 궁금한게 있는데 내가 member-batch를 만든다면 batch용 db를 또 하나 파야 하지 않을까 싶은데 그럼 계정도 배치 db용 계정 만들고 db도 batch용으로 만들고 그래야함?

## 답변

- 보통은 새 DB 불필요. member-batch는 member 도메인 데이터를 처리하므로 같은 `member` DB 사용이 자연스러움

## batch가 저장하는 데이터

| 데이터 | 내용 | 위치 |
|---|---|---|
| 업무 데이터 | 회원 테이블 등 batch가 읽고 쓰는 대상 | `member` DB (member-api와 같은 DB) |
| Spring Batch 메타데이터 | `BATCH_JOB_INSTANCE`, `BATCH_JOB_EXECUTION` 등 실행 이력 | 어디든 가능 |

- PostgreSQL은 한 연결로 다른 DB 테이블을 조회할 수 없음. 메타데이터만 별도 DB로 빼면 batch 앱에 datasource가 두 개 필요해 복잡해짐

## 선택지

| 방법 | 구성 | 언제 |
|---|---|---|
| A. 같은 DB, 같은 계정 | `member` DB, `choimory_dev_member`. 메타데이터도 같은 DB | 가장 단순 |
| B. 같은 DB, 스키마 분리 (추천) | `member` DB 안 `batch` 스키마에 메타데이터. 계정은 같은 것 또는 batch 전용 | 업무 테이블과 이력 테이블을 나누고 싶을 때 |
| C. 별도 DB와 계정 | `member_batch` DB, `choimory_dev_member_batch` 계정 | batch 데이터가 member와 독립적일 때. member 데이터를 다루는 batch에는 비추천 |

- batch 전용 계정은 권한 분리의 문제. 같은 계정이면 간단, 전용 계정이면 필요한 테이블만 GRANT 가능하지만 권한 SQL 관리 필요

## 나중에 추가할 때

- `Cluster` 재생성 불필요 (`initdb`는 첫 DB만)

| 추가할 것 | 방법 |
|---|---|
| 계정 | `Cluster`의 `managed.roles` + SealedSecret(basic-auth) |
| DB | CNPG `Database` 리소스 (owner 지정) |
| 스키마 | 앱의 Flyway 마이그레이션, Spring Batch 설정(`spring.batch.jdbc.table-prefix`) |

---

# member DB 스키마 분리

> 그럼 db는 같은 member로 하고 schema로 구분함?

## 답변

- 맞음. `member` DB 하나에 스키마 두 개

```
DB member (owner: choimory_dev_member)
├── public 스키마  ← member-api 업무 테이블 (지금 Flyway가 만드는 곳, 그대로)
└── batch 스키마   ← member-batch의 Spring Batch 메타데이터
```

- `choimory_dev_member`가 DB 소유자라 `batch` 스키마 생성 권한 있음. gitops 변경 없음

## member-batch 만들 때 챙길 점 (앱)

1. 메타데이터 위치: `spring.batch.jdbc.table-prefix: batch.BATCH_`
2. 테이블 생성은 Flyway로: `initialize-schema` 자동 생성은 스키마 이름 없이 만들어 위치가 어긋날 수 있음. Spring Batch의 `schema-postgresql.sql`을 Flyway 마이그레이션으로 옮기고 앞에 `CREATE SCHEMA batch;`
3. Flyway 이력 분리 (가장 중요): member-api, member-batch가 둘 다 `public.flyway_schema_history`를 쓰면 서로의 마이그레이션을 모르는 버전으로 보고 충돌. member-batch는 `spring.flyway.schemas: batch`로 이력을 따로 둠
4. 회원 테이블 구조 변경(마이그레이션)은 member-api 한 곳에서만

- 계정은 처음엔 `choimory_dev_member` 공용, 권한을 좁힐 필요가 생기면 batch 전용 계정을 `managed.roles`로 추가

---

# member-api 스키마 확인

> api는 public으로 되어있지 않음?

## 답변

- 맞음. member-api는 `public` 사용
  - 앱 설정(`application-datasource.yml`, `application-flyway.yml`)에 스키마 지정 없음
  - 마이그레이션 SQL(`V1__init_schema.sql` 등)도 스키마 이름 없이 `create table if not exists member`
  - 지정이 없으면 PostgreSQL 기본 스키마 `public`에 생성

## api도 이름 있는 스키마로 옮길지

| | public 유지 (추천) | `api` 스키마로 이동 |
|---|---|---|
| 앱 변경 | 없음 | Flyway `schemas: api`, JDBC URL `currentSchema=api` 등 앱 수정과 재배포 |
| 장점 | 지금 그대로 동작 | 이름으로 용도 구분 |
| 시점 | - | 하려면 지금이 가장 쉬움 (DB 재생성이라 데이터 이전 불필요) |

- 추천 public 유지. 스키마 둘이면 `public`=api, `batch`=batch로 충분히 구분. 옮기면 앱 수정, 재배포가 이번 범위에 들어옴

---

# 스키마 간 CRUD

> 다른 schema끼리 crud 할 수 있어?

## 답변

- 가능. 같은 DB 안이면 스키마가 달라도 CRUD, JOIN 자유롭게 가능 (스키마로 나누자고 한 이유)

## 방법

```sql
-- batch 스키마에서 public의 회원 조회
SELECT * FROM public.member WHERE ...;

-- 스키마를 넘나드는 JOIN
SELECT m.id, e.status
FROM public.member m
JOIN batch.batch_job_execution e ON ...;

-- INSERT, UPDATE, DELETE도 같은 방식
UPDATE public.member SET ... WHERE ...;
```

- 스키마 이름을 생략하면 `search_path`(기본 `public`)에서 찾음. member-api가 `member`만 써도 `public.member`를 찾는 이유
- 한 트랜잭션 안에서 여러 스키마를 같이 수정 가능 (커밋, 롤백 함께 적용)

## 조건: 권한

| 상황 | 필요한 것 |
|---|---|
| 같은 계정 (`choimory_dev_member`가 두 스키마 모두 소유) | 추가 설정 없음 |
| batch 전용 계정을 따로 만들 경우 | `GRANT USAGE ON SCHEMA public`, 테이블별 `GRANT SELECT, UPDATE ...` |

## 앱에서

- JPA 엔티티는 `@Table(schema = "batch", name = "...")`로 스키마 지정. 지정하지 않으면 기본 스키마(`public`)
- member-batch는 회원 엔티티를 기본값(`public`) 그대로, Spring Batch 메타데이터만 `table-prefix`로 `batch`

## DB가 다르면 불가

| 구분 | 서로 조회 |
|---|---|
| 같은 DB, 다른 스키마 | 가능 (`schema.table`) |
| 다른 DB (예: `member`와 `blog`) | 불가. `dblink`, `postgres_fdw` 같은 확장 필요, 트랜잭션도 묶이지 않음 |

- member-batch용 DB를 따로 만들지 말자고 한 이유. 반대로 member와 blog처럼 도메인이 다르면 DB를 나눠 서로의 데이터를 직접 건드리지 못하게 하는 것이 의도에 맞음

---

# 모듈별 스키마 분리

> public은 사용안하고 api, grpc, batch 이런식으로 모듈별로 schema 나누는건 어케 생각해?

## 답변

- batch에는 맞지만 api, grpc에는 맞지 않음. 스키마는 "어느 모듈이 접속하는가"가 아니라 "데이터의 주인이 누구인가"로 나누는 것이 좋음

## 모듈별로 나누면 생기는 문제

```
api 스키마  ← member 테이블을 여기 두면
grpc 모듈   → api.member를 읽고 씀 (grpc인데 api 스키마를 씀)
batch 모듈  → api.member를 읽고 씀
```

- api, grpc는 진입 방식(HTTP, gRPC)만 다르고 같은 회원 데이터를 다룸
- 데이터는 하나여야 하므로 grpc 스키마에 복사할 수 없음. 스키마 이름이 실제 주인과 어긋남

## 데이터 주인 기준

| 데이터 | 주인 | 스키마 |
|---|---|---|
| 회원, 권한, 약관 등 업무 데이터 | member 도메인 (api, grpc, batch 공유) | 도메인 공용 스키마 하나 |
| Spring Batch 실행 이력 | batch 모듈만 | `batch` |
| (예시) grpc 전용 데이터 | grpc 모듈만 | `grpc` |

- 모듈 이름 스키마는 그 모듈만 쓰는 데이터가 있을 때만

## public을 쓰지 않으려면

```
DB member
├── member  ← 업무 데이터 (api, grpc, batch 공유)
└── batch   ← Spring Batch 메타데이터
(public은 비워 둠)
```

- 이름은 `member`(DB 이름과 같아도 됨)나 `core` 추천
- 앱에서 Flyway `schemas: member`, JDBC URL `currentSchema=member` 설정 필요. 하려면 DB 재생성 중인 지금이 가장 쉬움 (앱 수정, 재배포 포함)

## 마이그레이션 주인

- 같은 업무 스키마를 여러 모듈이 쓰면 Flyway 마이그레이션은 한 곳에서만 (member-api 또는 멀티 모듈의 공용 모듈). 각자 하면 버전 충돌

---

# batch의 public, batch 스키마 동시 사용

> 그럼 batch가 있는데 회원정보 테이블은 public에 있으면 batch 어플리케이션이 두 스키마를 모두 오고갈수 있어?

## 답변

- 가능. 연결 하나로 두 스키마 모두 접근

```
member-batch ──(연결 1개: DB member, 계정 choimory_dev_member)──┐
                                                                ├── public.member 등  ← 회원 데이터 읽기, 쓰기
                                                                └── batch.BATCH_JOB_* ← 실행 이력 기록
```

- 연결은 DB 단위, 스키마는 DB 안의 구분. datasource 하나면 충분

| 대상 | 설정 | 결과 |
|---|---|---|
| 회원 엔티티 | 지정 없음 | 기본 `search_path`인 `public` |
| Spring Batch 메타데이터 | `spring.batch.jdbc.table-prefix: batch.BATCH_` | `batch.BATCH_JOB_INSTANCE` 등 |

- Step 안의 회원 데이터 처리와 실행 이력 기록이 같은 트랜잭션으로 묶임 (같은 DB)
- 조건: 같은 계정이면 권한 설정 없음. Flyway 이력은 `spring.flyway.schemas: batch`로 분리
- public을 비우고 `member` 스키마로 옮기는 것은 깔끔함을 위한 선택 사항

---

# JPA 엔티티별 스키마

> 하나의 spring batch JPA에서 엔티티를 설정할때 스키마가 다 달라도?

## 답변

- 가능. EntityManagerFactory 하나 안에서 엔티티마다 스키마를 다르게 지정 가능 (같은 DB 조건)

```java
@Entity
@Table(name = "member")                     // 스키마 생략 → 기본 스키마(public)
public class Member { ... }

@Entity
@Table(schema = "batch", name = "member_sync_log")   // batch 스키마
public class MemberSyncLog { ... }

@Entity
@Table(schema = "stats", name = "daily_signup")      // 또 다른 스키마도 가능
public class DailySignup { ... }
```

- Hibernate가 SQL에 `public.member`, `batch.member_sync_log`처럼 스키마를 붙임
- DataSource, TransactionManager 하나. 한 트랜잭션으로 여러 스키마 엔티티 저장, 롤백
- 연관관계와 FK도 스키마를 넘어 가능

## 기본 스키마 정하기

| 방법 | 설정 |
|---|---|
| Hibernate 기본값 | `spring.jpa.properties.hibernate.default_schema: member` |
| DB 연결 기본값 | JDBC URL `?currentSchema=member` (`search_path` 변경) |
| 아무것도 안 함 | PostgreSQL 기본 `public` |

## Spring Batch 메타데이터는 별개

- `BATCH_JOB_*`는 JPA 엔티티가 아니라 Spring Batch가 JDBC로 직접 다룸. `@Table` 대신 `spring.batch.jdbc.table-prefix`로 지정, DataSource는 같은 것 사용

## 주의할 점

| 항목 | 내용 |
|---|---|
| 네이티브 쿼리 | Hibernate가 스키마를 붙여 주지 않으므로 직접 기재 |
| 테이블 생성 | `ddl-auto` 대신 Flyway, 스키마는 `CREATE SCHEMA`로 미리 생성 |
| 다른 DB 테이블 | 엔티티 하나로 접근 불가. DataSource, EntityManagerFactory를 따로 둬야 함 |

---

# 최종 스키마 구성 확인

> 그럼 일반적인 회원도메인 테이블은 public 스키마에 두고, 배치에서만 쓰는 메타데이터 테이블은 batch 스키마에두고 이런게 가능한거야?

## 답변

- 가능. 추천한 방식

```
DB member (계정 choimory_dev_member)
├── public  ← 회원 도메인 테이블 (member-api, member-batch가 같이 사용)
└── batch   ← Spring Batch 메타데이터 (member-batch만 사용)
```

| 대상 | 지정 방법 | 생성 |
|---|---|---|
| 회원 도메인 테이블 | 엔티티 스키마 지정 없음 (기본 `public`) | member-api Flyway (지금 그대로) |
| Batch 메타데이터 | `spring.batch.jdbc.table-prefix: batch.BATCH_` | member-batch Flyway (`spring.flyway.schemas: batch`) |

- member-batch는 연결, JPA 설정, 트랜잭션 하나로 두 스키마 사용
- 이번 gitops 작업에는 영향 없음. member-batch 만들 때 앱 설정 두 줄과 Flyway 마이그레이션만 추가
- batch 전용 업무 테이블이 생기면 `@Table(schema = "batch")`로 같은 스키마에

---

# ~/.cloudflared 정체

> ~/.cloudflared 이건 뭐였지

## 답변

- 2단계(Cloudflare Tunnel, task3)에서 서버에 cloudflared CLI를 설치하고 Tunnel을 만들 때 생긴 폴더

| 명령 | 생긴 파일 | 정체 |
|---|---|---|
| `cloudflared tunnel login` | `cert.pem` | Cloudflare 계정 인증서. `choimory.dev` 영역에서 Tunnel 생성·삭제, DNS 레코드(`tunnel route dns`) 생성 가능 |
| `cloudflared tunnel create choimory-tunnel` | `<UUID>.json` | Tunnel 인증 파일(credentials). 있으면 누구든 `choimory-tunnel`의 connector가 될 수 있음 |

- `<UUID>.json`: SealedSecret `choimory-cloudflared-credentials`로 gitops에 들어가 클러스터 cloudflared Pod가 사용. 서버 원본은 쓸 일이 없어 "연결 확인 후 삭제" 대상이었음
- `cert.pem`: Tunnel 설정 변경, CLI로 DNS 레코드 추가 대비로 보관 예정이었음 (권한 `600`)

---

# ~/.cloudflared 전체 삭제

> 다 지우면 돼?

## 답변

- 전부 삭제 가능. 오히려 더 안전

| 파일 | 지운 뒤 영향 | 다시 필요할 때 |
|---|---|---|
| `<UUID>.json` | 없음. 클러스터는 SealedSecret 사용 | 클러스터 Secret에서 꺼내거나 `cloudflared tunnel token` |
| `cert.pem` | 없음. 실행 중인 Tunnel과 무관, CLI 관리용 | `cloudflared tunnel login` 한 번으로 재발급 (브라우저 인증) |

- DNS 레코드는 대시보드에서도 생성 가능하고 필요할 때 다시 로그인하면 되므로, 쓰지 않는 계정 인증서를 서버에 두지 않음

```bash
rm -rf ~/.cloudflared
```

- 서버의 cloudflared CLI 프로그램은 재로그인용으로 남겨 둬도 됨
