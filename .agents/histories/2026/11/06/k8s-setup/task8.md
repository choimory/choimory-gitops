# 목차

- [개요](#개요)
- [네임스페이스 변경 시작](#네임스페이스-변경-시작)
  - [답변](#답변)
  - [1번: Git 수정](#1번-git-수정)
    - [변경 파일](#변경-파일)
    - [바꾸지 않은 것](#바꾸지-않은-것)
    - [검증](#검증)
    - [추가로 확인한 것](#추가로-확인한-것)
  - [0번: 사전 확인과 키 재백업 (서버)](#0번-사전-확인과-키-재백업-서버)
  - [결과에서 볼 것](#결과에서-볼-것)

---

# 개요

- 공용 도구 네임스페이스 `infrastructure`를 `choimory-infra`로 변경하는 작업을 기록함
- 분석과 계획은 [plan10.md](./plan10.md) 참고
- 진행 순서: 0 사전 확인과 키 재백업 → 1 Git 수정 → 2 옛 구성 정리 → 3 gitops 커밋, main 반영 → 4 `choimory-infra`에 재설치 → 5 확인과 정리
- 이전 작업은 [task7.md](./task7.md) 참고. 앱 레포 머지와 Run workflow는 이 작업 후 진행

---

# 네임스페이스 변경 시작

> 시작

## 답변

- 1번 Git 수정은 서버 결과와 무관해 먼저 진행함. 커밋은 하지 않음
- 0번 사전 확인과 키 재백업 명령어를 안내함

## 1번: Git 수정

### 변경 파일

- 20개 파일, 33곳을 `infrastructure` → `choimory-infra`로 변경

| 파일 | 변경 |
|---|---|
| `bootstrap/root.yaml` | Application 네임스페이스, 배포 대상 네임스페이스 |
| `infrastructure/apps/*.yaml` 8개 (argocd, choimory-dev, cloudnative-pg, envoy-gateway, gateway, local-path-provisioner, metrics-server, sealed-secrets) | Application 네임스페이스, 배포 대상 네임스페이스 |
| `infrastructure/apps/namespace.yaml` | Namespace 이름 |
| `infrastructure/gateway/gateway.yaml` | GatewayClass의 EnvoyProxy 참조 네임스페이스 |
| `infrastructure/gateway/kustomization.yaml` | 네임스페이스, 주석 |
| `infrastructure/local-path-provisioner/kustomization.yaml` | 네임스페이스, 주석 |
| `infrastructure/sealed-secrets/values.yaml` | `kubeseal --controller-namespace` 안내 주석 |
| `projects/choimory-dev/argocd/*.yaml` 3개 | AppProject, ApplicationSet 2개의 네임스페이스 |
| `projects/choimory-dev/choimory-dev-{front,member-api}/base/httproute.yaml` | `parentRefs` 네임스페이스 |
| `projects/choimory-dev/namespaces/base/networkpolicy.yaml` | 허용 네임스페이스, 주석, 이름(`allow-same-namespace-and-infra`) |

### 바꾸지 않은 것

- 디렉토리 경로: `infrastructure/apps`, `$values/infrastructure/...` 등 레포 디렉토리 `infrastructure/`는 유지
- 네임스페이스로 쓰인 `infrastructure`는 더 남아 있지 않음

### 검증

- `kubectl kustomize`로 gateway, local-path, 프로젝트 overlay 10개를 빌드했으며 모두 정상
- 공용 리소스는 `choimory-infra`, 앱은 `choimory-dev-{stg,prod}`로 생성됨
- 커밋은 2번 정리가 끝난 뒤 진행

### 추가로 확인한 것

- NetworkPolicy 이름이 `allow-same-namespace-and-infra`로 바뀜
- `namespaces` 앱은 prune이 꺼져 있어 옛 이름(`allow-same-namespace-and-infrastructure`)이 자동으로 지워지지 않음
- 5번 정리 목록에 옛 NetworkPolicy 삭제를 추가함

## 0번: 사전 확인과 키 재백업 (서버)

```bash
# 1) sealed-secrets 키 재백업
kubectl -n infrastructure get secret -l sealedsecrets.bitnami.com/sealed-secrets-key -o yaml > ~/sealed-secrets-key-backup.yaml
chmod 600 ~/sealed-secrets-key-backup.yaml
kubectl -n infrastructure get secret -l sealedsecrets.bitnami.com/sealed-secrets-key
grep -c "tls.key" ~/sealed-secrets-key-backup.yaml

# 2) finalizer 확인
kubectl -n infrastructure get applications.argoproj.io,applicationsets.argoproj.io,appprojects.argoproj.io \
  -o custom-columns='KIND:.kind,NAME:.metadata.name,FINALIZERS:.metadata.finalizers'

# 3) Helm 릴리스, 네임스페이스 안의 Pod와 PVC
helm list -n infrastructure
kubectl -n infrastructure get pods
kubectl -n infrastructure get pvc

# 4) infrastructure를 가리키는 클러스터 단위 리소스 (나중에 비교용)
kubectl get validatingwebhookconfigurations,mutatingwebhookconfigurations
kubectl get apiservices | grep -v Local
kubectl get clusterrolebindings -o custom-columns='NAME:.metadata.name,NS:.subjects[*].namespace' | grep infrastructure
```

## 결과에서 볼 것

- 1) 키 Secret이 1개 이상, `grep -c` 결과가 1 이상. 키 내용은 출력되지 않음
- 2) finalizer가 붙은 항목 여부. 2번 정리 명령어를 여기에 맞춰 확정
- 3) Helm 릴리스가 `argocd` 하나인지, PVC가 없는지 (없어야 지워질 데이터가 없음)
- 4) 5번 정리 때 비교할 기준
