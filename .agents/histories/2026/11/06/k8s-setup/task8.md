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
