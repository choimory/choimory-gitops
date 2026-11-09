# 목차

- [개요](#개요)
- [Quota 조정과 구현 시작](#quota-조정과-구현-시작)
  - [답변](#답변)
  - [Quota 조정](#quota-조정)
  - [1단계 작성안](#1단계-작성안)
  - [확인된 점](#확인된-점)
- [1단계 member-api 작성](#1단계-member-api-작성)
  - [답변](#답변-1)
  - [변경 파일](#변경-파일)
  - [오타 수정의 영향](#오타-수정의-영향)
  - [커밋](#커밋)
  - [다음 단계: 2단계 Actions 워크플로](#다음-단계-2단계-actions-워크플로)

---

# 개요

- plan1~plan4에서 확정한 계획에 따라 k8s 구축 구현을 시작함
- 이 문서는 구현 순서 1단계(member-api stg 프로필 작성)를 기록함
- calendar 상태를 `TASK:k8s 구축 작업(k8s-setup)`으로 변경함
- 참고 문서: [plan3.md](./plan3.md)(자원 값), [plan4.md](./plan4.md)(구현 순서)

---

# Quota 조정과 구현 시작

> Quota 그 값으로 조정하고 구현 시작하자

## 답변

- ResourceQuota 값을 조정하고, calendar를 TASK로 변경함
- member-api의 현재 설정을 확인한 뒤, 파일을 고치기 전에 1단계 작성안을 먼저 제시함

## Quota 조정

- plan3의 Quota(prod 2Gi, stg 1.5Gi)로는 롤링 업데이트 중 새 Pod가 Quota에 걸림
  - prod: Pod 상한 합계 1,664Mi + member-api 추가 1개 768Mi = 2,432Mi로 2Gi 초과. 새 Pod 생성이 거부되어 배포가 멈춤
  - stg: Pod 상한 합계 1,024Mi + member-api 추가 1개 512Mi = 1,536Mi로 1.5Gi에 딱 맞아 여유가 없음
- Pod별 상한은 plan3 값을 그대로 두고 네임스페이스 Quota만 올림

| 네임스페이스 | 이전 | 확정 |
| --- | --- | --- |
| `choimory-dev-prod` | 2Gi | 3Gi |
| `choimory-dev-stg` | 1.5Gi | 2Gi |

- 3단계 gitops 작성 때 반영 예정

## 1단계 작성안

- 대상: `choimory-dev-member-api`의 `dev` 브랜치 작업 트리
- `application-stg.yml` 새로 작성, `application.yml` group에 `stg: stg, flyway` 추가

## 확인된 점

- `application-prod.yml`, `application-dev.yml`의 로그 키가 `org:hibernate.SQL`로, 점이 아니라 콜론으로 잘못 적혀 있었음
- member-api에 `Dockerfile`, `.dockerignore`가 스테이징만 된 상태였음. 2단계 Actions 빌드에 필요
- member-api의 git remote는 아직 `choimory-io-member-external-api`로 남아 있음

---

# 1단계 member-api 작성

> 오타도 같이 고치고 작성해줘

## 답변

- 1단계 파일 작업을 완료함
- 커밋은 하지 않았고, 빌드나 앱 실행으로 검증하지 않음

## 변경 파일

| 파일 (`member-api/src/main/resources/`) | 변경 |
| --- | --- |
| `application-stg.yml` | 새로 작성 |
| `application.yml` | group에 `stg: stg, flyway` 추가 |
| `application-prod.yml` | `org:hibernate.SQL` → `org.hibernate.SQL` |
| `application-dev.yml` | `org:hibernate.SQL` → `org.hibernate.SQL` |

### application-stg.yml

```yaml
spring:
  config:
    activate:
      on-profile: stg

  jpa:
    show-sql: false
    generate-ddl: false
    hibernate:
      ddl-auto: none
    properties:
      hibernate:
        show_sql: false
        format_sql: false

logging:
  level:
    root: info
    dev.choimory: debug
    org.hibernate.SQL: info

# CORS 허용 Origin. 프론트 주소(스킴, 포트 포함)
cors:
  allowed-origins:
    - https://staging.choimory.dev
```

### application.yml

```diff
             prod: prod, flyway
+            stg: stg, flyway
             dev: dev, flyway
```

## 오타 수정의 영향

- 앞으로 Hibernate SQL 로그 레벨이 의도대로 적용됨 (prod `error`, dev `info`)
- dev는 원래 `show-sql: true`라서 쿼리 출력에는 큰 차이가 없을 것으로 예상

## 커밋

- `application-*` 변경과 스테이징된 `Dockerfile`, `.dockerignore`를 `dev` 브랜치에 함께 커밋 필요 (사용자가 직접 진행)

## 다음 단계: 2단계 Actions 워크플로

- 두 앱 레포의 `Dockerfile`, 기존 `.github/workflows`, 빌드 설정을 확인한 뒤 작성 전에 내용을 먼저 제시
- 트리거: `stg` push → stg, `main` push → prod
- 이미지: `ghcr.io/choimory/choimory-dev-member-api`, `ghcr.io/choimory/choimory-dev-front`, 태그 `stg-<sha>`, `prod-<sha>`
- 마지막 단계: gitops 레포의 해당 overlay 이미지 태그를 수정해 커밋
- [ ] gitops 레포 커밋 인증 방식 (추천: `choimory-gitops`에만 Contents 쓰기 권한을 준 Fine-grained PAT)
