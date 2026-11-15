# 목차

- [개요](#개요)
- [현재 상태](#현재-상태)
- [남은 작업 순서](#남은-작업-순서)
  - [1. 6단계 마무리](#1-6단계-마무리)
  - [2. 7단계 전 준비 (직접)](#2-7단계-전-준비-직접)
  - [3. 7단계 1차 확인](#3-7단계-1차-확인)
  - [4. 2차 외부 공개](#4-2차-외부-공개)
- [중간에 정리할 것](#중간에-정리할-것)
- [미뤄둔 작업](#미뤄둔-작업)
  - [디렉토리 이름 변경](#디렉토리-이름-변경)
- [아직 검증하지 못한 것](#아직-검증하지-못한-것)
- [범위를 정해야 하는 것](#범위를-정해야-하는-것)
- [답변 대기 중인 문서 항목](#답변-대기-중인-문서-항목)

---

# 개요

- task5에서 5단계(ArgoCD 설치와 root 등록)를 완료하고, 6단계(SealedSecret)를 레포에 반영함
- 6단계는 커밋, push 후 클러스터 반영 확인만 남음
- 이 문서는 이후 진행할 작업과 미뤄둔 작업을 정리함
- 진행 기록은 [task5.md](task5.md) 참고

---

# 현재 상태

| 항목 | 상태 |
|---|---|
| 공용 도구(ArgoCD, sealed-secrets, local-path, metrics-server, cloudnative-pg, envoy-gateway, gateway) | 모두 `Synced`, `Healthy` |
| `choimory-dev` ApplicationSet, AppProject, Application 8개 | 생성됨 |
| `*-namespaces` | `Healthy` |
| postgres(`choimory-dev-postgres-1`) | stg, prod 모두 `Running` |
| redis(`choimory-dev-redis-0`) | Secret이 반영되기 전이라 `CreateContainerConfigError` |
| front, member-api | 이미지 태그가 `*-bootstrap` 임시값이라 `ImagePullBackOff` |
| SealedSecret 4개 | 레포에 추가됨, 커밋 전 |

---

# 남은 작업 순서

## 1. 6단계 마무리

1. 커밋, push(직접): SealedSecret 4개, kustomization 4개, task5, plan8. `main`까지 반영
2. 반영 확인(서버). push 후 1~3분 뒤 실행

```bash
kubectl get sealedsecrets -n choimory-dev-stg
kubectl get sealedsecrets -n choimory-dev-prod
kubectl get secret -n choimory-dev-stg | grep choimory-dev
kubectl get secret -n choimory-dev-prod | grep choimory-dev
kubectl -n choimory-dev-stg get pods
kubectl -n choimory-dev-prod get pods
kubectl -n infrastructure get applications | grep choimory-dev
```

- 완료 기준
  - Secret `choimory-dev-redis-secret`, `choimory-dev-member-api-secret`이 stg, prod에 생성됨
  - `choimory-dev-redis-0`이 `Running`
  - `choimory-dev-{stg,prod}-data`가 `Healthy`

## 2. 7단계 전 준비 (직접)

1. Deploy Key 등록
   - gitops 레포에 쓰기 권한이 있는 Deploy Key 추가
   - 같은 개인키를 앱 레포 2개(`choimory-dev-front`, `choimory-dev-member-api`)의 Secret `GITOPS_DEPLOY_KEY`로 등록
2. front의 GitHub Environments 변수 등록(stg, prod)
3. 앱 레포를 `stg`에 머지
   - Actions가 이미지를 빌드하고 gitops의 `overlays/stg` 이미지 태그를 실제 값으로 커밋
   - 앱 레포의 미push 커밋도 이때 올라감
4. GHCR 패키지 public 전환
   - 첫 빌드 후 `choimory-dev-member-api`, `choimory-dev-front` 패키지를 public으로 변경
   - 매니페스트에 `imagePullSecrets`가 없어 비공개면 계속 `ImagePullBackOff`

- 주의: 미뤄둔 디렉토리 이름 변경을 먼저 하게 되면, 앱 레포 `deploy.yml`의 경로도 같이 바꾼 뒤 머지해야 함

## 3. 7단계 1차 확인

- stg Pod가 모두 `Running`이 된 뒤 진행. 명령어는 그때 정리
- 확인 항목
  - Pod 정상 여부, 재시작 횟수
  - member-api actuator(8081) `/actuator/health/liveness`, `/actuator/health/readiness` 응답
  - `port-forward`와 `curl`로 front, member-api 헬스체크
  - 로그인 API(회원가입, 로그인, 토큰 재발급) 동작. Redis에 토큰이 저장되는지
  - ArgoCD가 Actions의 태그 커밋을 받아 자동으로 배포하는지
- stg 확인 후 prod도 같은 방식으로 진행(앱 레포 `main` 머지)

## 4. 2차 외부 공개

- 도메인 연결과 Cloudflare Tunnel
  - cloudflared를 클러스터 안에 배포해 `shared-gateway`의 ClusterIP Service로 연결
  - Tunnel 토큰은 SealedSecret으로 관리
  - 도메인: `choimory.dev`, `api.choimory.dev`, `staging.choimory.dev`, `staging-api.choimory.dev`(HTTPRoute에 이미 지정됨)
- 세부 계획은 1차 확인 후 새 문서에서 진행

---

# 중간에 정리할 것

- ArgoCD 화면 로그인 후 `argocd-initial-admin-secret` 삭제(직접)
- 서버의 `~/sealed`, Mac의 `~/Downloads/sealed` 삭제
- `~/sealed-secrets-key-backup.yaml`을 서버 밖 안전한 곳(비밀번호 관리자 등)에 보관. 레포에는 절대 넣지 않음

---

# 미뤄둔 작업

## 디렉토리 이름 변경

- 요청: `projects/choimory-dev`의 `front` → `choimory-dev-front`, `member-api` → `choimory-dev-member-api`, `data` → `database`
- 검토 내용은 [task5.md 디렉토리 이름 변경 검토](task5.md#디렉토리-이름-변경-검토) 참고
- 추천안
  - ApplicationSet 요소에 `dir`을 추가해 디렉토리와 Application 이름을 분리
  - 경로는 `projects/choimory-dev/{{ .dir }}/overlays/{{ .env }}`
  - Application 이름은 `*-front`, `*-member-api`, `*-namespaces` 유지, `*-data`만 `*-database`로 변경
- 같이 바꿀 것
  - ApplicationSet 2개
  - 앱 레포 2개의 `.github/workflows/deploy.yml` 경로(주석 1줄, 명령 2줄씩)
  - 경로가 언급된 매니페스트 주석
- 순서: gitops 변경을 `main`에 먼저 push한 뒤 앱 레포 변경을 머지
- 결정 필요
  - Application 이름 추천안 적용 여부
  - 앱 레포 `deploy.yml` 수정을 같이 진행할지
- 7단계 전 준비(앱 레포 `stg` 머지) 전에 하면 Actions 경로를 한 번에 맞출 수 있음

---

# 아직 검증하지 못한 것

- Mac에서 Tailscale로 `kubectl` 접속(4단계에서 건너뛴 항목)
- GitHub Actions 실행과 ArgoCD 자동 배포 흐름
- member-api의 actuator 응답(8081)
- 확인 완료: PVC를 만들면 `/data/dev/choimory-dev/k8s/<네임스페이스>/<PVC 이름>/` 폴더가 생성됨

---

# 범위를 정해야 하는 것

- etcd와 `/etc/kubernetes/pki` 백업. 데이터가 쌓이기 시작하면 먼저 챙기는 것을 추천
- PostgreSQL 백업(CloudNativePG 백업 설정 여부)
- Uptime Kuma, Homepage 연동
- 배포 실패 알림
- HPA 테스트

---

# 답변 대기 중인 문서 항목

- task4 4번의 clone 경로를 실제 경로(`/data/dev/choimory/repo/choimory-gitops`)로 고칠지
- plan3의 RAM 기준(32GB)을 실제 값(26Gi)으로 정정할지
- `plan1.md:37`의 노션 URL 처리
- 노션 TIL에 서버 호스트명(`choimory-server`)을 그대로 둘지
