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
- [키 재백업 결과](#키-재백업-결과)
  - [답변](#답변-1)
  - [확인 결과](#확인-결과)
- [push 시점](#push-시점)
  - [답변](#답변-2)
  - [지금 main에 push하면 생기는 일](#지금-main에-push하면-생기는-일)
  - [브랜치별 가능 여부](#브랜치별-가능-여부)
- [finalizer 확인과 2번 정리 전반부](#finalizer-확인과-2번-정리-전반부)
  - [답변](#답변-3)
  - [확인 결과](#확인-결과-1)
  - [남은 0번 4) (서버)](#남은-0번-4-서버)
  - [2번 정리 전반부 (서버)](#2번-정리-전반부-서버)
  - [결과에서 볼 것](#결과에서-볼-것-1)
- [클러스터 단위 리소스 기준 목록](#클러스터-단위-리소스-기준-목록)
  - [답변](#답변-4)
  - [클러스터 단위 리소스 분류](#클러스터-단위-리소스-분류)
  - [APIService를 먼저 지우는 이유](#apiservice를-먼저-지우는-이유)
  - [envoy 웹훅을 바로 지우는 이유](#envoy-웹훅을-바로-지우는-이유)
- [2번 정리 전반부 결과와 후반부](#2번-정리-전반부-결과와-후반부)
  - [답변](#답변-5)
  - [확인 결과](#확인-결과-2)
  - [2번 정리 후반부 (서버)](#2번-정리-후반부-서버)
  - [2-8에서 5분이 지나도 안 끝나면](#2-8에서-5분이-지나도-안-끝나면)
- [2번 정리 완료와 재설치 안내](#2번-정리-완료와-재설치-안내)
  - [답변](#답변-6)
  - [2번 결과](#2번-결과)
  - [3번: gitops 커밋과 main 반영 (직접)](#3번-gitops-커밋과-main-반영-직접)
  - [4번: choimory-infra에 재설치 (서버)](#4번-choimory-infra에-재설치-서버)
  - [4-5 이후](#4-5-이후)
- [재설치 결과와 5번 확인](#재설치-결과와-5번-확인)
  - [답변](#답변-7)
  - [4번 결과](#4번-결과)
  - [5번: 확인 (서버)](#5번-확인-서버)
  - [정상 기준](#정상-기준)
- [5번 확인 결과와 정리](#5번-확인-결과와-정리)
  - [답변](#답변-8)
  - [5번 확인 결과](#5번-확인-결과)
  - [namespaces 앱이 OutOfSync인 이유](#namespaces-앱이-outofsync인-이유)
  - [5번 정리 (서버)](#5번-정리-서버)
  - [마지막 정리 (직접)](#마지막-정리-직접)
- [5번 정리 결과](#5번-정리-결과)
  - [답변](#답변-9)
  - [정리 결과](#정리-결과)
  - [남은 정리 (서버)](#남은-정리-서버)
  - [마지막 정리 (직접)](#마지막-정리-직접-1)
  - [네임스페이스 변경 완료 후](#네임스페이스-변경-완료-후)
- [네임스페이스 변경 완료](#네임스페이스-변경-완료)
  - [답변](#답변-10)
  - [작업 결과](#작업-결과)
  - [남은 직접 작업](#남은-직접-작업)
  - [다음: 7단계로 복귀](#다음-7단계로-복귀)
- [ArgoCD 접속 방법](#argocd-접속-방법)
  - [답변](#답변-11)
  - [1. 서버에서 실행](#1-서버에서-실행)
  - [2. Mac에서 실행](#2-mac에서-실행)
  - [3. Mac 브라우저에서 접속](#3-mac-브라우저에서-접속)
  - [연결 구조](#연결-구조)
  - [접속 후](#접속-후)
- [ArgoCD 접속 처리 보류와 task8 종료](#argocd-접속-처리-보류와-task8-종료)
  - [답변](#답변-12)
  - [보류한 것](#보류한-것)
  - [보류 중 접속 방법](#보류-중-접속-방법)
  - [재개할 때 순서](#재개할-때-순서)
  - [task8 종료](#task8-종료)

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

---

# 키 재백업 결과

> ```
> choimory@choimory-server:/data/storage/media$ kubectl -n infrastructure get secret -l sealedsecrets.bitnami.com/sealed-secrets-key -o yaml > ~/sealed-secrets-key-backup.yaml
> choimory@choimory-server:/data/storage/media$ chmod 600 ~/sealed-secrets-key-backup.yaml
> choimory@choimory-server:/data/storage/media$ kubectl -n infrastructure get secret -l sealedsecrets.bitnami.com/sealed-secrets-key
> NAME                      TYPE                DATA   AGE
> sealed-secrets-key<KEY_ID>   kubernetes.io/tls   2      5d5h
> choimory@choimory-server:/data/storage/media$ grep -c "tls.key" ~/sealed-secrets-key-backup.yaml
> 1
> ```

## 답변

- 키 재백업은 정상
- 0번의 2)~4)는 아직 실행되지 않아 이어서 실행을 요청함. 2번 명령어가 줄바꿈으로 끊길 때를 대비해 한 줄 버전도 안내함

## 확인 결과

| 확인 항목 | 결과 |
|---|---|
| 키 Secret | 1개 (5d5h). 6단계에서 SealedSecret을 만들 때 쓴 키 |
| 백업 파일 | `tls.key` 포함 확인 |

- 백업 파일 `~/sealed-secrets-key-backup.yaml`은 작업이 끝날 때까지 유지하고, 끝나면 서버 밖 안전한 곳으로 옮김

---

# push 시점

> 지금 작업했던 코드는 아직 푸쉬 안했는데 푸쉬 해놓고 해야하나?

## 답변

- `main`에는 아직 push하면 안 됨. 옛 ArgoCD가 살아 있는 지금 `main`이 바뀌면 prune 문제가 생길 수 있음

## 지금 main에 push하면 생기는 일

- root 앱이 바뀐 Git을 읽고, 옛 `infrastructure`의 Application들을 Git에 없는 것으로 보고 지우려 함
- 새 Application은 `choimory-infra`에 만들어지지만, 옛 ArgoCD는 그 네임스페이스를 보지 않아 아무도 처리하지 않음
- 공용 도구가 반쯤 지워진 어중간한 상태가 될 수 있음

## 브랜치별 가능 여부

| 작업 | 지금 해도 되는지 |
|---|---|
| 로컬 커밋 | 됨 |
| `dev` push | 됨. ArgoCD는 `main`만 봄 (`targetRevision: main`) |
| `stg` push | 됨. gitops의 `stg`도 ArgoCD가 보지 않음 |
| `main` 머지, push | 안 됨. 2번 정리가 끝난 뒤 (3번 단계) |

---

# finalizer 확인과 2번 정리 전반부

> ```
> choimory@choimory-server:/data/storage/media$ kubectl -n infrastructure get applications.argoproj.io,applicationsets.argoproj.io,appprojects.argoproj.io \
>   -o custom-columns='KIND:.kind,NAME:.metadata.name,FINALIZERS:.metadata.finalizers'
> KIND             NAME                           FINALIZERS
> Application      argocd                         <none>
> Application      choimory-dev                   <none>
> Application      choimory-dev-prod-front        [resources-finalizer.argocd.argoproj.io]
> Application      choimory-dev-prod-member-api   [resources-finalizer.argocd.argoproj.io]
> Application      choimory-dev-prod-namespaces   <none>
> Application      choimory-dev-prod-postgres     <none>
> Application      choimory-dev-prod-redis        <none>
> Application      choimory-dev-stg-front         [resources-finalizer.argocd.argoproj.io]
> Application      choimory-dev-stg-member-api    [resources-finalizer.argocd.argoproj.io]
> Application      choimory-dev-stg-namespaces    <none>
> Application      choimory-dev-stg-postgres      <none>
> Application      choimory-dev-stg-redis         <none>
> Application      cloudnative-pg                 <none>
> Application      envoy-gateway                  <none>
> Application      gateway                        <none>
> Application      local-path-provisioner         <none>
> Application      metrics-server                 <none>
> Application      root                           <none>
> Application      sealed-secrets                 <none>
> ApplicationSet   choimory-dev-apps              <none>
> ApplicationSet   choimory-dev-protected         <none>
> AppProject       choimory-dev                   <none>
> AppProject       default                        <none>
> choimory@choimory-server:/data/storage/media$ helm list -n infrastructure
> NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
> argocd  infrastructure  1               2026-11-10 18:22:35.694191768 +0900 KST deployed        argo-cd-10.17.0 v3.6.1     
> choimory@choimory-server:/data/storage/media$ kubectl -n infrastructure get pods
> NAME                                                           READY   STATUS    RESTARTS   AGE
> argocd-application-controller-0                                1/1     Running   0          5d5h
> argocd-applicationset-controller-797985f4b7-nm5hl              1/1     Running   0          5d5h
> argocd-redis-5fc69b46c8-qbd7h                                  1/1     Running   0          5d5h
> argocd-repo-server-7b86c89c58-9tklm                            1/1     Running   0          5d5h
> argocd-server-6fb5747584-k2mwk                                 1/1     Running   0          5d5h
> cloudnative-pg-55d68ccd6c-cs9ft                                1/1     Running   0          5d5h
> envoy-gateway-86869b759c-5sfrr                                 1/1     Running   0          5d4h
> envoy-infrastructure-shared-gateway-02d9540f-74bdf9fb6-m5rcm   2/2     Running   0          5d5h
> local-path-provisioner-54745b98cf-6szg2                        1/1     Running   0          5d5h
> metrics-server-55c6b8755f-nwdgv                                1/1     Running   0          5d5h
> sealed-secrets-controller-5b56d98858-5g8xw                     1/1     Running   0          5d5h
> choimory@choimory-server:/data/storage/media$ kubectl -n infrastructure get pvc
> No resources found in infrastructure namespace.
> ```

## 답변

- 예상대로 front와 member-api의 Application 4개에 finalizer가 있음. 먼저 떼야 안전하게 지울 수 있음
- 0번 4)는 아직 실행되지 않아 다시 안내함

## 확인 결과

| 항목 | 결과 | 판단 |
|---|---|---|
| finalizer | `choimory-dev-{stg,prod}-{front,member-api}` 4개에 `resources-finalizer` | 그대로 지우면 Deployment, Service, HTTPRoute, member-api SealedSecret까지 삭제됨. 반드시 먼저 제거 |
| | 나머지 Application, ApplicationSet, AppProject는 없음 | postgres, redis, namespaces 앱은 지워도 리소스가 남음 |
| Helm 릴리스 | `argocd` 1개 (argo-cd-10.17.0) | 계획대로 `helm uninstall argocd` |
| Pod | ArgoCD 5개와 공용 도구 6개 | 네임스페이스 삭제로 같이 정리됨 |
| PVC | 없음 | 지워질 데이터 없음 |

## 남은 0번 4) (서버)

- 나중에 남은 것을 정리할 때 비교할 기준. 2번 전에 먼저 실행

```bash
kubectl get validatingwebhookconfigurations,mutatingwebhookconfigurations
kubectl get apiservices | grep -v Local
kubectl get clusterrolebindings -o custom-columns='NAME:.metadata.name,NS:.subjects[*].namespace' | grep infrastructure
```

## 2번 정리 전반부 (서버)

- 아무것도 지우지 않는 단계와 ArgoCD 껍데기만 지우는 단계

```bash
# 2-1) ArgoCD 컨트롤러 정지. 정리하는 동안 다시 만들거나 지우지 못하게 함
kubectl -n infrastructure scale statefulset argocd-application-controller --replicas=0
kubectl -n infrastructure scale deployment argocd-applicationset-controller --replicas=0
kubectl -n infrastructure get pods | grep -E 'application-controller|applicationset-controller'
```

- 마지막 결과가 비어 있어야 함. `Terminating`이면 잠시 후 다시 확인

```bash
# 2-2) finalizer 제거 (4개)
for app in choimory-dev-stg-front choimory-dev-stg-member-api choimory-dev-prod-front choimory-dev-prod-member-api; do
  kubectl -n infrastructure patch application "$app" --type json -p '[{"op":"remove","path":"/metadata/finalizers"}]'
done
kubectl -n infrastructure get applications -o custom-columns='NAME:.metadata.name,FINALIZERS:.metadata.finalizers' | grep -v '<none>'
```

- 마지막 결과는 헤더(`NAME FINALIZERS`)만 나와야 함

```bash
# 2-3) ApplicationSet, Application, AppProject 삭제. 실제 리소스는 남음
kubectl -n infrastructure delete applicationsets.argoproj.io --all
kubectl -n infrastructure delete applications.argoproj.io --all
kubectl -n infrastructure delete appprojects.argoproj.io --all

# 2-4) 프로젝트 리소스가 그대로인지 확인
kubectl -n choimory-dev-stg get pods,pvc,sealedsecret,httproute
kubectl -n choimory-dev-prod get pods,pvc,sealedsecret,httproute
```

## 결과에서 볼 것

- 2-4) postgres, redis Pod의 AGE 그대로
- front, member-api Pod도 남아 있음 (`ImagePullBackOff` 그대로)
- PVC 4개, SealedSecret 4개, HTTPRoute 2개씩 그대로
- 확인 후 2번 후반부(`helm uninstall`, `infrastructure` 네임스페이스 삭제) 진행

---

# 클러스터 단위 리소스 기준 목록

> ```
> choimory@choimory-server:/data/storage/media$ kubectl get validatingwebhookconfigurations,mutatingwebhookconfigurations
> kubectl get apiservices | grep -v Local
> kubectl get clusterrolebindings -o custom-columns='NAME:.metadata.name,NS:.subjects[*].namespace' | grep infrastructure
> NAME                                                                                                WEBHOOKS   AGE
> validatingwebhookconfiguration.admissionregistration.k8s.io/cnpg-validating-webhook-configuration   5          5d5h
>
> NAME                                                                                                       WEBHOOKS   AGE
> mutatingwebhookconfiguration.admissionregistration.k8s.io/cnpg-mutating-webhook-configuration              4          5d5h
> mutatingwebhookconfiguration.admissionregistration.k8s.io/envoy-gateway-topology-injector.infrastructure   1          5d5h
> NAME                                   SERVICE                         AVAILABLE   AGE
> v1beta1.metrics.k8s.io                 infrastructure/metrics-server   True        5d5h
> argocd-application-controller                                   infrastructure
> argocd-server                                                   infrastructure
> cloudnative-pg                                                  infrastructure
> envoy-gateway-gateway-helm-certgen:infrastructure               infrastructure
> envoy-gateway-gateway-helm-envoy-gateway-rolebinding            infrastructure
> local-path-provisioner-bind                                     infrastructure
> metrics-server:system:auth-delegator                            infrastructure
> sealed-secrets-controller                                       infrastructure
> system:metrics-server                                           infrastructure
> ```

## 답변

- 대부분은 재설치 때 같은 이름으로 덮어써지지만, 이름에 `infrastructure`가 들어간 2개는 새 설치와 이름이 달라 직접 지워야 함
- 네임스페이스 삭제 전에 metrics APIService를 먼저 지워야 함

## 클러스터 단위 리소스 분류

| 리소스 | 재설치 후 | 처리 |
|---|---|---|
| `cnpg-validating-webhook-configuration`, `cnpg-mutating-webhook-configuration` | 같은 이름으로 덮어써짐 | 없음 |
| APIService `v1beta1.metrics.k8s.io` | 같은 이름으로 덮어써짐 | 네임스페이스 삭제 전에 지움 |
| ClusterRoleBinding 7개 (argocd, cloudnative-pg, local-path, metrics-server, sealed-secrets 등) | 같은 이름, 대상만 `choimory-infra`로 덮어써짐 | 없음 |
| mutating webhook `envoy-gateway-topology-injector.infrastructure` | 새 이름(`...choimory-infra`)으로 따로 생김 | 2번 후반부에서 삭제 |
| ClusterRoleBinding `envoy-gateway-gateway-helm-certgen:infrastructure` | 새 이름으로 따로 생김 | 5번에서 삭제 |

## APIService를 먼저 지우는 이유

- `v1beta1.metrics.k8s.io`는 `infrastructure/metrics-server`로 요청을 넘기는 설정
- 네임스페이스를 지우는 도중 metrics-server가 먼저 사라지면, Kubernetes가 이 API를 조회하지 못해 네임스페이스 삭제가 `Terminating`에서 멈출 수 있음 (알려진 문제)
- 미리 지워두면 문제가 생기지 않으며, 재설치 때 metrics-server 앱이 다시 만듦
- 지운 동안에는 `kubectl top`만 동작하지 않음

## envoy 웹훅을 바로 지우는 이유

- Pod 생성 시 끼어드는 웹훅인데, 네임스페이스가 지워지면 연결할 곳이 없어짐
- 남겨둘 이유가 없어 네임스페이스 삭제 직후 바로 지움

---

# 2번 정리 전반부 결과와 후반부

> ```
> choimory@choimory-server:/data/storage/media$ 
> kubectl -n infrastructure scale statefulset argocd-application-controller --replicas=0
> kubectl -n infrastructure scale deployment argocd-applicationset-controller --replicas=0
> kubectl -n infrastructure get pods | grep -E 'application-controller|applicationset-controller'
> statefulset.apps/argocd-application-controller scaled
> deployment.apps/argocd-applicationset-controller scaled
> argocd-application-controller-0                                1/1     Terminating   0          5d5h
> argocd-applicationset-controller-797985f4b7-nm5hl              1/1     Terminating   0          5d5h
> choimory@choimory-server:/data/storage/media$ 
> for app in choimory-dev-stg-front choimory-dev-stg-member-api choimory-dev-prod-front choimory-dev-prod-member-api; do
>   kubectl -n infrastructure patch application "$app" --type json -p '[{"op":"remove","path":"/metadata/finalizers"}]'
> done
> kubectl -n infrastructure get applications -o custom-columns='NAME:.metadata.name,FINALIZERS:.metadata.finalizers' | grep -v '<none>'
> application.argoproj.io/choimory-dev-stg-front patched
> application.argoproj.io/choimory-dev-stg-member-api patched
> application.argoproj.io/choimory-dev-prod-front patched
> application.argoproj.io/choimory-dev-prod-member-api patched
> NAME                           FINALIZERS
> choimory@choimory-server:/data/storage/media$ 
> kubectl -n infrastructure delete applicationsets.argoproj.io --all
> kubectl -n infrastructure delete applications.argoproj.io --all
> kubectl -n infrastructure delete appprojects.argoproj.io --all
> applicationset.argoproj.io "choimory-dev-apps" deleted from infrastructure namespace
> applicationset.argoproj.io "choimory-dev-protected" deleted from infrastructure namespace
> application.argoproj.io "argocd" deleted from infrastructure namespace
> application.argoproj.io "choimory-dev" deleted from infrastructure namespace
> application.argoproj.io "cloudnative-pg" deleted from infrastructure namespace
> application.argoproj.io "envoy-gateway" deleted from infrastructure namespace
> application.argoproj.io "gateway" deleted from infrastructure namespace
> application.argoproj.io "local-path-provisioner" deleted from infrastructure namespace
> application.argoproj.io "metrics-server" deleted from infrastructure namespace
> application.argoproj.io "root" deleted from infrastructure namespace
> application.argoproj.io "sealed-secrets" deleted from infrastructure namespace
> appproject.argoproj.io "choimory-dev" deleted from infrastructure namespace
> appproject.argoproj.io "default" deleted from infrastructure namespace
> choimory@choimory-server:/data/storage/media$ 
> kubectl -n choimory-dev-stg get pods,pvc,sealedsecret,httproute
> kubectl -n choimory-dev-prod get pods,pvc,sealedsecret,httproute
> NAME                                           READY   STATUS             RESTARTS   AGE
> pod/choimory-dev-front-85469c8fb5-klvlp        0/1     ImagePullBackOff   0          5d2h
> pod/choimory-dev-member-api-69cd5894dd-vcmbc   0/1     ImagePullBackOff   0          5d2h
> pod/choimory-dev-postgres-1                    1/1     Running            0          5d2h
> pod/choimory-dev-redis-0                       1/1     Running            0          5d2h
>
> NAME                                              STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS              VOLUMEATTRIBUTESCLASS   AGE
> persistentvolumeclaim/choimory-dev-postgres-1     Bound    pvc-<STG_POSTGRES_PV_ID>   5Gi        RWO            local-path-choimory-dev   <unset>                 5d2h
> persistentvolumeclaim/data-choimory-dev-redis-0   Bound    pvc-<STG_REDIS_PV_ID>   1Gi        RWO            local-path-choimory-dev   <unset>                 5d2h
>
> NAME                                                      STATUS   SYNCED   AGE
> sealedsecret.bitnami.com/choimory-dev-member-api-secret            True     4h37m
> sealedsecret.bitnami.com/choimory-dev-redis-secret                 True     4h38m
>
> NAME                                                          HOSTNAMES                             AGE
> httproute.gateway.networking.k8s.io/choimory-dev-front        ["staging.choimory.dev"]              5d2h
> httproute.gateway.networking.k8s.io/choimory-dev-member-api   ["staging-member-api.choimory.dev"]   5d2h
> NAME                                          READY   STATUS             RESTARTS   AGE
> pod/choimory-dev-front-56889d449f-x6z6t       0/1     ImagePullBackOff   0          5d2h
> pod/choimory-dev-member-api-dd677d978-pqfk6   0/1     ImagePullBackOff   0          5d2h
> pod/choimory-dev-postgres-1                   1/1     Running            0          5d2h
> pod/choimory-dev-redis-0                      1/1     Running            0          5d2h
>
> NAME                                              STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS              VOLUMEATTRIBUTESCLASS   AGE
> persistentvolumeclaim/choimory-dev-postgres-1     Bound    pvc-<PROD_POSTGRES_PV_ID>   10Gi       RWO            local-path-choimory-dev   <unset>                 5d2h
> persistentvolumeclaim/data-choimory-dev-redis-0   Bound    pvc-<PROD_REDIS_PV_ID>   1Gi        RWO            local-path-choimory-dev   <unset>                 5d2h
>
> NAME                                                      STATUS   SYNCED   AGE
> sealedsecret.bitnami.com/choimory-dev-member-api-secret            True     4h40m
> sealedsecret.bitnami.com/choimory-dev-redis-secret                 True     4h38m
>
> NAME                                                          HOSTNAMES                     AGE
> httproute.gateway.networking.k8s.io/choimory-dev-front        ["choimory.dev"]              5d2h
> httproute.gateway.networking.k8s.io/choimory-dev-member-api   ["member-api.choimory.dev"]   5d2h
> ```

## 답변

- 2번 전반부가 정상으로 끝났으며 프로젝트 리소스는 하나도 지워지지 않음
- finalizer를 먼저 떼어, Application을 지워도 front와 member-api의 Deployment, Service, HTTPRoute, SealedSecret이 남음

## 확인 결과

| 항목 | 결과 |
|---|---|
| ArgoCD 컨트롤러 | 2개 모두 `Terminating`으로 정지 중 |
| finalizer | 4개 제거, 헤더만 출력됨 |
| ApplicationSet, Application, AppProject | 2개, 17개, 2개 모두 삭제 |
| postgres, redis Pod | stg, prod 모두 `Running`. AGE 5d2h 그대로, RESTARTS 0 |
| front, member-api Pod | 남아 있음 (`ImagePullBackOff` 그대로) |
| PVC | 4개 모두 `Bound`. 볼륨 이름이 이전과 같음 |
| SealedSecret | 4개 모두 `SYNCED True` |
| HTTPRoute | stg, prod 각각 2개. hostname 그대로 |

## 2번 정리 후반부 (서버)

```bash
# 2-5) Gateway를 먼저 삭제. Envoy Gateway 컨트롤러가 살아 있을 때 지워야 정리가 끝까지 됨
kubectl -n infrastructure delete gateway shared-gateway --timeout=2m
kubectl -n infrastructure get pods | grep envoy-infrastructure
```

- 마지막 결과가 비어야 함 (Envoy 프록시 Pod 삭제됨). `Terminating`이면 잠시 후 다시 확인
- 컨트롤러가 없을 때 Gateway를 지우면 finalizer가 남아 네임스페이스 삭제가 멈출 수 있어 먼저 지움

```bash
# 2-6) metrics APIService 삭제. 네임스페이스 삭제가 Terminating에서 멈추는 것을 막음
kubectl delete apiservice v1beta1.metrics.k8s.io

# 2-7) ArgoCD 삭제
helm uninstall argocd -n infrastructure
kubectl get crd | grep argoproj
```

- `grep argoproj` 결과는 남아 있어도, 없어도 됨. 재설치 때 다시 만들어짐

```bash
# 2-8) infrastructure 네임스페이스 삭제
kubectl delete namespace infrastructure --timeout=5m
kubectl get namespace infrastructure
```

- 마지막 결과가 `NotFound`면 정상

```bash
# 2-9) 이름에 infrastructure가 들어간 envoy 웹훅 삭제
kubectl delete mutatingwebhookconfiguration envoy-gateway-topology-injector.infrastructure

# 2-10) 프로젝트 리소스가 그대로인지 다시 확인
kubectl -n choimory-dev-stg get pods,pvc
kubectl -n choimory-dev-prod get pods,pvc
```

## 2-8에서 5분이 지나도 안 끝나면

```bash
kubectl get namespace infrastructure -o jsonpath='{.status.conditions}{"\n"}'
kubectl api-resources --verbs=list --namespaced -o name | xargs -n1 kubectl get -n infrastructure --ignore-not-found --show-kind 2>/dev/null
```

- 남은 리소스와 원인을 확인. 이상이 없으면 3번(gitops 커밋, `main` 반영)으로 넘어감

---

# 2번 정리 완료와 재설치 안내

> ```
> choimory@choimory-server:/data/storage/media$ 
> kubectl -n infrastructure delete gateway shared-gateway --timeout=2m
> kubectl -n infrastructure get pods | grep envoy-infrastructure
> gateway.gateway.networking.k8s.io "shared-gateway" deleted from infrastructure namespace
> envoy-infrastructure-shared-gateway-02d9540f-74bdf9fb6-m5rcm   2/2     Terminating   0          5d5h
> choimory@choimory-server:/data/storage/media$ 
> kubectl delete apiservice v1beta1.metrics.k8s.io
> apiservice.apiregistration.k8s.io "v1beta1.metrics.k8s.io" deleted
> choimory@choimory-server:/data/storage/media$ 
> helm uninstall argocd -n infrastructure
> kubectl get crd | grep argoproj
> These resources were kept due to the resource policy:
> [CustomResourceDefinition] applications.argoproj.io
> [CustomResourceDefinition] applicationsets.argoproj.io
> [CustomResourceDefinition] appprojects.argoproj.io
>
> release "argocd" uninstalled
> applications.argoproj.io                              2026-11-10T09:23:13Z
> applicationsets.argoproj.io                           2026-11-10T09:23:13Z
> appprojects.argoproj.io                               2026-11-10T09:23:13Z
> choimory@choimory-server:/data/storage/media$ 
> kubectl delete namespace infrastructure --timeout=5m
> kubectl get namespace infrastructure
> namespace "infrastructure" deleted
> Error from server (NotFound): namespaces "infrastructure" not found
> choimory@choimory-server:/data/storage/media$ 
> kubectl delete mutatingwebhookconfiguration envoy-gateway-topology-injector.infrastructure
> mutatingwebhookconfiguration.admissionregistration.k8s.io "envoy-gateway-topology-injector.infrastructure" deleted
> choimory@choimory-server:/data/storage/media$ 
> kubectl -n choimory-dev-stg get pods,pvc
> kubectl -n choimory-dev-prod get pods,pvc
> NAME                                           READY   STATUS             RESTARTS   AGE
> pod/choimory-dev-front-85469c8fb5-klvlp        0/1     ImagePullBackOff   0          5d2h
> pod/choimory-dev-member-api-69cd5894dd-vcmbc   0/1     ImagePullBackOff   0          5d2h
> pod/choimory-dev-postgres-1                    1/1     Running            0          5d2h
> pod/choimory-dev-redis-0                       1/1     Running            0          5d2h
>
> NAME                                              STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS              VOLUMEATTRIBUTESCLASS   AGE
> persistentvolumeclaim/choimory-dev-postgres-1     Bound    pvc-<STG_POSTGRES_PV_ID>   5Gi        RWO            local-path-choimory-dev   <unset>                 5d2h
> persistentvolumeclaim/data-choimory-dev-redis-0   Bound    pvc-<STG_REDIS_PV_ID>   1Gi        RWO            local-path-choimory-dev   <unset>                 5d2h
> NAME                                          READY   STATUS             RESTARTS   AGE
> pod/choimory-dev-front-56889d449f-x6z6t       0/1     ImagePullBackOff   0          5d2h
> pod/choimory-dev-member-api-dd677d978-pqfk6   0/1     ImagePullBackOff   0          5d2h
> pod/choimory-dev-postgres-1                   1/1     Running            0          5d2h
> pod/choimory-dev-redis-0                      1/1     Running            0          5d2h
>
> NAME                                              STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS              VOLUMEATTRIBUTESCLASS   AGE
> persistentvolumeclaim/choimory-dev-postgres-1     Bound    pvc-<PROD_POSTGRES_PV_ID>   10Gi       RWO            local-path-choimory-dev   <unset>                 5d2h
> persistentvolumeclaim/data-choimory-dev-redis-0   Bound    pvc-<PROD_REDIS_PV_ID>   1Gi        RWO            local-path-choimory-dev   <unset>                 5d2h
> choimory@choimory-server:/data/storage/media$ 
> kubectl get namespace infrastructure -o jsonpath='{.status.conditions}{"\n"}'
> kubectl api-resources --verbs=list --namespaced -o name | xargs -n1 kubectl get -n infrastructure --ignore-not-found --show-kind 2>/dev/null
> Error from server (NotFound): namespaces "infrastructure" not found
> ```

## 답변

- 2번 정리가 모두 끝남. 로컬 커밋도 확인했으므로 3번 push 후 바로 4번 재설치로 넘어감

## 2번 결과

| 항목 | 결과 |
|---|---|
| Gateway, Envoy 프록시 | 삭제됨 |
| metrics APIService | 삭제됨 |
| ArgoCD | Helm 릴리스 삭제. CRD 3개는 차트 설정(resource policy)상 남음 |
| `infrastructure` 네임스페이스 | 삭제 완료 (`NotFound`). 멈추지 않고 바로 끝남 |
| envoy 웹훅 | 삭제됨 |
| 프로젝트 리소스 | postgres, redis Pod의 AGE와 RESTARTS 그대로, PVC 4개 `Bound` |

## 3번: gitops 커밋과 main 반영 (직접)

- 로컬 `dev`에 커밋 `ce323ee`(refactor: infra 네임스페이스 변경)가 있으며 push 전
- 매니페스트 20개 파일이 모두 포함되어 있고, 네임스페이스로 쓰인 `infrastructure`는 남아 있지 않음
- task8 최근 수정분은 커밋 전. 같이 커밋하거나 나중에 커밋
- 옛 ArgoCD가 없으므로 `dev` push 후 release로 `main`까지 반영

## 4번: choimory-infra에 재설치 (서버)

- `main` 반영 후 순서대로 실행

```bash
# 4-1) 서버의 gitops 클론 갱신
cd /data/dev/choimory/repo/choimory-gitops
git pull
grep -n "namespace:" bootstrap/root.yaml
```

- `namespace: choimory-infra`가 두 줄 나와야 함

```bash
# 4-2) 남은 ArgoCD CRD 삭제
kubectl get applications.argoproj.io,applicationsets.argoproj.io,appprojects.argoproj.io -A
kubectl delete crd applications.argoproj.io applicationsets.argoproj.io appprojects.argoproj.io
```

- 첫 명령어 결과가 `No resources found`일 때만 삭제
- 남은 CRD에는 `infrastructure` 네임스페이스 Helm 릴리스의 소유 표시가 있어, 그대로 두면 새 네임스페이스로 `helm install` 할 때 소유권 오류로 실패함
- 안에 든 리소스가 없어 지워도 안전하며, 설치 때 다시 만들어짐

```bash
# 4-3) 네임스페이스 생성과 sealed-secrets 키 복원. root 등록보다 반드시 먼저
kubectl create namespace choimory-infra
sed -e 's/namespace: infrastructure/namespace: choimory-infra/' \
    -e '/resourceVersion:/d' -e '/uid:/d' -e '/creationTimestamp:/d' \
    ~/sealed-secrets-key-backup.yaml > ~/sealed-secrets-key-restore.yaml
chmod 600 ~/sealed-secrets-key-restore.yaml
kubectl apply -f ~/sealed-secrets-key-restore.yaml
kubectl -n choimory-infra get secret -l sealedsecrets.bitnami.com/sealed-secrets-key
rm ~/sealed-secrets-key-restore.yaml
```

- `choimory-infra`에 기존 키 Secret이 보여야 함
- 원본 백업 파일 `~/sealed-secrets-key-backup.yaml`은 유지

```bash
# 4-4) ArgoCD 설치
helm repo update argo
helm install argocd argo/argo-cd --version 10.17.0 -n choimory-infra -f infrastructure/argocd/values.yaml
kubectl -n choimory-infra rollout status deploy/argocd-server --timeout=5m
kubectl -n choimory-infra get pods
```

- ArgoCD Pod 5개가 `Running`이면 다음으로 진행

```bash
# 4-5) root 등록
kubectl apply -f bootstrap/root.yaml
```

## 4-5 이후

- ArgoCD가 공용 도구를 차례로 설치함. 5~10분 뒤 아래 결과 확인

```bash
kubectl -n choimory-infra get applications
kubectl -n choimory-infra get pods
```

- 중간에 막히는 단계가 있으면 그 단계까지의 출력으로 확인

---

# 재설치 결과와 5번 확인

> ```
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ grep -n "namespace:" bootstrap/root.yaml
> 7:  namespace: choimory-infra
> 17:    namespace: choimory-infra
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ 
> kubectl get applications.argoproj.io,applicationsets.argoproj.io,appprojects.argoproj.io -A
> kubectl delete crd applications.argoproj.io applicationsets.argoproj.io appprojects.argoproj.io
> No resources found
> customresourcedefinition.apiextensions.k8s.io "applications.argoproj.io" deleted
> customresourcedefinition.apiextensions.k8s.io "applicationsets.argoproj.io" deleted
> customresourcedefinition.apiextensions.k8s.io "appprojects.argoproj.io" deleted
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ 
> kubectl create namespace choimory-infra
> sed -e 's/namespace: infrastructure/namespace: choimory-infra/' \
>     -e '/resourceVersion:/d' -e '/uid:/d' -e '/creationTimestamp:/d' \
>     ~/sealed-secrets-key-backup.yaml > ~/sealed-secrets-key-restore.yaml
> chmod 600 ~/sealed-secrets-key-restore.yaml
> kubectl apply -f ~/sealed-secrets-key-restore.yaml
> kubectl -n choimory-infra get secret -l sealedsecrets.bitnami.com/sealed-secrets-key
> rm ~/sealed-secrets-key-restore.yaml
> namespace/choimory-infra created
> secret/sealed-secrets-key<KEY_ID> created
> NAME                      TYPE                DATA   AGE
> sealed-secrets-key<KEY_ID>   kubernetes.io/tls   2      0s
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ 
> helm repo update argo
> helm install argocd argo/argo-cd --version 10.17.0 -n choimory-infra -f infrastructure/argocd/values.yaml
> kubectl -n choimory-infra rollout status deploy/argocd-server --timeout=5m
> kubectl -n choimory-infra get pods
> Hang tight while we grab the latest from your chart repositories...
> ...Successfully got an update from the "argo" chart repository
> Update Complete. ⎈Happy Helming!⎈
> NAME: argocd
> LAST DEPLOYED: Mon Nov 16 00:17:34 2026
> NAMESPACE: choimory-infra
> STATUS: deployed
> REVISION: 1
> DESCRIPTION: Install complete
> TEST SUITE: None
> NOTES:
> (ArgoCD 접속 안내 생략)
> Waiting for deployment "argocd-server" rollout to finish: 0 of 1 updated replicas are available...
> deployment "argocd-server" successfully rolled out
> NAME                                                READY   STATUS      RESTARTS   AGE
> argocd-application-controller-0                     1/1     Running     0          12s
> argocd-applicationset-controller-797985f4b7-8496p   1/1     Running     0          12s
> argocd-redis-5fc69b46c8-frrck                       1/1     Running     0          12s
> argocd-redis-secret-init-9klqp                      0/1     Completed   0          15s
> argocd-repo-server-7b86c89c58-qk9r7                 0/1     Running     0          12s
> argocd-server-6fb5747584-2q8z2                      1/1     Running     0          12s
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ 
> kubectl apply -f bootstrap/root.yaml
> application.argoproj.io/root created
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ 
> kubectl -n choimory-infra get applications
> kubectl -n choimory-infra get pods
> NAME   SYNC STATUS   HEALTH STATUS
> root   OutOfSync     Healthy
> NAME                                                READY   STATUS      RESTARTS   AGE
> argocd-application-controller-0                     1/1     Running     0          18s
> argocd-applicationset-controller-797985f4b7-8496p   1/1     Running     0          18s
> argocd-redis-5fc69b46c8-frrck                       1/1     Running     0          18s
> argocd-redis-secret-init-9klqp                      0/1     Completed   0          21s
> argocd-repo-server-7b86c89c58-qk9r7                 1/1     Running     0          18s
> argocd-server-6fb5747584-2q8z2                      1/1     Running     0          18s
> ```

## 답변

- 4-1~4-5가 모두 정상으로 끝남
- 마지막 확인은 root 등록 18초 뒤라 아직 동기화 전이며, 5~10분 뒤 다시 확인

## 4번 결과

| 단계 | 결과 |
|---|---|
| 4-1 gitops 클론 | `namespace: choimory-infra` 두 줄 확인 |
| 4-2 ArgoCD CRD | 남은 리소스가 없는 것을 확인 후 3개 삭제 |
| 4-3 키 복원 | `choimory-infra`에 기존 키 Secret 생성. 작업 전과 같은 이름이라 같은 키 |
| 4-4 ArgoCD 설치 | Helm 릴리스 `deployed`, Pod 5개 `Running` (`redis-secret-init`은 1회성 작업이라 `Completed`가 정상) |
| 4-5 root 등록 | `root` 생성. `OutOfSync`는 첫 동기화 전이라 정상 |

## 5번: 확인 (서버)

```bash
# 공용 도구와 프로젝트 앱 상태
kubectl -n choimory-infra get applications
kubectl -n choimory-infra get pods

# 기존 키로 SealedSecret이 풀리는지
kubectl get sealedsecrets -A

# 프로젝트 리소스와 Gateway 연결
kubectl -n choimory-dev-stg get pods,pvc,httproute
kubectl -n choimory-dev-prod get pods,pvc,httproute
kubectl -n choimory-infra get gateway

# 다시 만들어져야 하는 클러스터 단위 리소스
kubectl get apiservice v1beta1.metrics.k8s.io
kubectl get mutatingwebhookconfigurations
```

## 정상 기준

| 항목 | 기대 결과 |
|---|---|
| Application | 공용 도구 9개와 `choimory-dev`, 프로젝트 앱 10개가 `Synced`. front, member-api 4개만 `Degraded` (이미지 없음) |
| SealedSecret | 4개 모두 `SYNCED True`. 기존 키로 풀렸다는 뜻 |
| postgres, redis Pod | AGE 5d 이상 그대로, RESTARTS 0 |
| Gateway | `shared-gateway`가 `PROGRAMMED True` |
| APIService | `choimory-infra/metrics-server`, `AVAILABLE True` |
| 웹훅 | cnpg 2개, `envoy-gateway-topology-injector.choimory-infra` |

- 10분이 지나도 앱이 다 생기지 않거나 `gateway`가 `Degraded`면, task5의 GatewayClass 문제가 다시 생겼을 수 있음. 같은 방법으로 처리
- ArgoCD 화면 로그인은 확인 후 정리 단계에서 안내

---

# 5번 확인 결과와 정리

> ```
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ 
> kubectl -n choimory-infra get applications
> kubectl -n choimory-infra get pods
> NAME                           SYNC STATUS   HEALTH STATUS
> argocd                         Synced        Healthy
> choimory-dev                   Synced        Healthy
> choimory-dev-prod-front        Synced        Degraded
> choimory-dev-prod-member-api   Synced        Degraded
> choimory-dev-prod-namespaces   OutOfSync     Healthy
> choimory-dev-prod-postgres     Synced        Healthy
> choimory-dev-prod-redis        Synced        Healthy
> choimory-dev-stg-front         Synced        Degraded
> choimory-dev-stg-member-api    Synced        Degraded
> choimory-dev-stg-namespaces    OutOfSync     Healthy
> choimory-dev-stg-postgres      Synced        Healthy
> choimory-dev-stg-redis         Synced        Healthy
> cloudnative-pg                 Synced        Healthy
> envoy-gateway                  Synced        Healthy
> gateway                        Synced        Healthy
> local-path-provisioner         Synced        Healthy
> metrics-server                 Synced        Healthy
> root                           Synced        Healthy
> sealed-secrets                 Synced        Healthy
> NAME                                                           READY   STATUS    RESTARTS   AGE
> argocd-application-controller-0                                1/1     Running   0          2m19s
> argocd-applicationset-controller-797985f4b7-8496p              1/1     Running   0          2m19s
> argocd-redis-5fc69b46c8-frrck                                  1/1     Running   0          2m19s
> argocd-repo-server-7b86c89c58-qk9r7                            1/1     Running   0          2m19s
> argocd-server-6fb5747584-2q8z2                                 1/1     Running   0          2m19s
> cloudnative-pg-75f945f846-lczns                                1/1     Running   0          73s
> envoy-choimory-infra-shared-gateway-ab26edc9-f695c9848-r887q   2/2     Running   0          53s
> envoy-gateway-65c678d44b-hwz22                                 1/1     Running   0          54s
> local-path-provisioner-54745b98cf-24r7m                        1/1     Running   0          106s
> metrics-server-55c6b8755f-f26l9                                1/1     Running   0          99s
> sealed-secrets-controller-5b56d98858-wbm4r                     1/1     Running   0          108s
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ 
> kubectl get sealedsecrets -A
> NAMESPACE           NAME                             STATUS   SYNCED   AGE
> choimory-dev-prod   choimory-dev-member-api-secret            True     4h59m
> choimory-dev-prod   choimory-dev-redis-secret                 True     4h58m
> choimory-dev-stg    choimory-dev-member-api-secret            True     4h57m
> choimory-dev-stg    choimory-dev-redis-secret                 True     4h58m
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ 
> kubectl -n choimory-dev-stg get pods,pvc,httproute
> kubectl -n choimory-dev-prod get pods,pvc,httproute
> kubectl -n choimory-infra get gateway
> NAME                                           READY   STATUS             RESTARTS   AGE
> pod/choimory-dev-front-85469c8fb5-klvlp        0/1     ImagePullBackOff   0          5d2h
> pod/choimory-dev-member-api-69cd5894dd-vcmbc   0/1     ImagePullBackOff   0          5d2h
> pod/choimory-dev-postgres-1                    1/1     Running            0          5d2h
> pod/choimory-dev-redis-0                       1/1     Running            0          5d2h
>
> NAME                                              STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS              VOLUMEATTRIBUTESCLASS   AGE
> persistentvolumeclaim/choimory-dev-postgres-1     Bound    pvc-<STG_POSTGRES_PV_ID>   5Gi        RWO            local-path-choimory-dev   <unset>                 5d2h
> persistentvolumeclaim/data-choimory-dev-redis-0   Bound    pvc-<STG_REDIS_PV_ID>   1Gi        RWO            local-path-choimory-dev   <unset>                 5d2h
>
> NAME                                                          HOSTNAMES                             AGE
> httproute.gateway.networking.k8s.io/choimory-dev-front        ["staging.choimory.dev"]              5d2h
> httproute.gateway.networking.k8s.io/choimory-dev-member-api   ["staging-member-api.choimory.dev"]   5d2h
> NAME                                          READY   STATUS             RESTARTS   AGE
> pod/choimory-dev-front-56889d449f-x6z6t       0/1     ImagePullBackOff   0          5d2h
> pod/choimory-dev-member-api-dd677d978-pqfk6   0/1     ImagePullBackOff   0          5d2h
> pod/choimory-dev-postgres-1                   1/1     Running            0          5d2h
> pod/choimory-dev-redis-0                      1/1     Running            0          5d2h
>
> NAME                                              STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS              VOLUMEATTRIBUTESCLASS   AGE
> persistentvolumeclaim/choimory-dev-postgres-1     Bound    pvc-<PROD_POSTGRES_PV_ID>   10Gi       RWO            local-path-choimory-dev   <unset>                 5d2h
> persistentvolumeclaim/data-choimory-dev-redis-0   Bound    pvc-<PROD_REDIS_PV_ID>   1Gi        RWO            local-path-choimory-dev   <unset>                 5d2h
>
> NAME                                                          HOSTNAMES                     AGE
> httproute.gateway.networking.k8s.io/choimory-dev-front        ["choimory.dev"]              5d2h
> httproute.gateway.networking.k8s.io/choimory-dev-member-api   ["member-api.choimory.dev"]   5d2h
> NAME             CLASS   ADDRESS          PROGRAMMED   AGE
> shared-gateway   envoy   <GATEWAY_CLUSTER_IP>   True         68s
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ 
> kubectl get apiservice v1beta1.metrics.k8s.io
> kubectl get mutatingwebhookconfigurations
> NAME                     SERVICE                         AVAILABLE   AGE
> v1beta1.metrics.k8s.io   choimory-infra/metrics-server   True        112s
> NAME                                             WEBHOOKS   AGE
> cnpg-mutating-webhook-configuration              4          5d5h
> envoy-gateway-topology-injector.choimory-infra   1          79s
> ```

## 답변

- 재설치 완료. 공용 도구는 모두 `choimory-infra`에서 정상 동작
- 남은 것은 예상했던 정리 작업뿐

## 5번 확인 결과

| 항목 | 결과 |
|---|---|
| 공용 도구 앱 9개와 `root`, `choimory-dev` | 모두 `Synced`, `Healthy` |
| 공용 도구 Pod | ArgoCD 5개와 공용 도구 6개 모두 `Running`. Envoy 프록시 이름이 `envoy-choimory-infra-shared-gateway-...`로 바뀜 |
| SealedSecret | 4개 모두 `SYNCED True`. 기존 키로 풀림 |
| postgres, redis Pod | AGE 5d2h 그대로, RESTARTS 0 |
| PVC | 4개 `Bound`, 볼륨 이름 그대로 |
| Gateway | `shared-gateway` `PROGRAMMED True` |
| APIService | `choimory-infra/metrics-server`, `AVAILABLE True` |
| 웹훅 | cnpg, `envoy-gateway-topology-injector.choimory-infra` |
| front, member-api | `Degraded` (이미지 없음, 예상대로) |
| `choimory-dev-{stg,prod}-namespaces` | `OutOfSync` (정리 대상) |

## namespaces 앱이 OutOfSync인 이유

- 옛 NetworkPolicy `allow-same-namespace-and-infrastructure`가 Git에는 없고 클러스터에는 남아 있음
- namespaces 앱은 prune이 꺼져 있어 ArgoCD가 지우지 않고 "Git과 다름"으로만 표시함
- 이미 없는 `infrastructure` 네임스페이스만 허용하는 규칙이라 지워도 통신에 영향 없음. 직접 지우면 `Synced`가 됨

## 5번 정리 (서버)

```bash
# 5-1) 옛 NetworkPolicy 삭제와 새 NetworkPolicy 확인
kubectl -n choimory-dev-stg delete networkpolicy allow-same-namespace-and-infrastructure
kubectl -n choimory-dev-prod delete networkpolicy allow-same-namespace-and-infrastructure
kubectl -n choimory-dev-stg get networkpolicy
kubectl -n choimory-dev-prod get networkpolicy
```

- 각 네임스페이스에 `allow-same-namespace-and-infra`만 남아야 함

```bash
# 5-2) 이름에 infrastructure가 들어간 클러스터 단위 리소스 정리
kubectl get clusterroles,clusterrolebindings | grep infrastructure
kubectl delete clusterrolebinding envoy-gateway-gateway-helm-certgen:infrastructure
kubectl get clusterroles | grep infrastructure
```

- 마지막 명령어에 `...:infrastructure` 같은 ClusterRole이 나오면 이어서 삭제

```bash
# 5-3) 남은 참조 확인. 결과가 비어 있어야 함
kubectl get clusterrolebindings -o custom-columns='NAME:.metadata.name,NS:.subjects[*].namespace' | grep infrastructure

# 5-4) HTTPRoute가 새 Gateway에 연결됐는지
kubectl -n choimory-dev-stg get httproute -o custom-columns='NAME:.metadata.name,GATEWAY_NS:.status.parents[*].parentRef.namespace,ACCEPTED:.status.parents[*].conditions[?(@.type=="Accepted")].status'
kubectl -n choimory-dev-prod get httproute -o custom-columns='NAME:.metadata.name,GATEWAY_NS:.status.parents[*].parentRef.namespace,ACCEPTED:.status.parents[*].conditions[?(@.type=="Accepted")].status'

# 5-5) namespaces 앱 상태
kubectl -n choimory-infra get applications | grep namespaces
```

- 5-4) `GATEWAY_NS`가 `choimory-infra`, `ACCEPTED`가 `True`
- 5-5) 두 앱이 `Synced` (1~3분 뒤 확인)

## 마지막 정리 (직접)

- ArgoCD 로그인: 새 비밀번호로 화면에 로그인한 뒤 `argocd-initial-admin-secret` 삭제
- 키 백업 보관: `~/sealed-secrets-key-backup.yaml`을 서버 밖 안전한 곳(비밀번호 관리자 등)으로 옮기고 위치를 기록

---

# 5번 정리 결과

> ```
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ 
> kubectl -n choimory-dev-stg delete networkpolicy allow-same-namespace-and-infrastructure
> kubectl -n choimory-dev-prod delete networkpolicy allow-same-namespace-and-infrastructure
> kubectl -n choimory-dev-stg get networkpolicy
> kubectl -n choimory-dev-prod get networkpolicy
> networkpolicy.networking.k8s.io "allow-same-namespace-and-infrastructure" deleted from choimory-dev-stg namespace
> networkpolicy.networking.k8s.io "allow-same-namespace-and-infrastructure" deleted from choimory-dev-prod namespace
> NAME                             POD-SELECTOR   AGE
> allow-same-namespace-and-infra   <none>         2m46s
> NAME                             POD-SELECTOR   AGE
> allow-same-namespace-and-infra   <none>         2m46s
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ 
> kubectl get clusterroles,clusterrolebindings | grep infrastructure
> kubectl delete clusterrolebinding envoy-gateway-gateway-helm-certgen:infrastructure
> kubectl get clusterroles | grep infrastructure
> clusterrole.rbac.authorization.k8s.io/envoy-gateway-gateway-helm-certgen:infrastructure                      2026-11-10T09:30:11Z
> clusterrolebinding.rbac.authorization.k8s.io/envoy-gateway-gateway-helm-certgen:infrastructure               ClusterRole/envoy-gateway-gateway-helm-certgen:infrastructure                      5d5h
> clusterrolebinding.rbac.authorization.k8s.io "envoy-gateway-gateway-helm-certgen:infrastructure" deleted
> envoy-gateway-gateway-helm-certgen:infrastructure                      2026-11-10T09:30:11Z
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ 
> kubectl get clusterrolebindings -o custom-columns='NAME:.metadata.name,NS:.subjects[*].namespace' | grep infrastructure
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ 
> kubectl -n choimory-dev-stg get httproute -o custom-columns='NAME:.metadata.name,GATEWAY_NS:.status.parents[*].parentRef.namespace,ACCEPTED:.status.parents[*].conditions[?(@.type=="Accepted")].status'
> kubectl -n choimory-dev-prod get httproute -o custom-columns='NAME:.metadata.name,GATEWAY_NS:.status.parents[*].parentRef.namespace,ACCEPTED:.status.parents[*].conditions[?(@.type=="Accepted")].status'
> NAME                      GATEWAY_NS       ACCEPTED
> choimory-dev-front        choimory-infra   True
> choimory-dev-member-api   choimory-infra   True
> NAME                      GATEWAY_NS       ACCEPTED
> choimory-dev-front        choimory-infra   True
> choimory-dev-member-api   choimory-infra   True
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ 
> kubectl -n choimory-infra get applications | grep namespaces
> choimory-dev-prod-namespaces   Synced        Healthy
> choimory-dev-stg-namespaces    Synced        Healthy
> ```

## 답변

- 5번 정리 확인. 남은 것은 ClusterRole `envoy-gateway-gateway-helm-certgen:infrastructure` 하나이며, 이것만 지우면 `infrastructure` 흔적이 모두 사라짐

## 정리 결과

| 항목 | 결과 |
|---|---|
| 옛 NetworkPolicy | stg, prod 모두 삭제. 새 `allow-same-namespace-and-infra`만 남음 |
| ClusterRoleBinding `...certgen:infrastructure` | 삭제됨 |
| `infrastructure`를 가리키는 ClusterRoleBinding | 없음 |
| HTTPRoute | 4개 모두 `choimory-infra` Gateway에 연결, `ACCEPTED True` |
| namespaces 앱 | stg, prod 모두 `Synced`, `Healthy` |
| ClusterRole `...certgen:infrastructure` | 남아 있음 (삭제 필요) |

## 남은 정리 (서버)

```bash
kubectl delete clusterrole envoy-gateway-gateway-helm-certgen:infrastructure
kubectl get clusterroles,clusterrolebindings | grep infrastructure
```

- 두 번째 명령어 결과가 비어 있으면 완료

## 마지막 정리 (직접)

- ArgoCD 로그인과 초기 비밀번호 삭제

```bash
# 초기 비밀번호 확인 (서버)
kubectl -n choimory-infra get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

- task5 때와 같은 방법으로 화면에 접속해 `admin`으로 로그인. 네임스페이스만 `choimory-infra`로 바뀜
- 로그인 후 User Info → Update Password에서 비밀번호 변경 추천. 변경 후 초기 비밀번호 Secret 삭제

```bash
kubectl -n choimory-infra delete secret argocd-initial-admin-secret
```

- 키 백업 보관
  - `~/sealed-secrets-key-backup.yaml`을 서버 밖 안전한 곳(비밀번호 관리자 등)에 옮기고 서버의 파일은 삭제
  - 보관 위치를 기록. 클러스터를 다시 설치할 때 이 키가 있어야 기존 SealedSecret을 풀 수 있음

## 네임스페이스 변경 완료 후

- 위 정리가 끝나면 `infrastructure` → `choimory-infra` 변경 작업 완료
- 원래 하던 7단계로 돌아감
  1. 앱 레포 2개의 `deploy.yml` 커밋, `dev` push, `stg`·`main` 머지
  2. Actions 테스트 결과 확인
  3. `stg` 브랜치로 Run workflow 실행

---

# 네임스페이스 변경 완료

> ```
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ 
> kubectl delete clusterrole envoy-gateway-gateway-helm-certgen:infrastructure
> kubectl get clusterroles,clusterrolebindings | grep infrastructure
> clusterrole.rbac.authorization.k8s.io "envoy-gateway-gateway-helm-certgen:infrastructure" deleted
> ```

## 답변

- `infrastructure` → `choimory-infra` 네임스페이스 변경 완료. 클러스터에 `infrastructure` 흔적이 남아 있지 않음

## 작업 결과

| 항목 | 상태 |
|---|---|
| 공용 도구 | 모두 `choimory-infra`에서 동작. 앱 `Synced`, `Healthy` |
| SealedSecret | 기존 키로 4개 모두 풀림 |
| postgres, redis | 재시작 없이 계속 동작. PVC 그대로 |
| HTTPRoute | 새 Gateway에 연결됨 |
| 남은 옛 리소스 | 없음 (NetworkPolicy, 웹훅, ClusterRole, ClusterRoleBinding 모두 정리) |

## 남은 직접 작업

- ArgoCD: 로그인 → 비밀번호 변경 → `argocd-initial-admin-secret` 삭제
- 키 백업: `~/sealed-secrets-key-backup.yaml`을 서버 밖 안전한 곳에 옮기고 위치를 기록한 뒤 서버의 파일 삭제

## 다음: 7단계로 복귀

1. 앱 레포 2개(`choimory-dev-front`, `choimory-dev-member-api`)의 `deploy.yml` 수정 커밋, `dev` push
2. release 브랜치로 `stg`, `main` 머지
3. Actions에서 test job 성공, deploy job skipped 확인. `main` 머지 후 Run workflow 버튼 확인
4. `stg` 브랜치로 Run workflow 실행

---

# ArgoCD 접속 방법

> argocd 접속 아직 한번도 안해봤음. 접속주소가 어케됨?

## 답변

- ArgoCD는 외부에 공개하지 않아 접속 주소가 없음
- 서버에서 port-forward로 열고, Mac에서 SSH 터널로 연결해 접속함
- task5의 "관리 화면 접속" 방법과 같고 네임스페이스만 `choimory-infra`로 바뀜

## 1. 서버에서 실행

```bash
# 초기 비밀번호 확인
kubectl -n choimory-infra get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d; echo

# ArgoCD 화면을 서버의 8080 포트로 연결. 접속하는 동안 이 터미널은 켜 둠
kubectl -n choimory-infra port-forward svc/argocd-server 8080:443
```

## 2. Mac에서 실행

- 새 터미널에서 실행. `<서버계정>`은 실제 서버 계정
- Tailscale 이름(`choimory-server`)을 쓰므로 집 밖에서도 가능

```bash
ssh -L 8080:localhost:8080 <서버계정>@choimory-server
```

## 3. Mac 브라우저에서 접속

- 주소: `https://localhost:8080`
- 계정: `admin`, 비밀번호는 1번에서 확인한 값
- 자체 서명 인증서라 "안전하지 않음" 경고가 뜨는 것이 정상. 고급 → 계속 진행

## 연결 구조

```
Mac 브라우저 → localhost:8080
  → SSH 터널 → 서버 localhost:8080
  → port-forward → choimory-infra/argocd-server:443
```

## 접속 후

1. User Info → Update Password로 비밀번호 변경
2. 서버에서 초기 비밀번호 Secret 삭제

```bash
kubectl -n choimory-infra delete secret argocd-initial-admin-secret
```

3. 다 쓰면 서버 port-forward 터미널은 `Ctrl+C`, Mac SSH 터미널은 `exit`

- 초기 비밀번호는 문서에 남기지 않음
- 매번 여는 것이 번거로우면 2차 Cloudflare 작업 때 접근 제한을 걸고 도메인으로 여는 방법도 있음. 지금은 이 방식이 가장 안전함

---

# ArgoCD 접속 처리 보류와 task8 종료

> 그럼 그냥 argocd 보류한다는 내용만 task8에 추가하고 task8은 닫자

## 답변

- ArgoCD 접속 처리를 보류하고 task8(네임스페이스 변경 작업)을 종료함
- 결정 과정은 [plan10.md 인프라 도구 접속은 이후로 미룸](./plan10.md#인프라-도구-접속은-이후로-미룸), [plan10.md 도메인 연결 후 초기 비밀번호 처리](./plan10.md#도메인-연결-후-초기-비밀번호-처리) 참고

## 보류한 것

| 항목 | 현재 상태 | 처리 시점 |
|---|---|---|
| ArgoCD 도메인 접속 | 외부 접속 주소 없음 | 앱 배포와 Cloudflare Tunnel 이후, 구매한 도메인으로 인프라 도구 연결할 때 |
| ArgoCD 초기 비밀번호 | `argocd-initial-admin-secret` 유지, 비밀번호 초기값 | 위 도메인 연결 작업의 첫 단계 |

## 보류 중 접속 방법

- 필요하면 port-forward와 SSH 터널로 접속 ([ArgoCD 접속 방법](#argocd-접속-방법))

## 재개할 때 순서

1. Cloudflare Access(로그인 관문)를 먼저 걸고 ArgoCD 도메인 연결
2. 초기 비밀번호 확인 후 로그인
3. 비밀번호 변경
4. `argocd-initial-admin-secret` 삭제

- 비밀번호를 바꾸기 전에 Secret만 지우지 않음
- Cloudflare Access 없이 도메인을 먼저 열지 않음

## task8 종료

- `infrastructure` → `choimory-infra` 네임스페이스 변경 작업 완료
- 직접 할 일로 남은 것: sealed-secrets 키 백업(`~/sealed-secrets-key-backup.yaml`)을 서버 밖 안전한 곳에 보관하고 서버의 파일 삭제
- 이후 작업(7단계 앱 배포)은 [plan11.md](./plan11.md)에서 이어서 진행
