# 목차

- [개요](#개요)
- [infrastructure 네임스페이스를 choimory-infra로 변경](#infrastructure-네임스페이스를-choimory-infra로-변경)
  - [답변](#답변)
  - [영향 범위](#영향-범위)
  - [영향이 없는 것](#영향이-없는-것)
  - [진행안](#진행안)
    - [1번에서 바꿀 곳](#1번에서-바꿀-곳)
  - [확인할 것](#확인할-것)
  - [추천](#추천)
- [전체 계획](#전체-계획)
  - [답변](#답변-1)
  - [전체 흐름](#전체-흐름)
  - [0. 사전 확인과 키 재백업 (서버)](#0-사전-확인과-키-재백업-서버)
  - [1. Git 수정 (Claude)](#1-git-수정-claude)
  - [2. 옛 구성 정리 (서버)](#2-옛-구성-정리-서버)
  - [3. gitops 커밋, main 반영 (직접)](#3-gitops-커밋-main-반영-직접)
  - [4. choimory-infra에 재설치 (서버)](#4-choimory-infra에-재설치-서버)
  - [5. 확인과 정리 (서버)](#5-확인과-정리-서버)
  - [문제가 생기면](#문제가-생기면)
  - [예상 소요](#예상-소요)

---

# 개요

- 공용 도구 네임스페이스 `infrastructure`를 `choimory-infra`로 변경하려 함
- 네임스페이스 안에 ArgoCD, sealed-secrets, CloudNativePG 오퍼레이터, Envoy Gateway 등 공용 도구가 모두 있어, 단순 이름 변경이 아니라 공용 도구 재설치 작업이 됨
- 앱 레포 머지와 Run workflow(7단계)는 이 작업 후 진행
- 참고 문서: [plan3.md](./plan3.md)(네임스페이스 규칙), [task5.md](./task5.md)(ArgoCD 설치, sealed-secrets 키 백업), [plan9.md](./plan9.md), [task7.md](./task7.md)

---

# infrastructure 네임스페이스를 choimory-infra로 변경

> gitops 하나만 더 수정하자. infrastructure 네임스페이스를 choimory-infra로 바꿔줘

## 답변

- 변경 가능하지만, 공용 도구 전체를 새 네임스페이스로 다시 설치하는 작업임
- Git만 수정해서 push하는 방식으로는 안전하게 옮길 수 없고, 서버에서 옛 구성을 정리하고 ArgoCD를 다시 설치하는 절차가 필요함
- 아직 파일은 수정하지 않음

## 영향 범위

| 들어 있는 것 | 옮길 때 문제 |
|---|---|
| ArgoCD 본체, 모든 Application·ApplicationSet·AppProject | ArgoCD는 자기 네임스페이스의 Application만 봄. Git만 바꾸면 새 네임스페이스의 Application은 아무도 처리하지 않고, 옛 Application은 prune되면서 연결된 리소스까지 지워질 수 있음 |
| sealed-secrets 컨트롤러와 암호화 키 | 새 네임스페이스에 새로 설치하면 새 키가 만들어져 기존 SealedSecret 4개를 풀지 못함. 백업 키(`~/sealed-secrets-key-backup.yaml`)를 먼저 복원해야 함 |
| CloudNativePG 오퍼레이터, Envoy Gateway, shared-gateway, local-path, metrics-server | 옛 것과 새 것이 동시에 돌면 컨트롤러끼리 충돌함. 옛 것을 먼저 지워야 함 |

## 영향이 없는 것

- postgres, redis Pod와 PVC는 `choimory-dev-stg`, `choimory-dev-prod`에 있어 그대로 유지됨. 이미 만들어진 Secret도 유지됨
- CRD(Cluster, SealedSecret 등)는 클러스터 단위 리소스라 네임스페이스를 지워도 남음
- 외부에 공개된 서비스가 없어 작업 중 Gateway가 잠시 멈춰도 영향 없음

## 진행안

| 순서 | 작업 | 담당 |
|---|---|---|
| 0 | 사전 확인: 키 백업 파일 위치, Application의 finalizer 유무 (finalizer가 있으면 Application 삭제 시 리소스도 같이 지워짐) | 직접 실행, 판단은 Claude |
| 1 | Git 수정: 네임스페이스 `infrastructure` → `choimory-infra` 전체 변경 | Claude |
| 2 | 서버: 모든 Application의 finalizer 제거 → ArgoCD 삭제 → `infrastructure` 네임스페이스 삭제 | 직접 |
| 3 | gitops 커밋, `main` 반영 | 직접 |
| 4 | 서버: `choimory-infra`에 ArgoCD 설치 → sealed-secrets 키 복원 → root 등록 | 직접 |
| 5 | 확인: 모든 앱 `Healthy`, SealedSecret 정상, postgres·redis 재시작 없음, 남은 클러스터 리소스 정리 | 직접 실행, 확인은 Claude |

- 2번은 반드시 3번(push)보다 먼저 진행. 옛 ArgoCD가 살아 있는 상태에서 바뀐 Git을 읽으면 prune 문제가 생김
- 명령어는 단계마다 하나씩 안내

### 1번에서 바꿀 곳

- `bootstrap/root.yaml`, `infrastructure/apps/*.yaml` 9개: Application의 네임스페이스와 배포 대상 네임스페이스
- `infrastructure/apps/namespace.yaml`: Namespace 이름
- `infrastructure/gateway`, `local-path-provisioner`의 kustomization 네임스페이스
- `projects/choimory-dev/argocd` 3개: AppProject, ApplicationSet 2개
- HTTPRoute 2개: `parentRefs` 네임스페이스
- NetworkPolicy: 허용 네임스페이스와 이름(`allow-same-namespace-and-infra`)
- 주석과 sealed-secrets values의 `kubeseal --controller-namespace` 안내

## 확인할 것

1. sealed-secrets 키 백업 파일이 서버의 `~/sealed-secrets-key-backup.yaml`에 아직 있는지, 다른 곳에 보관했는지. 없으면 진행 전에 지금 키를 다시 백업해야 함
2. gitops 레포의 디렉토리 이름 `infrastructure/`는 그대로 두고 네임스페이스만 바꾸는지
3. 재설치하면 ArgoCD 관리자 비밀번호가 새로 생성됨

## 추천

- 할 거라면 지금 진행 추천. 앱이 아직 배포 전이고 DB가 비어 있어, 나중에 할수록 부담이 커짐
- 앱 레포 머지와 Run workflow는 이 작업이 끝난 뒤 진행. 앱 레포 워크플로는 네임스페이스와 무관해 수정할 필요 없음

---

# 전체 계획

> 1. 어디에 있는지 모르겠음
> 2. 디렉토리명은 infrastructure로 유지
> 3. 계획 한번 말해줘

## 답변

- 키 백업 위치를 모르므로 0번에서 지금 키를 다시 백업하는 것으로 시작함. 키는 아직 `infrastructure`의 컨트롤러에 있어 다시 받을 수 있음
- gitops 디렉토리 `infrastructure/`는 유지하고 네임스페이스만 변경

## 전체 흐름

```
0. 사전 확인과 키 재백업 (서버)
1. Git 수정 (Claude, 커밋은 아직 안 함)
2. 옛 구성 정리 (서버)          ← 반드시 3번보다 먼저
3. gitops 커밋, main 반영 (직접)
4. choimory-infra에 재설치 (서버)
5. 확인과 정리 (서버)
```

## 0. 사전 확인과 키 재백업 (서버)

- sealed-secrets 키 재백업. 이 파일이 없으면 기존 SealedSecret 4개를 풀 수 없게 됨
- Application, ApplicationSet, AppProject의 finalizer 유무 확인
- Helm 릴리스 목록과 웹훅(클러스터 단위 설정) 확인. 5번에서 남은 것을 정리할 때 비교용
- 결과를 보고 2번 명령어를 확정

## 1. Git 수정 (Claude)

- 네임스페이스만 `infrastructure` → `choimory-infra`로 변경. 디렉토리 `infrastructure/`는 유지
- 바꿀 곳은 [1번에서 바꿀 곳](#1번에서-바꿀-곳) 목록과 같음
- `kubectl kustomize`로 빌드 검증. 커밋은 2번이 끝날 때까지 하지 않음

## 2. 옛 구성 정리 (서버)

| 순서 | 작업 | 이유 |
|---|---|---|
| 1 | ArgoCD 컨트롤러(application-controller, applicationset-controller)를 0개로 줄임 | 정리하는 동안 ArgoCD가 다시 만들거나 지우지 못하게 멈춤 |
| 2 | Application, ApplicationSet, AppProject의 finalizer 제거 | Application을 지울 때 연결된 리소스(DB 등)가 같이 지워지지 않게 함 |
| 3 | Application, ApplicationSet, AppProject 삭제 | 껍데기만 지우고 실제 리소스는 남음 |
| 4 | `helm uninstall argocd -n infrastructure` | ArgoCD 본체 삭제. CRD는 차트 기본 설정상 남음 |
| 5 | `infrastructure` 네임스페이스 삭제 | 남은 공용 도구(sealed-secrets, CloudNativePG 오퍼레이터, Envoy Gateway, local-path, metrics-server) 삭제 |

- 이 동안에도 postgres, redis Pod는 계속 돎. 오퍼레이터가 없어도 이미 떠 있는 Pod는 멈추지 않음

## 3. gitops 커밋, main 반영 (직접)

- 옛 ArgoCD가 없어진 뒤에 push하므로, 바뀐 Git을 옛 ArgoCD가 읽을 일이 없음

## 4. choimory-infra에 재설치 (서버)

| 순서 | 작업 | 이유 |
|---|---|---|
| 1 | 서버의 gitops 클론을 `git pull` | 바뀐 values와 root.yaml을 씀 |
| 2 | `choimory-infra` 네임스페이스 생성 | |
| 3 | 백업한 키를 네임스페이스만 바꿔 `choimory-infra`에 복원 | sealed-secrets가 새 키를 만들지 않고 기존 키를 씀. root 등록보다 먼저 해야 함 |
| 4 | `helm install argocd ... -n choimory-infra` | task5와 같은 명령어에 네임스페이스만 변경 |
| 5 | `kubectl apply -f bootstrap/root.yaml` | 이후 공용 도구와 프로젝트 앱은 ArgoCD가 설치 |

- task5에서 겪은 gateway Degraded, GatewayClass 문제가 다시 나올 수 있음. 그때와 같은 방법으로 처리

## 5. 확인과 정리 (서버)

- 확인
  - 모든 Application `Synced`, `Healthy` (front, member-api는 이미지가 없어 `Degraded`가 정상)
  - SealedSecret 4개 `SYNCED True`. 기존 키로 풀렸다는 뜻
  - postgres, redis Pod의 AGE와 RESTARTS 그대로, PVC `Bound`
  - HTTPRoute가 새 Gateway(`choimory-infra`)에 연결됨
- 정리
  - 0번 결과와 비교해 옛 네임스페이스를 가리키는 클러스터 단위 리소스(웹훅, ClusterRoleBinding 등)가 남아 있으면 삭제
  - ArgoCD 새 관리자 비밀번호로 로그인 후 `argocd-initial-admin-secret` 삭제
  - 키 백업 파일은 서버 밖 안전한 곳(비밀번호 관리자 등)에 보관하고 위치를 기록

## 문제가 생기면

- 4번에서 문제가 생겨도 백업 키가 있으므로 되돌릴 수 있음. Git을 되돌린 뒤 `infrastructure`에 같은 방법으로 다시 설치
- 앱 데이터(DB, Redis)는 프로젝트 네임스페이스에 있어 어느 단계에서도 지우지 않음

## 예상 소요

- 30분~1시간. 대부분은 4번에서 ArgoCD가 공용 도구를 차례로 설치하는 대기 시간
