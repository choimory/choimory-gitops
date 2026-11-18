# 목차

- [개요](#개요)
- [계정 재생성 작업 시작](#계정-재생성-작업-시작)
  - [답변](#답변)
  - [변경 파일](#변경-파일)
  - [검증](#검증)
  - [SealedSecret 생성 (서버)](#sealedsecret-생성-서버)
  - [다음](#다음)
- [서버 홈 파일 정리](#서버-홈-파일-정리)
- [SealedSecret 파일 gitops 반영](#sealedsecret-파일-gitops-반영)
  - [변경 파일](#변경-파일-1)
  - [검증](#검증-1)
  - [커밋](#커밋)
- [push 후 Cluster 재생성](#push-후-cluster-재생성)
  - [Cluster 재생성 중 문제와 해결](#cluster-재생성-중-문제와-해결)
  - [결과](#결과)
- [접속 확인](#접속-확인)
  - [결과](#결과-1)
  - [남은 일](#남은-일)

---

# 개요

- plan8에서 확정한 PostgreSQL, Redis 계정 재생성을 진행함
- 확정 값: PostgreSQL 계정 `choimory_dev_member`(DB `member`), Redis ACL 사용자 `choimory-dev`(member는 database 1), stg·prod 동일
- member-api 앱 코드 수정 없음. gitops env만 변경
- 참고 문서: [plan7.md](./plan7.md)(추가1, member-api Redis, DB 설정 확인), [plan8.md](./plan8.md)(계획 확인)

---

# 계정 재생성 작업 시작

> 작업 시작하자

## 답변

- SealedSecret 파일이 필요 없는 gitops 수정을 먼저 진행 (plan8 3번)
- SealedSecret 파일 추가와 postgres overlay `resources` 등록(plan8 2번)은 서버에서 파일을 만든 뒤 진행. 파일 없이 등록하면 빌드 실패

## 변경 파일

| 파일 | 변경 |
|---|---|
| `postgres/base/postgres.yaml` | `initdb.owner: choimory_dev_member`, `initdb.secret.name: choimory-dev-postgres-member`. 맨 위 주석을 새 Secret 기준으로, "initdb는 처음 만들 때만 적용" 주의 추가 |
| `redis/base/statefulset.yaml` | env `REDIS_USERNAME` 추가 (`choimory-dev-redis-secret`의 `username`) |
| `redis/overlays/{stg,prod}/kustomization.yaml` | args `--requirepass $(REDIS_PASSWORD)` → `--user default off --user $(REDIS_USERNAME) on ">$(REDIS_PASSWORD)" "~*" "&*" "+@all"` |
| `choimory-dev-member-api/base/deployment.yaml` | `rdb-host`·`rdb-port`·`rdb-database` 평문(`choimory-dev-postgres-rw`, `5432`, `member`), `rdb-user`·`rdb-password`는 `choimory-dev-postgres-member`, `redis-username`은 `choimory-dev-redis-secret`의 `username`, `SPRING_DATA_REDIS_DATABASE: "1"` 추가 |
| `README.md` | 비밀값 표(`choimory-dev-postgres-app` → `choimory-dev-postgres-member`, Redis Secret 설명), DB 접속 표(사용자, database 번호), 계정 규칙과 Redis ACL 설명 |

- Redis args의 `>`, `&`는 YAML 특수 문자라 따옴표로 감쌈

## 검증

- 수정 전 6개 overlay(postgres, redis, member-api × stg, prod) 빌드 결과를 저장해 두고 수정 후와 비교

| overlay | 결과 |
|---|---|
| postgres stg, prod | `initdb`의 `owner`, `secret`만 변경 |
| redis stg, prod | args의 인증 부분만 ACL로 변경, env `REDIS_USERNAME` 추가. AOF, maxmemory 등 나머지 동일 |
| member-api stg, prod | DB, Redis env만 변경, `SPRING_DATA_REDIS_DATABASE` 추가 |

## SealedSecret 생성 (서버)

- 비밀번호는 한 번씩 입력해 stg, prod 파일 4개 생성

```bash
read -rs PG_PW; echo
read -rs REDIS_PW; echo

for env in stg prod; do
  kube create secret generic choimory-dev-postgres-member -n choimory-dev-$env \
    --type=kubernetes.io/basic-auth \
    --from-literal=username=choimory_dev_member \
    --from-literal=password="$PG_PW" \
    --dry-run=client -o yaml \
  | kubeseal --controller-namespace choimory-infra -o yaml \
  > ~/choimory-dev-postgres-member-$env.yaml

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

- `read -rs` 두 줄은 각각 실행 후 비밀번호 입력, Enter (첫 번째 PostgreSQL, 두 번째 Redis). 입력 내용은 화면에 보이지 않음

## 다음

- 아직 커밋, push 금지. SealedSecret 파일 없이 반영되면 Pod가 뜨지 않음
- 서버에서 생성한 파일 내용 확인 (암호화 값이라 채팅에 붙여도 됨)

```bash
cat ~/choimory-dev-postgres-member-stg.yaml ~/choimory-dev-postgres-member-prod.yaml ~/choimory-dev-redis-secret-stg.yaml ~/choimory-dev-redis-secret-prod.yaml
```

- 그 내용으로 gitops에 SealedSecret 파일 추가, postgres overlay kustomization `resources` 등록 (plan8 2번)

---

# 서버 홈 파일 정리

> 예전것들인데 sealed secret key backup 이것도 지워도 되나

## 답변

- `sealed-secrets-key-backup.yaml`은 지우면 안 됨. controller 개인키라 이것만 있으면 저장소의 모든 SealedSecret을 풀 수 있고, 재설치 시 이 키를 복원해야 기존 SealedSecret이 풀림
- 서버 안에만 있으면 서버 고장 시 백업도 같이 사라짐 → Mac으로 복사 후 비밀번호 관리자 등 암호화된 저장소에 보관(git, 일반 클라우드, 노션 금지), 그다음 평문 사본 삭제

| 파일 | 정체 | 삭제 |
|---|---|---|
| `choimory-cloudflared-credentials.yaml`, `cloudflare-api-token.yaml`, `operator-oauth.yaml` | kubeseal 결과. gitops에 커밋됨 | 가능 |
| `sealed/` | k8s-setup 때 SealedSecret 결과로 추정 | 내용 확인 후 가능 |
| `kubeadm-config.yaml` | Tailscale 값이 채워진 실제 사용본 | 보관 권장 |
| `cilium-values.yaml` | 저장소 `bootstrap/`에 같은 파일 | 가능 |
| `docker-before.txt` | k8s 전환 전 메모로 추정 | 직접 판단 |

- 숨김 폴더의 `~/.cloudflared/<UUID>.json`(Tunnel 인증 원본)은 삭제 대상, `cert.pem`은 보관
- 이후 사용자가 예전 파일을 정리하고 `kubernetes-keys/` 폴더로 키 백업을 옮김 (서버 밖 보관은 아직)

---

# SealedSecret 파일 gitops 반영

> (서버에서 SealedSecret 생성 명령 실행, 결과 파일 4개 `cat` 출력. 암호화 값 마스킹)
>
> ```
> -rw-rw-r-- 1 choimory choimory 1772 Nov 18 19:27 /home/choimory/choimory-dev-postgres-member-prod.yaml
> -rw-rw-r-- 1 choimory choimory 1770 Nov 18 19:27 /home/choimory/choimory-dev-postgres-member-stg.yaml
> -rw-rw-r-- 1 choimory choimory 1723 Nov 18 19:27 /home/choimory/choimory-dev-redis-secret-prod.yaml
> -rw-rw-r-- 1 choimory choimory 1721 Nov 18 19:27 /home/choimory/choimory-dev-redis-secret-stg.yaml
> ```

## 답변

- 파일 4개 정상 생성. 이름, 네임스페이스, key, 형식이 계획과 일치

## 변경 파일

| 파일 | 변경 |
|---|---|
| `postgres/overlays/{stg,prod}/choimory-dev-postgres-member.yaml` | 신규. basic-auth(`username`, `password`), 맨 위 설명 주석 |
| `postgres/overlays/{stg,prod}/kustomization.yaml` | `resources`에 위 파일 등록 |
| `redis/overlays/{stg,prod}/choimory-dev-redis-secret.yaml` | 새 SealedSecret으로 교체(`username` key 추가), 주석을 ACL 기준으로 수정 |

## 검증

- overlay 6개 빌드 성공. SealedSecret 이름, 네임스페이스가 Cluster, StatefulSet, member-api 참조와 일치
- `choimory-dev-protected`(postgres, redis)는 `selfHeal: true`라 Cluster 삭제 시 ArgoCD가 자동 재생성

## 커밋

- 문서(`.agents/histories`) 제외, gitops 파일과 README만 staged. 커밋, push는 사용자 진행

---

# push 후 Cluster 재생성

> db 접속 안됨

## 답변

- push 반영은 정상. Redis는 ACL 설정으로 재시작됨
- `initdb`는 처음 만들 때만 적용되어 DB는 아직 예전 계정(`member`). 새 member-api는 없는 계정 `choimory_dev_member`로 접속하다 CrashLoopBackOff (예상된 상태)
- Secret 확인 → Cluster 삭제 → healthy 확인 → member-api 재시작 순서 안내

> redis도 안되는데

## 답변

- 예전 member-api Pod는 username 없이 예전 비밀번호로 접속해 ACL로 바뀐 Redis에 로그인 불가. 새 Pod는 DB 때문에 아직 뜨지 못함
- 클라이언트는 `default` 사용자가 꺼져 있어 `--user choimory-dev` 필수. 서버 `ACL LIST`, Mac `redis-cli`로 원인 구분 방법 안내

## Cluster 재생성 중 문제와 해결

| 환경 | 증상 | 원인 | 해결 |
|---|---|---|---|
| prod | 삭제 4초 뒤 재생성된 Cluster가 `Cluster is unrecoverable and needs manual intervention`, PRIMARY `choimory-dev-postgres-1`, Pod 없음 | selfHeal이 예전 인스턴스 정리 전에 같은 이름으로 재생성 → 새 Cluster가 예전 인스턴스 흔적을 자기 것으로 판단 | PVC가 없는 것을 확인하고 Cluster 한 번 더 삭제 → `initdb` Job 실행, `Setting up primary` |
| stg | 새 Cluster가 `Waiting for the instances to become active`, 예전 `postgres-1` Pod와 PVC가 Terminating에서 멈춤 | CNPG Pod의 긴 종료 대기 시간(기본 최대 30분). Cluster가 먼저 지워져 정상 종료가 끝나지 않음. Pod가 PVC를 잡고 있어 PVC 삭제도 멈춤 | `delete pod --grace-period=0 --force` → PVC 삭제 확인 → Cluster 한 번 더 삭제 |

```bash
kube -n choimory-dev-stg delete pod choimory-dev-postgres-1 --grace-period=0 --force
kube -n choimory-dev-stg get pvc
kube -n choimory-dev-stg delete cluster choimory-dev-postgres
```

- 다음에 Cluster를 다시 만들 때 순서: Cluster 삭제 → 예전 Pod, PVC가 사라졌는지 확인 → 새 Cluster 상태가 이상하면 한 번 더 삭제

## 결과

```
NAME                    AGE   INSTANCES   READY   STATUS                     PRIMARY
choimory-dev-postgres   75s   1           1       Cluster in healthy state   choimory-dev-postgres-1   (stg)
choimory-dev-postgres   2m22s 1           1       Cluster in healthy state   choimory-dev-postgres-1   (prod)
```

- member-api 재시작 후 stg, prod 모두 새 Pod `1/1 Running`, 예전 Pod 제거. 새 계정 DB 접속과 Flyway 마이그레이션 완료

---

# 접속 확인

> postgres는 잘 되는거 확인
> redis는 datagrip 접속 안되는중

## 답변

- DataGrip Redis는 Authentication을 `User & Password`로, User `choimory-dev`, Database `1`. `Password`만 쓰면 `default` 사용자로 시도해 실패
- 에러 메시지별 원인: `WRONGPASS`(사용자 이름 누락, 비밀번호 오타), `NOAUTH`(인증 미전송), timeout(Tailscale, Service)

> 웹페이지 회원가입 잘되는것도 확인. redis 접속만 해결하면 될듯

## 답변

- 회원가입은 DB만 사용. 로그인은 refresh token을 Redis에 저장하므로 로그인, 마이페이지로 앱의 Redis ACL 접속 확인

> redis도 잘 된다 계정명을 잘못적었어

## 결과

| 항목 | stg | prod |
|---|---|---|
| PostgreSQL `Cluster` | healthy, 계정 `choimory_dev_member` | healthy |
| member-api | 새 계정으로 DB, Redis 접속 | 같음 |
| 웹 회원가입 | 확인 | 확인 |
| DataGrip (PostgreSQL, Redis) | 확인 (Redis는 계정명 오타 수정 후) | - |

## 남은 일

1. 정리: plan7 "추가1"의 비밀번호 삭제 또는 마스킹, 서버 홈의 SealedSecret 결과 파일 4개 삭제, sealed-secrets 키 백업 서버 밖 보관, `~/.cloudflared/<UUID>.json` 삭제
2. domain-connect 마무리: done1.md, calendar `DONE`, README 최종 점검 (Cluster 재생성 순서 추가 검토)
