# 목차

- [개요](#개요)
- [5단계 gitops 작성](#5단계-gitops-작성)
  - [답변](#답변)
  - [변경 파일](#변경-파일)
  - [검증](#검증)
  - [push 후 확인](#push-후-확인)
  - [그다음 (접속 확인)](#그다음-접속-확인)

---

# 개요

- plan6에서 확정한 5단계(PostgreSQL, Redis Tailscale 접속)를 진행함
- 확정 값: PostgreSQL은 CNPG `Cluster`의 `managed.services.additional`(P1), Redis는 전용 Service를 별도 파일로(R1), 기존 `redis.yaml`도 리소스별 파일로 분리
- tailnet 기기 이름: `choimory-dev-postgres-{stg,prod}`, `choimory-dev-redis-{stg,prod}`. 접근 제한 없음 (tailnet 멤버 = 함께 개발하는 사람)
- 4단계에서 설치한 Tailscale Operator 재사용. NetworkPolicy, Tailscale 관리 화면, 인프라 영역 변경 없음
- 참고 문서: [plan6.md](./plan6.md)(5단계 최종 작업계획), [task4.md](./task4.md)(Operator 설치)

---

# 5단계 gitops 작성

> 응 5단계 작업 시작해줘

## 답변

- A(gitops 작성), B(검증) 완료. 커밋, push는 사용자 진행

## 변경 파일

| 파일 | 변경 |
|---|---|
| `postgres/base/postgres.yaml` | `managed.services.additional`에 rw 대상 `choimory-dev-postgres-tailnet` (`LoadBalancer`, `loadBalancerClass: tailscale`). 주석에 "overlay가 `additional[0]`을 패치하므로 순서를 바꾸면 overlay도 고칠 것" |
| `postgres/overlays/stg/kustomization.yaml` | JSON 패치로 `tailscale.com/hostname: choimory-dev-postgres-stg` |
| `postgres/overlays/prod/kustomization.yaml` | JSON 패치로 `tailscale.com/hostname: choimory-dev-postgres-prod` |
| `redis/base/redis.yaml` | 삭제 |
| `redis/base/service.yaml` | 기존 Service `choimory-dev-redis` (주석 포함 그대로) |
| `redis/base/statefulset.yaml` | 기존 StatefulSet `choimory-dev-redis` (주석 포함 그대로) |
| `redis/base/service-tailnet.yaml` | 새 Service `choimory-dev-redis-tailnet`. prune이 꺼져 있어 지울 때 직접 삭제해야 한다는 주석 |
| `redis/base/kustomization.yaml` | resources를 위 3개 파일로 |
| `redis/overlays/{stg,prod}/kustomization.yaml` | 병합 패치로 `tailscale.com/hostname: choimory-dev-redis-{stg,prod}` |
| `README.md` | 디렉토리 구조 설명, "관리 화면과 DB 접속"을 Tailscale 접속 표로 변경, tailnet 멤버 원칙과 prod Read-only 추가, port-forward는 "Tailscale 경로 문제 시" 비상용 |

- PostgreSQL overlay가 JSON 패치인 이유: `additional`은 CRD 안의 목록이라 일반 병합 패치는 목록 전체를 덮어씀

## 검증

- 수정 전 4개 overlay 빌드 결과를 저장해 두고 수정 후 결과와 비교

| overlay | 결과 |
|---|---|
| redis stg, prod | 기존 Service, StatefulSet이 수정 전과 완전히 같음. 추가는 `choimory-dev-redis-tailnet` Service 하나 (기기 이름 annotation 환경별 정상) |
| postgres stg, prod | `Cluster`에 `managed.services.additional` 블록만 추가. annotation `choimory-dev-postgres-{stg,prod}` 정상. 나머지 설정 변화 없음 |

- push해도 Redis, PostgreSQL Pod 재시작 없음
- `redis.yaml`은 `git rm`으로 삭제해 삭제만 스테이징된 상태. 나머지 변경은 스테이징하지 않음

## push 후 확인

- 반영까지 최대 3분. 늦으면 `kube -n choimory-infra annotate application root argocd.argoproj.io/refresh=hard --overwrite`

```bash
kube -n choimory-dev-stg get svc choimory-dev-postgres-tailnet choimory-dev-redis-tailnet
kube -n choimory-dev-prod get svc choimory-dev-postgres-tailnet choimory-dev-redis-tailnet
kube -n choimory-infra get pods | grep ts-
kube -n choimory-dev-stg get cluster choimory-dev-postgres
kube -n choimory-dev-prod get cluster choimory-dev-postgres
```

- Service 4개 모두 EXTERNAL-IP `100.x`
- `ts-` 프록시 Pod 4개 추가 (`infra-gateway`까지 5개)
- `Cluster` STATUS "Cluster in healthy state"
- 커밋 메시지 예: `feat: PostgreSQL, Redis를 Tailscale Operator로 tailnet에 노출 (redis 매니페스트 리소스별 분리)`

## 그다음 (접속 확인)

- Mac, Tailscale 켬

```bash
nc -vz choimory-dev-postgres-stg 5432
nc -vz choimory-dev-redis-stg 6379
```

- 포트가 열리면 DataGrip으로 접속. prod는 Read-only
