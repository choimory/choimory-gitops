# 목차

- [개요](#개요)
- [남은 작업](#남은-작업)
  - [이번 작업](#이번-작업)
  - [전체 남은 단계](#전체-남은-단계)
  - [7단계 전에 직접 할 일](#7단계-전에-직접-할-일)
  - [아직 검증하지 못한 것](#아직-검증하지-못한-것)
  - [범위를 정해야 하는 것](#범위를-정해야-하는-것)
- [5단계 실행 명령어](#5단계-실행-명령어)
  - [1. 시작 전 확인 (서버)](#1-시작-전-확인-서버)
  - [2. ArgoCD 설치 (서버)](#2-argocd-설치-서버)
  - [3. root 등록 (서버)](#3-root-등록-서버)
  - [4. 동기화 확인 (서버)](#4-동기화-확인-서버)
  - [5. 문제 발생 시 확인 (서버)](#5-문제-발생-시-확인-서버)
  - [6. 관리 화면 접속 (선택)](#6-관리-화면-접속-선택)
- [gateway Degraded 확인](#gateway-degraded-확인)
  - [현재 상태](#현재-상태)
  - [원인 후보](#원인-후보)
  - [확인 명령어 (서버)](#확인-명령어-서버)
- [gateway Degraded 원인과 조치](#gateway-degraded-원인과-조치)
  - [원인](#원인)
  - [지금 할 조치 (서버)](#지금-할-조치-서버)
  - [재발 방지 (제안, 레포 수정)](#재발-방지-제안-레포-수정)
- [gateway Healthy 전환과 wave 7 확인](#gateway-healthy-전환과-wave-7-확인)
  - [결과](#결과)
  - [원인 재확인](#원인-재확인)
  - [wave 7 확인 명령어 (서버)](#wave-7-확인-명령어-서버)
- [GatewayClass 원인 확정과 root 수동 동기화](#gatewayclass-원인-확정과-root-수동-동기화)
  - [GatewayClass 원인 확정](#gatewayclass-원인-확정)
  - [wave 7 상태](#wave-7-상태)
  - [root 동기화 명령어 (서버)](#root-동기화-명령어-서버)
  - [재발 방지 annotation (허락 요청)](#재발-방지-annotation-허락-요청)
- [root 동기화 명령어 재안내](#root-동기화-명령어-재안내)
  - [확인 결과](#확인-결과)
  - [실행할 명령어 (서버)](#실행할-명령어-서버)
- [5단계 완료](#5단계-완료)
  - [root 동기화 결과](#root-동기화-결과)
  - [앱 상태 확인 (선택)](#앱-상태-확인-선택)
  - [다음 진행 선택](#다음-진행-선택)
- [재발 방지 annotation 적용과 6단계 계획](#재발-방지-annotation-적용과-6단계-계획)
  - [재발 방지 annotation 적용](#재발-방지-annotation-적용)
  - [필요한 Secret](#필요한-secret)
  - [kubeseal 설치 (서버)](#kubeseal-설치-서버)
  - [암호화 키 백업 (서버)](#암호화-키-백업-서버)
  - [Mac으로 가져오기](#mac으로-가져오기)
  - [가져온 뒤 진행할 작업](#가져온-뒤-진행할-작업)
- [리소스 이름 변경 계획](#리소스-이름-변경-계획)
  - [Pod 상태](#pod-상태)
  - [변경 대상](#변경-대상)
  - [정할 것](#정할-것)
- [리소스 이름 변경 적용](#리소스-이름-변경-적용)
  - [변경 내용](#변경-내용)
  - [push 후 예전 리소스 정리 (서버)](#push-후-예전-리소스-정리-서버)
  - [SealedSecret 생성 (서버)](#sealedsecret-생성-서버)
  - [진행 순서](#진행-순서)
- [이름 변경 후 진행 순서](#이름-변경-후-진행-순서)
  - [1. 커밋과 push (Mac)](#1-커밋과-push-mac)
  - [2. 반영 확인과 예전 리소스 정리 (서버)](#2-반영-확인과-예전-리소스-정리-서버)
  - [3. SealedSecret 만들기 (서버, Mac)](#3-sealedsecret-만들기-서버-mac)
  - [4. 알려 줄 것](#4-알려-줄-것)
- [이름 변경 반영 확인](#이름-변경-반영-확인)
  - [반영 상태](#반영-상태)
  - [예전 리소스 정리 명령어 (서버)](#예전-리소스-정리-명령어-서버)
- [예전 PV와 폴더 정리](#예전-pv와-폴더-정리)
  - [정리 결과](#정리-결과)
  - [PV와 폴더 삭제 명령어 (서버)](#pv와-폴더-삭제-명령어-서버)
  - [kubeseal 설치 전 확인 (서버)](#kubeseal-설치-전-확인-서버)
- [PV와 폴더 정리 완료](#pv와-폴더-정리-완료)
  - [정리 확인](#정리-확인)
  - [다음 단계](#다음-단계)
- [kubeseal 0.41.2 설치와 SealedSecret 생성](#kubeseal-0412-설치와-sealedsecret-생성)
  - [1. kubeseal 설치 (서버)](#1-kubeseal-설치-서버)
  - [2. SealedSecret 생성 (서버)](#2-sealedsecret-생성-서버)
  - [3. 암호화 키 백업 (서버)](#3-암호화-키-백업-서버)
  - [4. Mac으로 가져오기 (Mac)](#4-mac으로-가져오기-mac)
- [SealedSecret 생성 결과](#sealedsecret-생성-결과)
  - [서버 작업 결과](#서버-작업-결과)
  - [Mac으로 가져오기 명령어](#mac으로-가져오기-명령어)
- [SealedSecret 레포 반영](#sealedsecret-레포-반영)
  - [파일 확인](#파일-확인)
  - [추가한 파일](#추가한-파일)
  - [커밋 후 반영 확인 (서버)](#커밋-후-반영-확인-서버)
  - [임시 파일 정리](#임시-파일-정리)
- [SealedSecret을 public 레포에 올려도 되는지](#sealedsecret을-public-레포에-올려도-되는지)
  - [답변](#답변)
  - [안전한 이유](#안전한-이유)
  - [주의할 점](#주의할-점)
- [디렉토리 이름 변경 검토](#디렉토리-이름-변경-검토)
  - [영향 범위](#영향-범위)
  - [Application 이름 문제](#application-이름-문제)
  - [앱 레포 workflow](#앱-레포-workflow)
- [디렉토리 이름 변경 보류와 다음 할 일](#디렉토리-이름-변경-보류와-다음-할-일)
  - [다음 할 일 요약](#다음-할-일-요약)
- [task5 정리](#task5-정리)
  - [완료한 것](#완료한-것)
  - [레포 변경](#레포-변경)
  - [겪은 문제와 해결](#겪은-문제와-해결)
  - [이어서 할 것](#이어서-할-것)

---

# 개요

- 구현 순서 5단계(ArgoCD 설치와 root 등록)에서 실행할 명령어를 순서대로 정리함
- 서버의 gitops 경로는 `/data/dev/choimory/repo/choimory-gitops`
- 명령어는 실제로 실행해 보지 않았음
- 사전 조건: 4단계 완료, gitops 변경이 `main`에 push되어 있어야 함
- ArgoCD는 GitHub의 `main`을 읽으므로, 서버의 clone은 최초 설치에 쓸 파일을 가져오는 용도

---

# 남은 작업

> 남은게 argocd랑 뭐뭐 남았음?

## 이번 작업

- 이번에는 5단계 ArgoCD 설치와 root 등록을 진행함
- 4단계(서버 k8s 설치)까지 완료된 상태
- 완료 기준은 `argocd`와 공용 도구 7개가 `Synced`, `Healthy`인 상태. `stg`, `prod` 앱의 정상 동작은 6~7단계에서 확인

## 전체 남은 단계

| 순서 | 작업 | 내용 | 상태 |
|---|---|---|---|
| 5 | ArgoCD 설치 | Helm 설치, root 등록, 공용 도구 동기화 | 이번 작업 |
| 6 | Secret 등록 | `kubeseal`로 DB, Redis, JWT 등을 암호화해 SealedSecret만 커밋 | 5단계 후 |
| 7 | 1차 확인 | 앱 Pod 정상 여부, `port-forward`와 `curl`로 헬스체크, 로그인 API 확인 | 6단계 후 |
| 2차 | 외부 공개 | 도메인 연결, Cloudflare Tunnel | 1차 후 |

## 7단계 전에 직접 할 일

1. 앱 레포 2개에 Deploy Key 등록(`GITOPS_DEPLOY_KEY`)
2. front의 GitHub Environments 변수 등록
3. 앱 레포를 `stg`에 머지해 Actions 실행. 이미지 빌드 후 gitops에 실제 태그가 커밋됨. 앱 레포의 미push 커밋도 이때 올라감

- GHCR 패키지 공개 설정 확인 필요. 매니페스트에 `imagePullSecrets`가 없어 패키지가 비공개면 `ImagePullBackOff`가 발생함. 첫 빌드 후 패키지를 public으로 바꾸거나 pull secret을 추가해야 함

## 아직 검증하지 못한 것

- Mac에서 Tailscale로 `kubectl` 접속(건너뛴 항목)
- GitHub Actions 실행, ArgoCD 동기화
- member-api의 actuator 응답(8081)
- `/data/dev/choimory-dev/k8s`에 PVC 폴더가 생성되는지

## 범위를 정해야 하는 것

- etcd와 `/etc/kubernetes/pki` 백업
- Uptime Kuma, Homepage 연동
- 배포 실패 알림
- HPA 테스트

- 1차 확인 뒤에 결정해도 되지만, 데이터가 쌓이기 시작하면 백업은 먼저 챙기는 것을 추천

---

# 5단계 실행 명령어

> 이제 다음작업 해야할 명령어 task5에 작성

## 1. 시작 전 확인 (서버)

```bash
cd /data/dev/choimory/repo/choimory-gitops
git pull
git log -1 --oneline
ls bootstrap/root.yaml infrastructure/argocd/values.yaml
kubectl get nodes
```

- 확인: 파일 2개가 있고 노드가 `Ready`

## 2. ArgoCD 설치 (서버)

- 릴리스 이름 `argocd`, 네임스페이스 `infrastructure`, 차트 버전 `10.17.0`, values 파일은 `infrastructure/apps/argocd.yaml`과 같아야 함

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argocd argo/argo-cd --version 10.17.0 -n infrastructure --create-namespace -f infrastructure/argocd/values.yaml

kubectl -n infrastructure rollout status statefulset/argocd-application-controller --timeout=5m
kubectl -n infrastructure rollout status deploy/argocd-server --timeout=5m
kubectl -n infrastructure rollout status deploy/argocd-repo-server --timeout=5m
kubectl -n infrastructure rollout status deploy/argocd-applicationset-controller --timeout=5m
kubectl -n infrastructure get pods

kubectl -n infrastructure get cm argocd-cm -o yaml | grep  'argoproj.io_Application'
```

- 확인: `STATUS: deployed`, rollout 4개 완료, Pod 모두 `Running`
- `dex`, `notifications`는 values에서 껐으므로 Pod가 없는 것이 정상

## 3. root 등록 (서버)

- ArgoCD Pod가 모두 준비된 뒤 한 번만 실행

```bash
kubectl apply -f bootstrap/root.yaml
kubectl -n infrastructure get applications
```

- 확인: `root` Application이 생성됨

## 4. 동기화 확인 (서버)

- sync wave 순서대로 설치되므로 전체 완료까지 몇 분 걸릴 수 있음. 아래 명령을 시간을 두고 반복 실행

```bash
kubectl -n infrastructure get applications
kubectl -n infrastructure get applicationsets
kubectl -n infrastructure get appprojects
kubectl get storageclass
kubectl get ns
kubectl get pods -A
```

- 확인: 공용 도구 Application이 `Synced`, `Healthy`

| wave | Application |
|---|---|
| 0 | `argocd` |
| 1 | `sealed-secrets` |
| 2 | `local-path-provisioner` |
| 3 | `metrics-server` |
| 4 | `cloudnative-pg` |
| 5 | `envoy-gateway` |
| 6 | `gateway` |
| 7 | `choimory-dev` |

- 확인: `choimory-dev`가 만든 Application 8개(`choimory-dev-{stg,prod}-{namespaces,data,member-api,front}`)가 생성됨
- 확인: StorageClass `local-path`, `local-path-choimory-dev`가 있고, 네임스페이스 `choimory-dev-stg`, `choimory-dev-prod`가 있음
- 6단계 전에는 `data`, `member-api`, `front`가 `Progressing`이나 `Degraded`일 수 있음. Secret(SealedSecret)이 아직 없고, 앱 이미지 태그 `stg-bootstrap`, `prod-bootstrap`은 임시값이므로 `CreateContainerConfigError`, `ImagePullBackOff`가 나올 수 있음

## 5. 문제 발생 시 확인 (서버)

- 공용 도구 Application이 `OutOfSync`, `Degraded`, `Unknown`에 머물 때 실행. `<이름>`은 실제 Application 이름으로 변경

```bash
kubectl -n infrastructure describe application <이름>
kubectl -n infrastructure logs deploy/argocd-repo-server --tail=50
kubectl -n infrastructure logs statefulset/argocd-application-controller --tail=50
```

## 6. 관리 화면 접속 (선택)

- 초기 비밀번호는 문서에 남기지 않음

서버에서 실행

```bash
kubectl -n infrastructure get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d; echo
kubectl -n infrastructure port-forward svc/argocd-server 8080:443
```

Mac에서 실행 (다른 터미널). `<서버계정>`은 실제 계정으로 변경

```bash
ssh -L 8080:localhost:8080 <서버계정>@choimory-server
```

- 브라우저에서 `https://localhost:8080` 접속. 계정은 `admin`
- 자체 서명 인증서라 브라우저 경고가 나오는 것이 정상

---

# gateway Degraded 확인

> ```
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl get applications -n infrastructure 
> NAME                     SYNC STATUS   HEALTH STATUS
> argocd                   Synced        Healthy
> cloudnative-pg           Synced        Healthy
> envoy-gateway            Synced        Healthy
> gateway                  Synced        Degraded
> local-path-provisioner   Synced        Healthy
> metrics-server           Synced        Healthy
> root                     OutOfSync     Degraded
> sealed-secrets           Synced        Healthy
> ```

## 현재 상태

- wave 0~5는 정상(`Synced`, `Healthy`)
- `envoy-gateway`는 한동안 `OutOfSync`, `Missing`에 머물렀으나, OCI 차트를 받아 오는 데 시간이 걸렸을 뿐 이후 정상이 됨. 수정 불필요
- wave 6 `gateway`가 `Synced`, `Degraded`
  - `Synced`: EnvoyProxy, GatewayClass, Gateway 리소스는 클러스터에 생성됨
  - `Degraded`: 생성된 리소스가 정상 상태가 되지 못함
- `root`의 `Degraded`는 `gateway` 상태를 따라간 것이며, 이 때문에 wave 7 `choimory-dev`가 아직 생성되지 않음

## 원인 후보

- 아직 추측이며 아래 명령어 출력으로 확인 필요
1. Gateway가 `Programmed`가 되지 못함. GatewayClass가 `Accepted`되지 않았거나 EnvoyProxy 설정이 거부된 경우
2. Envoy 프록시 Pod가 뜨지 못함. 이미지 pull 실패, Pending, CrashLoop 등

## 확인 명령어 (서버)

```bash
# 어떤 리소스가 Degraded인지
kubectl -n infrastructure get application gateway -o jsonpath='{range .status.resources[*]}{.kind}{"\t"}{.name}{"\t"}{.health.status}{"\t"}{.health.message}{"\n"}{end}'

# GatewayClass, Gateway 상태
kubectl get gatewayclass envoy -o jsonpath='{.status.conditions}'; echo
kubectl -n infrastructure get gateway shared-gateway -o jsonpath='{.status.conditions}'; echo

# Envoy Gateway 컨트롤러와 프록시 Pod
kubectl -n infrastructure get pods,svc | grep -i envoy
kubectl -n infrastructure logs deploy/envoy-gateway --tail=30
```

- 첫 번째 명령: `gateway` Application에 속한 리소스별 종류, 이름, health 상태와 메시지를 출력. `Degraded`인 리소스를 찾는 용도
- `gatewayclass` 조회: `Accepted` 조건이 `True`인지 확인. `False`면 컨트롤러가 GatewayClass나 EnvoyProxy를 받아들이지 않은 것
- `gateway` 조회: `Accepted`, `Programmed` 조건 확인. `Programmed`가 `False`면 메시지에 이유가 나옴
- `get pods,svc | grep -i envoy`: Envoy Gateway 컨트롤러 Pod와, Gateway를 위해 만들어진 프록시 Pod, Service 상태 확인
- `logs deploy/envoy-gateway`: 컨트롤러가 Gateway를 처리하다 낸 오류 확인

---

# gateway Degraded 원인과 조치

> ```
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n infrastructure get application gateway -o jsonpath='{range .status.resources[*]}{.kind}{"\t"}{.name}{"\t"}{.health.status}{"\t"}{.health.message}{"\n"}{end}'
> EnvoyProxy      envoy-proxy
> Gateway shared-gateway
> GatewayClass    envoy
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl get gatewayclass envoy -o jsonpath='{.status.conditions}'; echo
> [{"lastTransitionTime":"2026-11-10T09:30:52Z","message":"Invalid parametersRef: failed to find envoyproxy infrastructure/envoy-proxy for GatewayClass envoy: EnvoyProxy.gateway.envoyproxy.io \"envoy-proxy\" not found","observedGeneration":1,"reason":"InvalidParameters","status":"False","type":"Accepted"}]
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n infrastructure get gateway shared-gateway -o jsonpath='{.status.conditions}'; echo
> [{"lastTransitionTime":"2026-11-10T09:31:03Z","message":"The Gateway has been scheduled by Envoy Gateway","observedGeneration":1,"reason":"Accepted","status":"True","type":"Accepted"},{"lastTransitionTime":"2026-11-10T09:31:03Z","message":"Address assigned to the Gateway, 1/1 envoy replicas available","observedGeneration":1,"reason":"Programmed","status":"True","type":"Programmed"}]
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n infrastructure get pods,svc | grep -i envoy
> pod/envoy-gateway-65c678d44b-jb4zj                                 1/1     Running   0          80m
> pod/envoy-infrastructure-shared-gateway-02d9540f-74bdf9fb6-m5rcm   2/2     Running   0          80m
> service/envoy-gateway                                  ClusterIP   10.109.124.80    <none>        18000/TCP,18001/TCP,18002/TCP,19001/TCP,9443/TCP   80m
> service/envoy-infrastructure-shared-gateway-02d9540f   ClusterIP   10.108.239.252   <none>        80/TCP                                             80m
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n infrastructure logs deploy/envoy-gateway --tail=30
> (로그 생략: 모두 info 레벨, "reconciled gateways successfully", Gateway status 업데이트 수신. error 없음)
> ```

## 원인

- Gateway 자체는 정상. GatewayClass에 생성 직후의 오류 상태가 남아 있는 것이 원인

| 리소스 | 상태 | 의미 |
|---|---|---|
| Gateway `shared-gateway` | `Accepted` True, `Programmed` True (09:31:03) | 정상. 주소 할당, 프록시 1/1 가용 |
| 프록시 Pod, Service | `2/2 Running`, ClusterIP | 정상 |
| GatewayClass `envoy` | `Accepted` **False** (09:30:52) | `envoy-proxy`를 못 찾았다는 오류 |

- 생성 순서가 꼬임
  - 09:30:52: GatewayClass가 먼저 처리됨. 이때 EnvoyProxy가 아직 없어 `Accepted: False`가 기록됨
  - 09:31:03: EnvoyProxy가 생기고 Gateway는 정상 처리됨
  - 컨트롤러가 GatewayClass 상태를 다시 계산하지 않아 처음의 `False`가 그대로 남음
- ArgoCD는 이 `Accepted: False`를 보고 `gateway`를 `Degraded`로 판단한 것으로 보임. 그 결과 `root`도 `Degraded`가 되어 wave 7이 멈춤
- 첫 번째 명령에서 리소스별 health가 비어 있던 것은 ArgoCD 3.x가 기본적으로 리소스별 health를 Application status에 저장하지 않기 때문으로 보임. 이번 문제와는 무관

## 지금 할 조치 (서버)

- Envoy Gateway 컨트롤러를 재시작해 GatewayClass 상태를 다시 계산하게 함. 레포는 바꾸지 않음
- 재시작하는 것은 컨트롤러 Pod 하나뿐. 프록시 Pod는 이미 받은 설정으로 계속 동작하므로 영향이 거의 없고, 아직 외부 트래픽도 없음

```bash
kubectl -n infrastructure rollout restart deploy/envoy-gateway
kubectl -n infrastructure rollout status deploy/envoy-gateway --timeout=2m

# Accepted가 True로 바뀌었는지
kubectl get gatewayclass envoy -o jsonpath='{.status.conditions}'; echo

# 잠시 후 gateway Healthy, choimory-dev 생성 확인
kubectl -n infrastructure get applications
```

- `rollout restart`: 컨트롤러 Deployment의 Pod를 새로 띄움. 시작하면서 모든 리소스를 다시 처리함
- `rollout status`: 새 Pod가 준비될 때까지 대기
- `gatewayclass` 조회: `Accepted`가 `True`로 바뀌었는지 확인
- `get applications`: `gateway`가 `Healthy`가 되고 wave 7 `choimory-dev`가 생성되는지 확인

GatewayClass가 `True`가 됐는데도 Application 상태가 바로 바뀌지 않으면, 아래 명령으로 ArgoCD가 상태를 다시 읽게 함

```bash
kubectl -n infrastructure annotate application gateway argocd.argoproj.io/refresh=normal --overwrite
kubectl -n infrastructure annotate application root argocd.argoproj.io/refresh=normal --overwrite
```

- `refresh=normal` annotation: ArgoCD가 Application 상태를 즉시 다시 계산함. 처리 후 annotation은 ArgoCD가 지움

## 재발 방지 (제안, 레포 수정)

- 클러스터를 다시 설치하면 같은 경쟁 상태가 다시 생길 수 있음
- `infrastructure/gateway/gateway.yaml`의 EnvoyProxy에 아래 annotation을 추가하는 것을 제안. 아직 적용하지 않음

```yaml
annotations:
  # GatewayClass가 참조하므로 먼저 만들어야 한다.
  argocd.argoproj.io/sync-wave: "-1"
```

- `gateway` Application 안에서 EnvoyProxy가 먼저 만들어지고 Healthy가 된 뒤에 GatewayClass와 Gateway가 생성됨

---

# gateway Healthy 전환과 wave 7 확인

> ```
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n infrastructure rollout restart deploy/envoy-gateway
> deployment.apps/envoy-gateway restarted
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n infrastructure rollout status deploy/envoy-gateway --timeout=2m
> deployment "envoy-gateway" successfully rolled out
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl get gatewayclass envoy -o jsonpath='{.status.conditions}'; echo
> [{"lastTransitionTime":"2026-11-10T09:30:52Z","message":"Invalid parametersRef: failed to find envoyproxy infrastructure/envoy-proxy for GatewayClass envoy: EnvoyProxy.gateway.envoyproxy.io \"envoy-proxy\" not found","observedGeneration":1,"reason":"InvalidParameters","status":"False","type":"Accepted"}]
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n infrastructure get applications
> NAME                     SYNC STATUS   HEALTH STATUS
> argocd                   Synced        Healthy
> cloudnative-pg           Synced        Healthy
> envoy-gateway            Synced        Healthy
> gateway                  Synced        Degraded
> local-path-provisioner   Synced        Healthy
> metrics-server           Synced        Healthy
> root                     OutOfSync     Degraded
> sealed-secrets           Synced        Healthy
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n infrastructure annotate application gateway argocd.argoproj.io/refresh=normal --overwrite
> application.argoproj.io/gateway annotated
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n infrastructure annotate application root argocd.argoproj.io/refresh=normal --overwrite
> application.argoproj.io/root annotated
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n infrastructure get applications
> NAME                     SYNC STATUS   HEALTH STATUS
> argocd                   Synced        Healthy
> cloudnative-pg           Synced        Healthy
> envoy-gateway            Synced        Healthy
> gateway                  Synced        Healthy
> local-path-provisioner   Synced        Healthy
> metrics-server           Synced        Healthy
> root                     OutOfSync     Healthy
> sealed-secrets           Synced        Healthy
> ```

## 결과

- 컨트롤러 재시작과 `refresh` 후 `gateway`가 `Healthy`로 바뀜
- `root`도 `Healthy`가 되어, 막혀 있던 wave 7 `choimory-dev`가 이어서 동기화될 예정
- `root`의 `OutOfSync`는 wave 7이 아직 남아 있기 때문

## 원인 재확인

- 재시작 직후에도 GatewayClass의 `Accepted`는 여전히 `False`(09:30:52)였는데, `refresh` 후 `gateway`는 `Healthy`가 됨. 앞에서 추정한 원인이 아직 정확히 확인되지 않음
- 가능성
  1. 컨트롤러가 재시작 뒤 GatewayClass 상태를 늦게 갱신했고, 확인 명령을 너무 일찍 실행함
  2. `Degraded`의 원인이 GatewayClass가 아니라 ArgoCD가 캐시한 health였고, `refresh`로 다시 계산되며 풀림
- GatewayClass를 다시 조회해 `True`면 1번, 여전히 `False`면 2번. 이 결과로 재발 방지 annotation(EnvoyProxy `sync-wave: "-1"`)이 필요한지 판단

## wave 7 확인 명령어 (서버)

```bash
# GatewayClass 상태 재확인
kubectl get gatewayclass envoy -o jsonpath='{.status.conditions}'; echo

# wave 7 진행 확인 (1~2분 간격으로 몇 번 반복)
kubectl -n infrastructure get applications
kubectl -n infrastructure get applicationsets
kubectl -n infrastructure get appprojects
kubectl get storageclass
kubectl get ns
```

- `gatewayclass` 조회: `Accepted`가 `True`로 갱신됐는지 확인. 원인 재확인용
- `get applications`: `choimory-dev`와, 그것이 만든 Application 8개(`choimory-dev-{stg,prod}-{namespaces,data,member-api,front}`)가 생성됐는지 확인
- `get applicationsets`: `choimory-dev`의 ApplicationSet 2개(apps, protected)가 생성됐는지 확인
- `get appprojects`: AppProject `choimory-dev`가 생성됐는지 확인
- `get storageclass`: `local-path`, `local-path-choimory-dev`가 있는지 확인
- `get ns`: `choimory-dev-stg`, `choimory-dev-prod`가 있는지 확인

- 6단계 전에는 `data`, `member-api`, `front`가 `Degraded`나 `Progressing`이어도 정상. Secret(SealedSecret)과 실제 이미지 태그가 아직 없기 때문

---

# GatewayClass 원인 확정과 root 수동 동기화

> ```
> (앞부분의 rollout restart, refresh 출력은 이전 섹션 인용과 동일)
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl get gatewayclass envoy -o jsonpath='{.status.conditions}'; echo
> [{"lastTransitionTime":"2026-11-10T10:52:58Z","message":"Valid GatewayClass","observedGeneration":1,"reason":"Accepted","status":"True","type":"Accepted"}]
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n infrastructure get applications
> NAME                     SYNC STATUS   HEALTH STATUS
> argocd                   Synced        Healthy
> cloudnative-pg           Synced        Healthy
> envoy-gateway            Synced        Healthy
> gateway                  Synced        Healthy
> local-path-provisioner   Synced        Healthy
> metrics-server           Synced        Healthy
> root                     OutOfSync     Healthy
> sealed-secrets           Synced        Healthy
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n infrastructure get applicationsets
> No resources found in infrastructure namespace.
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n infrastructure get appprojects
> NAME      AGE
> default   123m
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl get storageclass
> NAME                      PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
> local-path (default)      rancher.io/local-path   Retain          WaitForFirstConsumer   false                  117m
> local-path-choimory-dev   rancher.io/local-path   Retain          WaitForFirstConsumer   false                  117m
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl get ns
> NAME              STATUS   AGE
> cilium-secrets    Active   20h
> default           Active   20h
> infrastructure    Active   123m
> kube-node-lease   Active   20h
> kube-public       Active   20h
> kube-system       Active   20h
> ```

## GatewayClass 원인 확정

- GatewayClass `Accepted`가 `True`로 바뀜. 갱신 시각 10:52:58은 컨트롤러 재시작 시점과 같음
- 앞에서 말한 가능성 중 1번이 맞음
  - 생성 순서 때문에 GatewayClass에 `False`가 남아 있었음
  - 재시작하면서 컨트롤러가 다시 계산해 `True`로 바뀜
  - 이전 확인에서는 명령을 너무 일찍 실행해 바뀌기 전 값을 본 것
- 클러스터를 재설치하면 같은 문제가 다시 생길 수 있으므로 재발 방지 annotation 추가를 권장

## wave 7 상태

| 항목 | 상태 |
|---|---|
| 공용 도구 wave 0~6 | 모두 `Synced`, `Healthy` |
| StorageClass | `local-path`(default), `local-path-choimory-dev` 정상 |
| wave 7 `choimory-dev` | 아직 생성 안 됨. ApplicationSet, AppProject `choimory-dev`, 네임스페이스도 없음 |
| `root` | `Healthy`인데 아직 `OutOfSync` |

- wave 7이 진행되지 않는 이유는 `root`의 이전 동기화가 실패로 끝났기 때문으로 추정. wave 6이 `Degraded`인 동안 동기화 작업이 실패로 끝났을 가능성이 높음
- ArgoCD의 자동 동기화는 같은 커밋에서 한 번 실패한 동기화를 스스로 다시 시도하지 않음(retry 설정이 없을 때). 그래서 `root`를 한 번 수동으로 동기화해야 함
- 이 추정은 아래 첫 번째 명령으로 확인

## root 동기화 명령어 (서버)

```bash
# root의 마지막 동기화 결과 확인
kubectl -n infrastructure get application root -o jsonpath='{.status.operationState.phase}{"\n"}{.status.operationState.message}{"\n"}'

# root 수동 동기화 (argocd CLI 없이 operation 필드로 요청)
kubectl -n infrastructure patch application root --type merge -p '{"operation":{"initiatedBy":{"username":"admin"},"sync":{"revision":"main"}}}'

# 1~2분 뒤 확인
kubectl -n infrastructure get applications
kubectl -n infrastructure get applicationsets
kubectl -n infrastructure get appprojects
kubectl get ns
```

- `operationState` 조회: 마지막 동기화의 단계(`Succeeded`, `Failed`, `Error`, `Running`)와 메시지를 출력. `Failed`나 `Error`면 추정이 맞고 메시지에 실패 이유가 나옴. `Running`이면 아직 진행 중이므로 patch 없이 대기
- `patch ... operation`: ArgoCD 화면의 Sync 버튼과 같은 효과. ArgoCD가 처리한 뒤 `operation` 필드는 자동으로 지워짐
- 확인 명령: `choimory-dev`와 Application 8개, ApplicationSet 2개, AppProject `choimory-dev`, 네임스페이스 `choimory-dev-stg`, `choimory-dev-prod`가 생성됐는지 확인

## 재발 방지 annotation (허락 요청)

- `infrastructure/gateway/gateway.yaml`의 EnvoyProxy `metadata`에 아래를 추가하는 것을 제안. 아직 적용하지 않음

```yaml
  annotations:
    # GatewayClass가 참조하므로 먼저 만들어야 한다. 같이 만들면 GatewayClass가 Accepted False로 남는다.
    argocd.argoproj.io/sync-wave: "-1"
```

- 이미 만들어진 리소스에는 영향이 없고, 다음에 새로 설치할 때 생성 순서만 바뀜
- 파일만 수정하고 커밋과 push는 직접 진행

---

# root 동기화 명령어 재안내

> ```
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n infrastructure get applications
> NAME                     SYNC STATUS   HEALTH STATUS
> argocd                   Synced        Healthy
> cloudnative-pg           Synced        Healthy
> envoy-gateway            Synced        Healthy
> gateway                  Synced        Healthy
> local-path-provisioner   Synced        Healthy
> metrics-server           Synced        Healthy
> root                     OutOfSync     Healthy
> sealed-secrets           Synced        Healthy
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n infrastructure get applicationsets
> No resources found in infrastructure namespace.
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n infrastructure get appprojects
> NAME      AGE
> default   124m
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl get storageclass
> NAME                      PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
> local-path (default)      rancher.io/local-path   Retain          WaitForFirstConsumer   false                  118m
> local-path-choimory-dev   rancher.io/local-path   Retain          WaitForFirstConsumer   false                  118m
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl get ns
> NAME              STATUS   AGE
> cilium-secrets    Active   20h
> default           Active   20h
> infrastructure    Active   125m
> kube-node-lease   Active   20h
> kube-public       Active   20h
> kube-system       Active   20h
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl get pods -A
> NAMESPACE        NAME                                                           READY   STATUS    RESTARTS      AGE
> infrastructure   argocd-application-controller-0                                1/1     Running   0             124m
> infrastructure   argocd-applicationset-controller-797985f4b7-nm5hl              1/1     Running   0             124m
> infrastructure   argocd-redis-5fc69b46c8-qbd7h                                  1/1     Running   0             124m
> infrastructure   argocd-repo-server-7b86c89c58-9tklm                            1/1     Running   0             124m
> infrastructure   argocd-server-6fb5747584-k2mwk                                 1/1     Running   0             124m
> infrastructure   cloudnative-pg-55d68ccd6c-cs9ft                                1/1     Running   0             117m
> infrastructure   envoy-gateway-86869b759c-5sfrr                                 1/1     Running   0             35m
> infrastructure   envoy-infrastructure-shared-gateway-02d9540f-74bdf9fb6-m5rcm   2/2     Running   0             116m
> infrastructure   local-path-provisioner-54745b98cf-6szg2                        1/1     Running   0             118m
> infrastructure   metrics-server-55c6b8755f-nwdgv                                1/1     Running   0             118m
> infrastructure   sealed-secrets-controller-5b56d98858-5g8xw                     1/1     Running   0             118m
> kube-system      cilium-envoy-f852f                                             1/1     Running   1 (20h ago)   20h
> kube-system      cilium-kzfj8                                                   1/1     Running   1 (20h ago)   20h
> kube-system      cilium-operator-6c4c574b46-pfvdj                               1/1     Running   1 (20h ago)   20h
> kube-system      coredns-589f44dc88-2s5xj                                       1/1     Running   1 (20h ago)   20h
> kube-system      coredns-589f44dc88-42qq7                                       1/1     Running   1 (20h ago)   20h
> kube-system      etcd-choimory-server                                           1/1     Running   1 (20h ago)   20h
> kube-system      kube-apiserver-choimory-server                                 1/1     Running   1 (20h ago)   20h
> kube-system      kube-controller-manager-choimory-server                        1/1     Running   1 (20h ago)   20h
> kube-system      kube-proxy-8dsgf                                               1/1     Running   1 (20h ago)   20h
> kube-system      kube-scheduler-choimory-server                                 1/1     Running   1 (20h ago)   20h
> ```

## 확인 결과

- 이전 확인 명령을 다시 실행한 결과로, `root` 동기화 결과 확인과 수동 동기화 명령은 아직 실행하지 않은 상태
- 상태는 그대로. `root`는 `OutOfSync`, `choimory-dev`의 ApplicationSet, AppProject, 네임스페이스는 아직 없음
- Pod는 모두 `Running`이라 공용 도구 쪽은 문제 없음
- `kube-proxy`는 Cilium에서 kube-proxy 대체 기능을 켜지 않았기 때문에 있는 것이라 정상

## 실행할 명령어 (서버)

```bash
# 1. root의 마지막 동기화 결과 확인
kubectl -n infrastructure get application root -o jsonpath='{.status.operationState.phase}{"\n"}{.status.operationState.message}{"\n"}'

# 2. 1번 결과가 Failed 또는 Error면 수동 동기화 (Running이면 생략하고 잠시 대기)
kubectl -n infrastructure patch application root --type merge -p '{"operation":{"initiatedBy":{"username":"admin"},"sync":{"revision":"main"}}}'

# 3. 1~2분 뒤 확인
kubectl -n infrastructure get applications
kubectl -n infrastructure get applicationsets
kubectl -n infrastructure get appprojects
kubectl get ns
```

- 각 명령의 역할은 위 [root 동기화 명령어 (서버)](#root-동기화-명령어-서버) 참고
- 재발 방지 annotation 수정 여부는 아직 답변 대기

---

# 5단계 완료

> ```
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n infrastructure get application root -o jsonpath='{.status.operationState.phase}{"\n"}{.status.operationState.message}{"\n"}'
> Failed
> one or more synchronization tasks completed unsuccessfully (retried 5 times).
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n infrastructure patch application root --type merge -p '{"operation":{"initiatedBy":{"username":"admin"},"sync":{"revision":"main"}}}'
> application.argoproj.io/root patched
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n infrastructure get applications
> ^C
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n infrastructure get applications
> NAME                           SYNC STATUS   HEALTH STATUS
> argocd                         Synced        Healthy
> choimory-dev                   Synced        Healthy
> choimory-dev-prod-data         Synced        Progressing
> choimory-dev-prod-front        Synced        Progressing
> choimory-dev-prod-member-api   Synced        Progressing
> choimory-dev-prod-namespaces   Synced        Healthy
> choimory-dev-stg-data          Synced        Progressing
> choimory-dev-stg-front         Synced        Progressing
> choimory-dev-stg-member-api    Synced        Progressing
> choimory-dev-stg-namespaces    Synced        Healthy
> cloudnative-pg                 Synced        Healthy
> envoy-gateway                  Synced        Healthy
> gateway                        Synced        Healthy
> local-path-provisioner         Synced        Healthy
> metrics-server                 Synced        Healthy
> root                           Synced        Healthy
> sealed-secrets                 Synced        Healthy
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n infrastructure get applicationsets
> NAME                     AGE
> choimory-dev-apps        114s
> choimory-dev-protected   114s
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n infrastructure get appprojects
> NAME           AGE
> choimory-dev   2m
> default        131m
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl get ns
> NAME                STATUS   AGE
> choimory-dev-prod   Active   2m3s
> choimory-dev-stg    Active   2m3s
> cilium-secrets      Active   20h
> default             Active   20h
> infrastructure      Active   131m
> kube-node-lease     Active   20h
> kube-public         Active   20h
> kube-system         Active   20h
> ```

## root 동기화 결과

- 5단계(ArgoCD 설치와 root 등록) 완료
- `root`의 이전 동기화는 `Failed`, `retried 5 times`. 재시도가 없었던 것이 아니라, wave 6이 `Degraded`인 동안 5번 재시도를 모두 쓰고 멈춘 것
- 수동 동기화 후 wave 7까지 모두 생성됨

| 항목 | 결과 |
|---|---|
| `root`, 공용 도구 7개, `choimory-dev` | 모두 `Synced`, `Healthy` |
| ApplicationSet | `choimory-dev-apps`, `choimory-dev-protected` |
| AppProject | `choimory-dev` |
| 네임스페이스 | `choimory-dev-stg`, `choimory-dev-prod` |
| `*-namespaces` 2개 | `Healthy` |
| `*-data`, `*-member-api`, `*-front` 6개 | `Progressing` |

- `Progressing` 6개는 예상한 상태. Secret(6단계)이 없고 이미지 태그가 `*-bootstrap` 임시값이기 때문

## 앱 상태 확인 (선택)

```bash
kubectl -n choimory-dev-stg get pods,pvc
kubectl -n choimory-dev-stg get events --sort-by=.lastTimestamp | tail -20
```

- 정상이라면 Pod가 `CreateContainerConfigError`(Secret 없음)나 `ImagePullBackOff`(임시 태그) 상태이고, 이벤트에 `secret "..." not found`가 나옴

## 다음 진행 선택

1. 재발 방지 annotation(EnvoyProxy `sync-wave: "-1"`) 추가
2. 6단계 Secret 등록
3. ArgoCD 화면 로그인 후 `argocd-initial-admin-secret` 삭제 (직접 진행)

---

# 재발 방지 annotation 적용과 6단계 계획

> 1. wㅐ발방지 어노테이션 추가 진행 
> 2. 진행해줘
> 3. 내가 할게

## 재발 방지 annotation 적용

- `infrastructure/gateway/gateway.yaml`의 EnvoyProxy에 추가함. 이미 만들어진 리소스에는 영향 없음. 커밋과 push는 직접 진행

```yaml
  annotations:
    # GatewayClass가 참조하므로 먼저 만들어야 한다. 같이 만들면 GatewayClass가 Accepted False로 남는다.
    argocd.argoproj.io/sync-wave: "-1"
```

## 필요한 Secret

- 환경(stg, prod)마다 2개씩, 모두 4개
- Secret 이름은 이후 [리소스 이름 변경 적용](#리소스-이름-변경-적용)에서 `choimory-dev-` 접두어를 붙인 이름으로 바뀜

| Secret | 키 | 쓰는 곳 | 둘 위치 |
|---|---|---|---|
| `redis-secret` | `password` | redis, member-api | `data/overlays/{env}/` |
| `member-api-secret` | `jwt-secret` | member-api | `member-api/overlays/{env}/` |
| `choimory-dev-pg-app` | - | member-api | CloudNativePG가 자동 생성하므로 만들 필요 없음 |

- `redis-secret`은 Redis를 관리하는 `data` 쪽에 둠. `data`는 prune이 꺼진 보호 대상이라 실수로 지워지지 않음. member-api는 같은 네임스페이스라 그대로 읽음
- `jwt-secret`은 member-api README 기준 HS256용, 32바이트 이상 필요. `openssl rand -hex 32`로 64자를 만들어 충족. Redis 비밀번호도 같은 방식. hex는 특수문자가 없어 Redis 실행 인자로 넘겨도 문제없음

## kubeseal 설치 (서버)

- kubeseal은 클러스터의 Sealed Secrets 컨트롤러와 버전을 맞춤. 먼저 컨트롤러 버전과 서버 아키텍처 확인

```bash
# 컨트롤러 이미지 버전과 서버 아키텍처 확인
kubectl -n infrastructure get deploy sealed-secrets-controller -o jsonpath='{.spec.template.spec.containers[0].image}'; echo
uname -m
```

- 이미지 태그의 숫자(예: `0.xx.x`)를 `KUBESEAL_VERSION`에 넣음. `uname -m`이 `aarch64`면 파일명의 `amd64`를 `arm64`로 변경

```bash
KUBESEAL_VERSION=<이미지 태그 숫자>
cd ~
curl -fsSLO "https://github.com/bitnami-labs/sealed-secrets/releases/download/v${KUBESEAL_VERSION}/kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz"
tar -xzf kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz kubeseal
sudo install -m 755 kubeseal /usr/local/bin/kubeseal
rm kubeseal kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz
kubeseal --version
kubeseal --controller-namespace infrastructure --fetch-cert > /dev/null && echo "cert ok"
```

- `curl`, `tar`: GitHub 릴리스에서 kubeseal 바이너리만 내려받아 압축 해제
- `install`: `/usr/local/bin`에 실행 권한과 함께 복사
- `--fetch-cert`: 컨트롤러의 공개 인증서를 가져올 수 있는지 확인. `cert ok`가 나오면 암호화 준비 완료

## 암호화 키 백업 (서버)

- 클러스터를 다시 설치하면 컨트롤러 키가 새로 만들어져, 레포의 SealedSecret을 하나도 풀 수 없게 됨
- 아래 파일은 평문 키이므로 레포에 절대 넣지 않고 별도 안전한 곳(서버 밖 백업, 비밀번호 관리자 등)에 보관

```bash
kubectl -n infrastructure get secret -l sealedsecrets.bitnami.com/sealed-secrets-key -o yaml > ~/sealed-secrets-key-backup.yaml
chmod 600 ~/sealed-secrets-key-backup.yaml
```

## Mac으로 가져오기

- Mac에서 실행. `<서버계정>`은 실제 계정으로 변경

```bash
mkdir -p ~/Downloads/sealed
scp '<서버계정>@choimory-server:~/sealed/*.yaml' ~/Downloads/sealed/
```

## 가져온 뒤 진행할 작업

1. 4개 파일에 `encryptedData`만 있고 평문 값이 없는지 확인
2. 각 overlay로 옮기고 파일 맨 위에 주석(용도, 다시 만드는 방법)을 붙임
3. 각 overlay의 `kustomization.yaml`에 `resources`로 추가

- 커밋과 push 후 ArgoCD가 동기화하면서 Secret이 만들어짐
- Redis, PostgreSQL은 정상으로 뜨고 `*-data`가 `Healthy`가 됨
- `member-api`, `front`는 이미지 태그가 아직 임시값이라 7단계 전 준비 작업 후에 해결됨
- SealedSecret 생성 명령은 이름 변경 후 버전을 사용. [SealedSecret 생성 (서버)](#sealedsecret-생성-서버) 참고

---

# 리소스 이름 변경 계획

> ```
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl get ns
> NAME                STATUS   AGE
> choimory-dev-prod   Active   2m3s
> choimory-dev-stg    Active   2m3s
> cilium-secrets      Active   20h
> default             Active   20h
> infrastructure      Active   131m
> kube-node-lease     Active   20h
> kube-public         Active   20h
> kube-system         Active   20h
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl get pods -n choimory-dev-stg
> NAME                          READY   STATUS                       RESTARTS   AGE
> choimory-dev-pg-1             1/1     Running                      0          2m21s
> front-7f59c4779b-cq5gf        0/1     ImagePullBackOff             0          3m17s
> member-api-8658b97fbf-5lgpp   0/1     ImagePullBackOff             0          3m17s
> redis-0                       0/1     CreateContainerConfigError   0          3m17s
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl get pods -n choimory-dev-prod
> NAME                          READY   STATUS                       RESTARTS   AGE
> choimory-dev-pg-1             1/1     Running                      0          2m25s
> front-6d5849bd-9d48j          0/1     ImagePullBackOff             0          3m22s
> member-api-69c799f747-25nhf   0/1     ImagePullBackOff             0          3m22s
> redis-0                       0/1     CreateContainerConfigError   0          3m22s
> ```
>
> 파드 이름들 바꿔줘 
> front: choimory-dev-front
> member-api: choimory-dev-member-api
> postgres: choimory-dev-postgres
> redis: choimory-dev-redis

## Pod 상태

- 예상한 그대로
  - postgres: `Running`
  - redis: `CreateContainerConfigError`. `redis-secret`이 아직 없음
  - front, member-api: `ImagePullBackOff`. 이미지 태그가 임시값
- SealedSecret을 만들기 전이고 데이터도 비어 있어 지금 이름을 바꾸는 것이 가장 쉬움

## 변경 대상

- Pod 이름은 상위 리소스 이름에서 정해지므로 아래 리소스 이름을 변경

| 대상 | 현재 | 변경 | 변경 후 Pod 이름 |
|---|---|---|---|
| front Deployment | `front` | `choimory-dev-front` | `choimory-dev-front-xxxx` |
| member-api Deployment | `member-api` | `choimory-dev-member-api` | `choimory-dev-member-api-xxxx` |
| CNPG Cluster | `choimory-dev-pg` | `choimory-dev-postgres` | `choimory-dev-postgres-1` |
| redis StatefulSet | `redis` | `choimory-dev-redis` | `choimory-dev-redis-0` |

- 같이 바뀌는 것
  - Service와 HTTPRoute 이름, `app` 라벨, 컨테이너 이름(추천: 같은 이름으로 맞춤)
  - CNPG가 Cluster 이름으로 Secret과 Service를 만들므로 `choimory-dev-pg-app` → `choimory-dev-postgres-app`. member-api가 읽는 Secret 이름 수정 필요
  - member-api의 `redis-host`, front의 `MEMBER_API_BASE_URL`
  - stg, prod overlay 패치의 `metadata.name`과 컨테이너 이름, 관련 주석
- 바뀌지 않는 것
  - 디렉토리 경로, 이미지 이름, ArgoCD Application 이름
  - 앱 레포 Actions: `overlays/{env}`에서 `kustomize edit set image`로 이미지 이름 기준으로 수정하므로 영향 없음
- push 후 `data`는 prune이 꺼져 있어 예전 Cluster, StatefulSet이 남으므로 직접 삭제 필요

## 정할 것

1. 변경 범위: Service, HTTPRoute, 라벨, 컨테이너 이름까지 같이 바꿀지, Pod 이름에 해당하는 리소스만 바꿀지
2. Secret 이름: `redis-secret`, `member-api-secret`도 `choimory-dev-` 접두어로 맞출지. kubeseal 명령의 이름도 바뀌므로 Secret 생성 전에 결정 필요

---

# 리소스 이름 변경 적용

> 1. 추천대로 2. 시크릿 이름도 맞춰줘

## 변경 내용

- 6개 overlay(`data`, `member-api`, `front` × stg, prod) 모두 `kubectl kustomize`로 정상 렌더링 확인

| 대상 | 변경 전 → 변경 후 |
|---|---|
| front Deployment, Service, HTTPRoute, 라벨, 컨테이너 | `front` → `choimory-dev-front` |
| member-api Deployment, Service, HTTPRoute, 라벨, 컨테이너 | `member-api` → `choimory-dev-member-api` |
| CNPG Cluster | `choimory-dev-pg` → `choimory-dev-postgres` |
| redis StatefulSet, Service, 라벨, 컨테이너 | `redis` → `choimory-dev-redis` |
| member-api가 읽는 DB Secret | `choimory-dev-pg-app` → `choimory-dev-postgres-app`(CNPG가 자동 생성) |
| member-api의 `redis-host` 값 | `redis` → `choimory-dev-redis` |
| front의 `MEMBER_API_BASE_URL` | `http://member-api:8080` → `http://choimory-dev-member-api:8080` |
| Secret 이름 | `redis-secret` → `choimory-dev-redis-secret`, `member-api-secret` → `choimory-dev-member-api-secret` |

- stg, prod overlay 패치의 이름도 같이 변경
- 이름이 들어간 주석도 수정(postgres Secret/Service 이름, redis PVC 경로)
- 바꾸지 않은 것
  - Redis 포트 이름 `redis`(리소스 이름이 아니라 포트 이름)
  - 디렉토리 경로, 이미지 이름, ArgoCD Application 이름
  - "member-api를 호출하는 주소"처럼 컴포넌트를 가리키는 주석
- EnvoyProxy annotation과 함께 아직 커밋하지 않은 상태

## push 후 예전 리소스 정리 (서버)

- front, member-api의 예전 리소스는 ArgoCD가 자동으로 지움(prune)
- `data`는 prune이 꺼져 있어 예전 리소스가 새 리소스와 함께 남으므로 직접 삭제

```bash
for ns in choimory-dev-stg choimory-dev-prod; do
  kubectl -n $ns delete cluster choimory-dev-pg
  kubectl -n $ns delete statefulset redis
  kubectl -n $ns delete service redis
  kubectl -n $ns delete pvc data-redis-0
done
kubectl -n choimory-dev-stg get pods,pvc
kubectl -n choimory-dev-prod get pods,pvc
```

- `delete cluster`: CNPG가 Pod와 PVC(`choimory-dev-pg-1`)를 같이 지움
- `delete statefulset`, `delete service`, `delete pvc`: 예전 redis 리소스 삭제. StatefulSet의 PVC는 자동으로 지워지지 않아 따로 삭제
- `get pods,pvc`: 새 이름의 리소스만 남았는지 확인

- 예전 PVC를 지워도 local-path가 `Retain`이라 디스크 폴더는 남음. 먼저 목록만 확인하고 지울 폴더를 정함

```bash
ls -la /data/dev/choimory-dev/k8s/*/
```

## SealedSecret 생성 (서버)

- 바뀐 Secret 이름 반영. kubeseal 설치, 키 백업, Mac으로 가져오기는 [재발 방지 annotation 적용과 6단계 계획](#재발-방지-annotation-적용과-6단계-계획)과 같음

```bash
mkdir -p ~/sealed && cd ~/sealed
for env in stg prod; do
  ns=choimory-dev-$env
  kubectl create secret generic choimory-dev-redis-secret -n $ns \
    --from-literal=password="$(openssl rand -hex 32)" \
    --dry-run=client -o yaml \
  | kubeseal --controller-namespace infrastructure --format yaml > choimory-dev-redis-secret-$env.yaml
  kubectl create secret generic choimory-dev-member-api-secret -n $ns \
    --from-literal=jwt-secret="$(openssl rand -hex 32)" \
    --dry-run=client -o yaml \
  | kubeseal --controller-namespace infrastructure --format yaml > choimory-dev-member-api-secret-$env.yaml
done
ls -l
```

- `openssl rand -hex 32`: 무작위 값 64자 생성. 원래 값은 화면이나 파일에 남지 않음. 필요하면 나중에 `kubectl get secret`으로 클러스터에서 확인
- `kubectl create secret ... --dry-run=client -o yaml`: 클러스터에 만들지 않고 Secret YAML만 출력
- `kubeseal --controller-namespace infrastructure`: 컨트롤러 공개키로 암호화해 SealedSecret YAML로 변환. 네임스페이스와 이름에 묶여 암호화되어 다른 곳에서는 풀리지 않으며 레포에 올려도 안전
- 이름과 네임스페이스만 맞으면 되므로, 이름 변경을 push하기 전에 만들어도 상관없음

## 진행 순서

1. 이름 변경 커밋, push (직접 진행)
2. 예전 리소스 정리
3. kubeseal 설치, SealedSecret 생성, 키 백업, Mac으로 가져오기
4. `~/Downloads/sealed`에 파일 4개가 생기면 각 overlay에 추가

---

# 이름 변경 후 진행 순서

> 이제 뭐해야하지

## 1. 커밋과 push (Mac)

- 커밋하지 않은 변경
  - `infrastructure/gateway/gateway.yaml`: EnvoyProxy annotation
  - `projects/choimory-dev/` 아래 13개 파일: 리소스 이름 변경
  - `.agents/histories/2026/11/06/k8s-setup/task5.md`
- ArgoCD는 `main`을 읽으므로, `dev`에 커밋했다면 `main`에 머지한 뒤 push까지 해야 서버에 반영됨

## 2. 반영 확인과 예전 리소스 정리 (서버)

- push 후 1~3분 기다렸다가 실행

```bash
kubectl -n infrastructure get applications | grep choimory-dev

for ns in choimory-dev-stg choimory-dev-prod; do
  kubectl -n $ns delete cluster choimory-dev-pg
  kubectl -n $ns delete statefulset redis
  kubectl -n $ns delete service redis
  kubectl -n $ns delete pvc data-redis-0
done
kubectl -n choimory-dev-stg get pods,pvc
kubectl -n choimory-dev-prod get pods,pvc
ls -la /data/dev/choimory-dev/k8s/*/
```

- 정상이라면
  - Pod가 `choimory-dev-postgres-1`, `choimory-dev-redis-0`, `choimory-dev-member-api-*`, `choimory-dev-front-*`만 남음
  - redis는 Secret이 없어 계속 `CreateContainerConfigError`, front와 member-api는 계속 `ImagePullBackOff`

## 3. SealedSecret 만들기 (서버, Mac)

1. [kubeseal 설치 (서버)](#kubeseal-설치-서버): 컨트롤러 버전과 아키텍처를 확인한 뒤 설치
2. [SealedSecret 생성 (서버)](#sealedsecret-생성-서버): 바뀐 Secret 이름이 반영된 명령
3. [암호화 키 백업 (서버)](#암호화-키-백업-서버): 백업 파일은 레포 밖에 보관
4. [Mac으로 가져오기](#mac으로-가져오기): `~/Downloads/sealed`로 가져옴

## 4. 알려 줄 것

- 2번의 `get pods,pvc` 결과와 `ls` 결과. 지울 예전 폴더를 정함
- `~/Downloads/sealed`에 파일 4개가 생긴 것. 평문 값이 없는지 확인한 뒤 각 overlay에 넣고 `kustomization.yaml`에 추가
- ArgoCD 로그인과 `argocd-initial-admin-secret` 삭제는 중간에 편할 때 진행

---

# 이름 변경 반영 확인

> ```
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl get pod -n choimory-dev-stg
> NAME                                       READY   STATUS                       RESTARTS   AGE
> choimory-dev-front-85469c8fb5-klvlp        0/1     ImagePullBackOff             0          33m
> choimory-dev-member-api-69cd5894dd-vcmbc   0/1     ImagePullBackOff             0          32m
> choimory-dev-pg-1                          1/1     Running                      0          109m
> choimory-dev-postgres-1                    1/1     Running                      0          33m
> choimory-dev-redis-0                       0/1     CreateContainerConfigError   0          33m
> redis-0                                    0/1     CreateContainerConfigError   0          110m
> ```

## 반영 상태

- 이름 변경 반영됨. front와 member-api의 예전 리소스는 ArgoCD가 자동으로 지웠고, 새 이름의 Pod 4개가 떠 있음
- 상태는 예상대로
  - postgres: `Running`
  - redis: `CreateContainerConfigError`. Secret이 아직 없음
  - front, member-api: `ImagePullBackOff`. 이미지 태그가 임시값
- `choimory-dev-pg-1`, `redis-0`이 남은 것도 예상한 상황. `data`는 prune이 꺼져 있어 직접 삭제 필요

## 예전 리소스 정리 명령어 (서버)

```bash
for ns in choimory-dev-stg choimory-dev-prod; do
  kubectl -n $ns delete cluster choimory-dev-pg
  kubectl -n $ns delete statefulset redis
  kubectl -n $ns delete service redis
  kubectl -n $ns delete pvc data-redis-0
done
kubectl -n choimory-dev-stg get pods,pvc
kubectl -n choimory-dev-prod get pods,pvc
ls -la /data/dev/choimory-dev/k8s/*/
```

- 정상이라면
  - Pod: `choimory-dev-*` 4개만 남음
  - PVC: `choimory-dev-postgres-1`, `data-choimory-dev-redis-0`만 남음
  - `ls`: 예전 폴더(`choimory-dev-pg-1`, `data-redis-0`)와 새 폴더가 같이 보임. 예전 폴더는 local-path가 `Retain`이라 디스크에 남은 것

---

# 예전 PV와 폴더 정리

> ```
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$   for ns in choimory-dev-stg choimory-dev-prod; do
>     kubectl -n $ns delete cluster choimory-dev-pg
>     kubectl -n $ns delete statefulset redis
>     kubectl -n $ns delete service redis
>     kubectl -n $ns delete pvc data-redis-0
>   done
> cluster.postgresql.cnpg.io "choimory-dev-pg" deleted from choimory-dev-stg namespace
> statefulset.apps "redis" deleted from choimory-dev-stg namespace
> service "redis" deleted from choimory-dev-stg namespace
> persistentvolumeclaim "data-redis-0" deleted from choimory-dev-stg namespace
> cluster.postgresql.cnpg.io "choimory-dev-pg" deleted from choimory-dev-prod namespace
> statefulset.apps "redis" deleted from choimory-dev-prod namespace
> service "redis" deleted from choimory-dev-prod namespace
> persistentvolumeclaim "data-redis-0" deleted from choimory-dev-prod namespace
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$   kubectl -n choimory-dev-stg get pods,pvc
> NAME                                           READY   STATUS                       RESTARTS   AGE
> pod/choimory-dev-front-85469c8fb5-klvlp        0/1     ImagePullBackOff             0          34m
> pod/choimory-dev-member-api-69cd5894dd-vcmbc   0/1     ImagePullBackOff             0          33m
> pod/choimory-dev-pg-1                          1/1     Terminating                  0          110m
> pod/choimory-dev-postgres-1                    1/1     Running                      0          34m
> pod/choimory-dev-redis-0                       0/1     CreateContainerConfigError   0          34m
>
> NAME                                              STATUS        VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS              VOLUMEATTRIBUTESCLASS   AGE
> persistentvolumeclaim/choimory-dev-pg-1           Terminating   pvc-bf9b5aad-47e6-48ca-8008-13284ea9b7aa   5Gi        RWO            local-path-choimory-dev   <unset>                 111m
> persistentvolumeclaim/choimory-dev-postgres-1     Bound         pvc-5581332e-cda2-4ac1-b40a-3ead8f7adbdb   5Gi        RWO            local-path-choimory-dev   <unset>                 34m
> persistentvolumeclaim/data-choimory-dev-redis-0   Bound         pvc-e7f681ab-d246-4fcc-9771-d11151e81045   1Gi        RWO            local-path-choimory-dev   <unset>                 34m
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$   kubectl -n choimory-dev-prod get pods,pvc
> NAME                                          READY   STATUS                       RESTARTS   AGE
> pod/choimory-dev-front-56889d449f-x6z6t       0/1     ImagePullBackOff             0          33m
> pod/choimory-dev-member-api-dd677d978-pqfk6   0/1     ImagePullBackOff             0          34m
> pod/choimory-dev-postgres-1                   1/1     Running                      0          34m
> pod/choimory-dev-redis-0                      0/1     CreateContainerConfigError   0          34m
>
> NAME                                              STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS              VOLUMEATTRIBUTESCLASS   AGE
> persistentvolumeclaim/choimory-dev-postgres-1     Bound    pvc-7c1d4bcb-dbe1-493c-afb4-823b415f58f8   10Gi       RWO            local-path-choimory-dev   <unset>                 34m
> persistentvolumeclaim/data-choimory-dev-redis-0   Bound    pvc-9c9e696f-76c0-4e3a-81ad-ca8b668e225e   1Gi        RWO            local-path-choimory-dev   <unset>                 34m
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$   ls -la /data/dev/choimory-dev/k8s/*/
> /data/dev/choimory-dev/k8s/choimory-dev-prod/:
> total 24
> drwxr-sr-x 6 root     homeserver 4096 Nov 10 21:49 .
> drwxrwsr-x 4 choimory homeserver 4096 Nov 10 20:32 ..
> drwxrwxrwx 3 root     homeserver 4096 Nov 10 20:33 choimory-dev-pg-1
> drwxrwxrwx 3 root     homeserver 4096 Nov 10 21:49 choimory-dev-postgres-1
> drwxrwxrwx 2 root     homeserver 4096 Nov 10 21:49 data-choimory-dev-redis-0
> drwxrwxrwx 2 root     homeserver 4096 Nov 10 20:32 data-redis-0
>
> /data/dev/choimory-dev/k8s/choimory-dev-stg/:
> total 24
> drwxr-sr-x 6 root     homeserver 4096 Nov 10 21:49 .
> drwxrwsr-x 4 choimory homeserver 4096 Nov 10 20:32 ..
> drwxrwxrwx 3 root     homeserver 4096 Nov 10 20:33 choimory-dev-pg-1
> drwxrwxrwx 3 root     homeserver 4096 Nov 10 21:49 choimory-dev-postgres-1
> drwxrwxrwx 2 root     homeserver 4096 Nov 10 21:49 data-choimory-dev-redis-0
> drwxrwxrwx 2 root     homeserver 4096 Nov 10 20:32 data-redis-0
> ```

## 정리 결과

- 예전 리소스 정리 완료. stg의 `choimory-dev-pg-1`이 `Terminating`인 것은 CNPG가 지우는 중이라 잠시 후 사라짐
- 새 리소스(`choimory-dev-postgres-1`, `data-choimory-dev-redis-0`)는 두 환경 모두 `Bound`
- local-path가 `Retain`이라 PVC를 지워도 아래 두 가지가 남음
  1. PV 객체: `Released` 상태로 남음
  2. 디스크 폴더: `choimory-dev-pg-1`, `data-redis-0`이 환경별로 남음. 모두 4개

## PV와 폴더 삭제 명령어 (서버)

```bash
# 1. stg Pod 삭제 완료 대기 후, Released PV 확인
kubectl -n choimory-dev-stg wait --for=delete pod/choimory-dev-pg-1 --timeout=2m
kubectl get pv | grep -E 'Released|NAME'
```

- `CLAIM` 열이 `choimory-dev-{stg,prod}/choimory-dev-pg-1`, `choimory-dev-{stg,prod}/data-redis-0`인 PV 4개만 있는지 확인한 뒤 삭제

```bash
# 2. 예전 PV 4개 삭제 (CLAIM이 예전 이름인 것만)
kubectl get pv -o jsonpath='{range .items[?(@.status.phase=="Released")]}{.metadata.name}{"\t"}{.spec.claimRef.namespace}/{.spec.claimRef.name}{"\n"}{end}' \
  | grep -E '/(choimory-dev-pg-1|data-redis-0)$' | cut -f1 | xargs -r kubectl delete pv

# 3. 예전 폴더 4개 삭제 (경로를 직접 지정)
sudo rm -rf \
  /data/dev/choimory-dev/k8s/choimory-dev-stg/choimory-dev-pg-1 \
  /data/dev/choimory-dev/k8s/choimory-dev-stg/data-redis-0 \
  /data/dev/choimory-dev/k8s/choimory-dev-prod/choimory-dev-pg-1 \
  /data/dev/choimory-dev/k8s/choimory-dev-prod/data-redis-0

# 4. 확인
kubectl get pv
ls -la /data/dev/choimory-dev/k8s/*/
```

- `wait --for=delete`: CNPG가 예전 Pod를 다 지울 때까지 대기
- 2번: `Released` 상태이면서 예전 PVC를 가리키던 PV만 골라 삭제. 새 PV는 `Bound`라 대상이 아님
- 3번: 폴더 소유자가 root라 `sudo` 필요. 지우는 것은 빈 테스트 DB 폴더뿐이고, 경로를 하나씩 직접 지정해 다른 폴더가 지워지지 않게 함
- 정상이라면 `kubectl get pv`에 `Bound` PV 4개만 남고, 각 네임스페이스 폴더에 `choimory-dev-postgres-1`, `data-choimory-dev-redis-0`만 남음

## kubeseal 설치 전 확인 (서버)

```bash
kubectl -n infrastructure get deploy sealed-secrets-controller -o jsonpath='{.spec.template.spec.containers[0].image}'; echo
uname -m
```

- 컨트롤러 이미지 태그로 설치할 kubeseal 버전을, `uname -m`으로 내려받을 파일의 아키텍처를 정함

---

# PV와 폴더 정리 완료

> ```
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n choimory-dev-stg wait --for=delete pod/choimory-dev-pg-1 --timeout=2m
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl get pv | grep -E 'Released|NAME'
> NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM                                         STORAGECLASS              VOLUMEATTRIBUTESCLASS   REASON   AGE
> pvc-8e82dfde-6c2c-4a1d-ad78-9520ba9096a3   1Gi        RWO            Retain           Released   choimory-dev-stg/data-redis-0                 local-path-choimory-dev   <unset>                          112m
> pvc-b8d1e2bb-0b77-4359-8b41-789947b7f223   1Gi        RWO            Retain           Released   choimory-dev-prod/data-redis-0                local-path-choimory-dev   <unset>                          112m
> pvc-bf9b5aad-47e6-48ca-8008-13284ea9b7aa   5Gi        RWO            Retain           Released   choimory-dev-stg/choimory-dev-pg-1            local-path-choimory-dev   <unset>                          112m
> pvc-c2d7f348-795c-443d-bbbb-549d9dff5c94   10Gi       RWO            Retain           Released   choimory-dev-prod/choimory-dev-pg-1           local-path-choimory-dev   <unset>                          112m
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl get pv -o jsonpath='{range .items[?(@.status.phase=="Released")]}{.metadata.name}{"\t"}{.spec.claimRef.namespace}/{.spec.claimRef.name}{"\n"}{end}' \
>   | grep -E '/(choimory-dev-pg-1|data-redis-0)$' | cut -f1 | xargs -r kubectl delete pv
> persistentvolume "pvc-8e82dfde-6c2c-4a1d-ad78-9520ba9096a3" deleted
> persistentvolume "pvc-b8d1e2bb-0b77-4359-8b41-789947b7f223" deleted
> persistentvolume "pvc-bf9b5aad-47e6-48ca-8008-13284ea9b7aa" deleted
> persistentvolume "pvc-c2d7f348-795c-443d-bbbb-549d9dff5c94" deleted
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ sudo rm -rf \
>   /data/dev/choimory-dev/k8s/choimory-dev-stg/choimory-dev-pg-1 \
>   /data/dev/choimory-dev/k8s/choimory-dev-stg/data-redis-0 \
>   /data/dev/choimory-dev/k8s/choimory-dev-prod/choimory-dev-pg-1 \
>   /data/dev/choimory-dev/k8s/choimory-dev-prod/data-redis-0
> [sudo: authenticate] Password:       
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl get pv
> NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                         STORAGECLASS              VOLUMEATTRIBUTESCLASS   REASON   AGE
> pvc-5581332e-cda2-4ac1-b40a-3ead8f7adbdb   5Gi        RWO            Retain           Bound    choimory-dev-stg/choimory-dev-postgres-1      local-path-choimory-dev   <unset>                          36m
> pvc-7c1d4bcb-dbe1-493c-afb4-823b415f58f8   10Gi       RWO            Retain           Bound    choimory-dev-prod/choimory-dev-postgres-1     local-path-choimory-dev   <unset>                          35m
> pvc-9c9e696f-76c0-4e3a-81ad-ca8b668e225e   1Gi        RWO            Retain           Bound    choimory-dev-prod/data-choimory-dev-redis-0   local-path-choimory-dev   <unset>                          35m
> pvc-e7f681ab-d246-4fcc-9771-d11151e81045   1Gi        RWO            Retain           Bound    choimory-dev-stg/data-choimory-dev-redis-0    local-path-choimory-dev   <unset>                          36m
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ ls -la /data/dev/choimory-dev/k8s/*/
> /data/dev/choimory-dev/k8s/choimory-dev-prod/:
> total 16
> drwxr-sr-x 4 root     homeserver 4096 Nov 10 22:25 .
> drwxrwsr-x 4 choimory homeserver 4096 Nov 10 20:32 ..
> drwxrwxrwx 3 root     homeserver 4096 Nov 10 21:49 choimory-dev-postgres-1
> drwxrwxrwx 2 root     homeserver 4096 Nov 10 21:49 data-choimory-dev-redis-0
>
> /data/dev/choimory-dev/k8s/choimory-dev-stg/:
> total 16
> drwxr-sr-x 4 root     homeserver 4096 Nov 10 22:25 .
> drwxrwsr-x 4 choimory homeserver 4096 Nov 10 20:32 ..
> drwxrwxrwx 3 root     homeserver 4096 Nov 10 21:49 choimory-dev-postgres-1
> drwxrwxrwx 2 root     homeserver 4096 Nov 10 21:49 data-choimory-dev-redis-0
> ```

## 정리 확인

- PV: 새 이름의 `Bound` PV 4개만 남음
- 디스크 폴더: 각 네임스페이스에 `choimory-dev-postgres-1`, `data-choimory-dev-redis-0`만 남음
- [아직 검증하지 못한 것](#아직-검증하지-못한-것) 중 PVC 폴더 생성도 확인됨. PVC를 만들면 `/data/dev/choimory-dev/k8s/<네임스페이스>/<PVC 이름>/` 폴더가 생성됨

## 다음 단계

- kubeseal 설치 전 컨트롤러 버전과 서버 아키텍처 확인

```bash
kubectl -n infrastructure get deploy sealed-secrets-controller -o jsonpath='{.spec.template.spec.containers[0].image}'; echo
uname -m
```

---

# kubeseal 0.41.2 설치와 SealedSecret 생성

> ```
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n infrastructure get deploy sealed-secrets-controller -o jsonpath='{.spec.template.spec.containers[0].image}'; echo
> docker.io/bitnami/sealed-secrets-controller:0.41.2
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ uname -m
> x86_64
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ 
> ```

- 컨트롤러 `0.41.2`, 서버 `x86_64`(amd64). 아래 명령을 순서대로 실행

## 1. kubeseal 설치 (서버)

```bash
KUBESEAL_VERSION=0.41.2
cd ~
curl -fsSLO "https://github.com/bitnami-labs/sealed-secrets/releases/download/v${KUBESEAL_VERSION}/kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz"
tar -xzf kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz kubeseal
sudo install -m 755 kubeseal /usr/local/bin/kubeseal
rm kubeseal kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz
kubeseal --version
kubeseal --controller-namespace infrastructure --fetch-cert > /dev/null && echo "cert ok"
```

- 정상이라면 `kubeseal version: 0.41.2`와 `cert ok`가 출력됨
- 명령별 역할은 [kubeseal 설치 (서버)](#kubeseal-설치-서버) 참고

## 2. SealedSecret 생성 (서버)

```bash
mkdir -p ~/sealed && cd ~/sealed
for env in stg prod; do
  ns=choimory-dev-$env
  kubectl create secret generic choimory-dev-redis-secret -n $ns \
    --from-literal=password="$(openssl rand -hex 32)" \
    --dry-run=client -o yaml \
  | kubeseal --controller-namespace infrastructure --format yaml > choimory-dev-redis-secret-$env.yaml
  kubectl create secret generic choimory-dev-member-api-secret -n $ns \
    --from-literal=jwt-secret="$(openssl rand -hex 32)" \
    --dry-run=client -o yaml \
  | kubeseal --controller-namespace infrastructure --format yaml > choimory-dev-member-api-secret-$env.yaml
done
ls -l
```

- 정상이라면 크기가 0이 아닌 파일 4개가 생성됨
- 명령별 역할은 [SealedSecret 생성 (서버)](#sealedsecret-생성-서버) 참고

## 3. 암호화 키 백업 (서버)

```bash
kubectl -n infrastructure get secret -l sealedsecrets.bitnami.com/sealed-secrets-key -o yaml > ~/sealed-secrets-key-backup.yaml
chmod 600 ~/sealed-secrets-key-backup.yaml
```

- 평문 키이므로 레포에 넣지 않고 서버 밖 안전한 곳에 따로 보관

## 4. Mac으로 가져오기 (Mac)

- `<서버계정>`은 실제 계정으로 변경

```bash
mkdir -p ~/Downloads/sealed
scp '<서버계정>@choimory-server:~/sealed/*.yaml' ~/Downloads/sealed/
```

- `~/Downloads/sealed`에 파일 4개가 생기면 평문 값이 없는지 확인한 뒤 각 overlay에 넣고 `kustomization.yaml`에 추가

---

# SealedSecret 생성 결과

> ```
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ KUBESEAL_VERSION=0.41.2
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ cd ~
> choimory@choimory-server:~$ curl -fsSLO "https://github.com/bitnami-labs/sealed-secrets/releases/download/v${KUBESEAL_VERSION}/kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz"
> choimory@choimory-server:~$ tar -xzf kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz kubeseal
> choimory@choimory-server:~$ sudo install -m 755 kubeseal /usr/local/bin/kubeseal
> [sudo: authenticate] Password:       
> choimory@choimory-server:~$ rm kubeseal kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz
> choimory@choimory-server:~$ kubeseal --version
> kubeseal version: 0.41.2
> choimory@choimory-server:~$ kubeseal --controller-namespace infrastructure --fetch-cert > /dev/null && echo "cert ok"
> cert ok
> choimory@choimory-server:~$ mkdir -p ~/sealed && cd ~/sealed
> choimory@choimory-server:~/sealed$ for env in stg prod; do
>   ns=choimory-dev-$env
>   kubectl create secret generic choimory-dev-redis-secret -n $ns \
>     --from-literal=password="$(openssl rand -hex 32)" \
>     --dry-run=client -o yaml \
>   | kubeseal --controller-namespace infrastructure --format yaml > choimory-dev-redis-secret-$env.yaml
>   kubectl create secret generic choimory-dev-member-api-secret -n $ns \
>     --from-literal=jwt-secret="$(openssl rand -hex 32)" \
>     --dry-run=client -o yaml \
>   | kubeseal --controller-namespace infrastructure --format yaml > choimory-dev-member-api-secret-$env.yaml
> done
> choimory@choimory-server:~/sealed$ ls -l
> total 16
> -rw-rw-r-- 1 choimory choimory 1072 Nov 11 00:21 choimory-dev-member-api-secret-prod.yaml
> -rw-rw-r-- 1 choimory choimory 1070 Nov 11 00:21 choimory-dev-member-api-secret-stg.yaml
> -rw-rw-r-- 1 choimory choimory 1060 Nov 11 00:21 choimory-dev-redis-secret-prod.yaml
> -rw-rw-r-- 1 choimory choimory 1058 Nov 11 00:21 choimory-dev-redis-secret-stg.yaml
> choimory@choimory-server:~/sealed$ kubectl -n infrastructure get secret -l sealedsecrets.bitnami.com/sealed-secrets-key -o yaml > ~/sealed-secrets-key-backup.yaml
> choimory@choimory-server:~/sealed$ chmod 600 ~/sealed-secrets-key-backup.yaml
> ```

## 서버 작업 결과

- kubeseal `0.41.2` 설치 완료, `cert ok` 확인
- SealedSecret 파일 4개 생성. 각 1KB 정도로 정상
- 암호화 키 백업 완료. `~/sealed-secrets-key-backup.yaml`은 평문 키이므로 서버 밖 안전한 곳에도 복사해 둘 것

## Mac으로 가져오기 명령어

- Mac에서 실행. `<서버계정>`은 실제 계정으로 변경

```bash
mkdir -p ~/Downloads/sealed
scp '<서버계정>@choimory-server:~/sealed/*.yaml' ~/Downloads/sealed/
```

- 가져온 뒤 진행할 작업
  1. 4개 파일에 `encryptedData`만 있는지, 평문 값이 없는지 확인
  2. `data/overlays/{stg,prod}/choimory-dev-redis-secret.yaml`, `member-api/overlays/{stg,prod}/choimory-dev-member-api-secret.yaml`로 옮기고 파일 맨 위에 주석을 붙임
  3. 각 overlay의 `kustomization.yaml`에 `resources`로 추가하고 `kubectl kustomize`로 렌더링 확인

---

# SealedSecret 레포 반영

> ```
> ╭─choimory@<DEVICE_NAME> ~ 
> ╰─$ mkdir -p ~/Downloads/sealed
>
> ╭─choimory@<DEVICE_NAME> ~ 
> ╰─$ scp 'choimory@choimory-server:~/sealed/*.yaml' ~/Downloads/sealed/  
>
> choimory@choimory-server's password: 
> choimory-dev-member-api-secret-prod.yaml                                                                                                                                                                                                                    100% 1072    25.6KB/s   00:00    
> choimory-dev-member-api-secret-stg.yaml                                                                                                                                                                                                                     100% 1070    33.9KB/s   00:00    
> choimory-dev-redis-secret-prod.yaml                                                                                                                                                                                                                         100% 1060    18.7KB/s   00:00    
> choimory-dev-redis-secret-stg.yaml                                                                                                                                                                                                                          100% 1058    33.9KB/s   00:00    
> ╭─choimory@<DEVICE_NAME> ~ 
> ╰─$ cd ~/Downloads                                                       
> ╭─choimory@<DEVICE_NAME> ~/Downloads 
> ╰─$ ls -l
> total 272
> -rw-r--r--@ 1 choimory  staff   21147  6월 21  2025 images_(38).jpg
> drwxr-xr-x@ 6 choimory  staff     192 11월 11 00:22 sealed
> -rw-r--r--@ 1 choimory  staff  114132  6월 21  2025 unnamed.jpg
> ╭─choimory@<DEVICE_NAME> ~/Downloads 
> ╰─$ cd sealed     
> ╭─choimory@<DEVICE_NAME> ~/Downloads/sealed 
> ╰─$ ls -l
> total 32
> -rw-r--r--@ 1 choimory  staff  1072 11월 11 00:22 choimory-dev-member-api-secret-prod.yaml
> -rw-r--r--@ 1 choimory  staff  1070 11월 11 00:22 choimory-dev-member-api-secret-stg.yaml
> -rw-r--r--@ 1 choimory  staff  1060 11월 11 00:22 choimory-dev-redis-secret-prod.yaml
> -rw-r--r--@ 1 choimory  staff  1058 11월 11 00:22 choimory-dev-redis-secret-stg.yaml
> ```

## 파일 확인

- 4개 파일 모두 `encryptedData`만 있고 평문 값 없음. 이름과 네임스페이스도 맞음
- `kubectl kustomize`로 4개 overlay 모두 정상 렌더링, 각 SealedSecret이 올바른 네임스페이스로 출력됨
- AppProject `choimory-dev`는 네임스페이스 안의 리소스 종류를 제한하지 않으므로 SealedSecret이 막히지 않음

## 추가한 파일

| 파일 | 내용 |
|---|---|
| `data/overlays/{stg,prod}/choimory-dev-redis-secret.yaml` | Redis 비밀번호 |
| `member-api/overlays/{stg,prod}/choimory-dev-member-api-secret.yaml` | JWT 서명 키 |
| 각 overlay의 `kustomization.yaml` | 위 파일을 `resources`에 추가 |

- 파일 맨 위에 용도, 이 네임스페이스와 이름에서만 풀린다는 점, 다시 만드는 방법을 주석으로 적음
- 값을 바꿀 때의 영향도 주석으로 남김
  - Redis 비밀번호: 바꾸면 redis와 member-api 재시작 필요
  - JWT 키: 바꾸면 기존 로그인 토큰이 모두 무효
- kubeseal 출력 맨 앞의 `---`는 제거

## 커밋 후 반영 확인 (서버)

- SealedSecret 4개와 kustomization 4개를 커밋하고 `main`까지 push(직접 진행). push 후 1~3분 뒤 실행

```bash
kubectl get sealedsecrets -n choimory-dev-stg
kubectl get sealedsecrets -n choimory-dev-prod
kubectl get secret -n choimory-dev-stg | grep choimory-dev
kubectl get secret -n choimory-dev-prod | grep choimory-dev
kubectl -n choimory-dev-stg get pods
kubectl -n choimory-dev-prod get pods
kubectl -n infrastructure get applications | grep choimory-dev
```

- `get sealedsecrets`: SealedSecret이 생성되고 컨트롤러가 처리했는지 확인
- `get secret`: SealedSecret이 풀려 Secret `choimory-dev-redis-secret`, `choimory-dev-member-api-secret`이 생겼는지 확인
- `get pods`: `choimory-dev-redis-0`이 `Running`이 되었는지 확인
- `get applications`: `*-data` Application이 `Healthy`가 되었는지 확인
- front, member-api는 이미지 태그가 임시값이라 계속 `ImagePullBackOff`. 7단계 전 준비 작업 후에 해결

## 임시 파일 정리

- Mac의 `~/Downloads/sealed`, 서버의 `~/sealed`는 레포에 복사를 마쳤으므로 삭제해도 됨
- `~/sealed-secrets-key-backup.yaml`은 지우기 전에 서버 밖 안전한 곳에 보관했는지 확인

---

# SealedSecret을 public 레포에 올려도 되는지

> secret.yaml도 git에 올려도 됨? public repo인데 문제되지 않음?

## 답변

- 올려도 됨. SealedSecret은 처음부터 public 레포에 커밋하라고 만든 방식
- 레포에 평문 Secret(`kind: Secret`, `stringData`)이나 키 백업 파일이 없는 것도 확인함

## 안전한 이유

- 암호화와 복호화의 키가 다름
  - 암호화는 컨트롤러의 공개키로 함
  - 풀 수 있는 개인키는 클러스터 안 `infrastructure` 네임스페이스의 Secret에만 있음
  - 레포의 암호문을 가져가도 클러스터 밖에서는 풀 수 없음
- 풀리는 위치가 고정됨
  - 기본 모드(strict)는 이름과 네임스페이스까지 묶어서 암호화
  - 다른 네임스페이스나 다른 이름으로 복사해 넣어도 풀리지 않음
- 레포에 보이는 것은 메타데이터뿐
  - Secret 이름, 네임스페이스, 키 이름(`password`, `jwt-secret`)
  - 이미 매니페스트에 공개되어 있어 새로 드러나는 것은 없음

## 주의할 점

1. 개인키 백업 파일이 핵심
   - `~/sealed-secrets-key-backup.yaml`이 유출되면 레포의 SealedSecret을 누구나 풀 수 있음
   - 절대 레포에 넣지 않고, 서버 밖에 보관할 때도 암호화된 곳(비밀번호 관리자 등)에 둠
2. git 기록에는 영구히 남음
   - 나중에 개인키가 유출되면 과거 커밋의 암호문까지 풀림. 파일만 지워서는 해결되지 않음
   - 그때는 컨트롤러 키 교체, 실제 값(Redis 비밀번호, JWT 키) 재발급, 새 SealedSecret 생성을 모두 해야 함
3. 공개키로는 누구나 암호화할 수 있음
   - 하지만 만든 SealedSecret을 클러스터에 넣으려면 레포 push 권한이나 클러스터 접근 권한이 필요하므로 문제되지 않음

- 정리: 레포의 SealedSecret 4개는 커밋해도 됨. 지켜야 할 것은 레포 밖에 있는 키 백업 파일 하나

---

# 디렉토리 이름 변경 검토

> project/choimory-dev의 디렉토리 명을 바꿨으면 좋게씅ㅁ 
>
>
> front -> choimory-dev-front
> member-api -> choimory-dev-member-api
> data -> database

## 영향 범위

| 대상 | 영향 |
|---|---|
| `projects/choimory-dev/{front,member-api,data}` | 디렉토리 이름 변경 |
| ApplicationSet 2개 | `component` 값이 디렉토리 경로와 Application 이름에 같이 쓰임 |
| 앱 레포 2개의 `.github/workflows/deploy.yml` | `projects/choimory-dev/front/overlays`, `.../member-api/overlays` 경로가 하드코딩됨. 바꾸지 않으면 Actions 실패 |
| 매니페스트 주석 | `data/base/redis.yaml` 같은 경로가 언급된 주석 |

## Application 이름 문제

- Application 이름이 `choimory-dev-{env}-{component}` 형식이라, 디렉토리 이름을 그대로 넣으면 `choimory-dev-stg-choimory-dev-front`처럼 중복됨
- 추천: 디렉토리와 Application 이름을 분리

```yaml
elements:
  - component: front
    dir: choimory-dev-front
  - component: member-api
    dir: choimory-dev-member-api
# protected
  - component: namespaces
    dir: namespaces
  - component: database
    dir: database
```

- 경로는 `projects/choimory-dev/{{ .dir }}/overlays/{{ .env }}`
- Application 이름은 `*-front`, `*-member-api`, `*-namespaces` 유지, `*-data`만 `*-database`로 변경
- `data` → `database`로 Application 이름이 바뀌면 ApplicationSet이 예전 Application을 지우고 새로 만듦
  - 템플릿에 `resources-finalizer`가 없고 protected는 `preserveResourcesOnDeletion: true`라, 예전 Application이 지워져도 DB, Redis, SealedSecret은 남음
  - 새 Application이 같은 이름의 리소스를 그대로 넘겨받음

## 앱 레포 workflow

- `choimory-dev-front`, `choimory-dev-member-api`의 `deploy.yml` 경로를 `choimory-dev-front/overlays`, `choimory-dev-member-api/overlays`로 변경 필요(주석 1줄, 명령 2줄)
- 앱 레포를 아직 `stg`에 머지하기 전이라 실행된 적은 없음
- gitops 변경이 `main`에 먼저 push되어 있어야 Actions가 정상 동작

---

# 디렉토리 이름 변경 보류와 다음 할 일

> 그럼 그건 걍 나중에하자 이제 뭐해야하지?

- 디렉토리 이름 변경은 보류. 다음 계획 문서(plan8)의 미뤄둔 작업으로 옮김

## 다음 할 일 요약

1. 커밋, push: SealedSecret 4개, kustomization 4개, task5. `main`까지 반영
2. 6단계 반영 확인(서버): [커밋 후 반영 확인 (서버)](#커밋-후-반영-확인-서버) 명령 실행. Secret 2개 생성, `choimory-dev-redis-0` `Running`, `*-data` `Healthy`면 6단계 완료
3. 7단계 전 준비(직접): Deploy Key 등록, front Environments 변수 등록, 앱 레포 `stg` 머지, GHCR 패키지 public 전환
4. 7단계 1차 확인: Pod 정상 여부, `port-forward`와 `curl` 헬스체크, actuator(8081), 로그인 API
- 중간에 편할 때: `argocd-initial-admin-secret` 삭제, `~/sealed`와 `~/Downloads/sealed` 삭제, 키 백업 파일 서버 밖 보관

---

# task5 정리

## 완료한 것

| 단계 | 내용 | 상태 |
|---|---|---|
| 5 | ArgoCD Helm 설치(`argocd`, 차트 10.17.0), root 등록, 공용 도구 wave 0~6 동기화 | 완료 |
| 5 | wave 7 `choimory-dev`: ApplicationSet 2개, AppProject, Application 8개, 네임스페이스 2개 | 완료 |
| - | 리소스 이름에 `choimory-dev-` 접두어 적용, 예전 리소스와 PV, 폴더 정리 | 완료 |
| 6 | kubeseal 0.41.2 설치, SealedSecret 4개 생성, 암호화 키 백업, 레포 반영 | push와 반영 확인 대기 |

## 레포 변경

- `infrastructure/gateway/gateway.yaml`: EnvoyProxy에 `sync-wave: "-1"` 추가
- `projects/choimory-dev/`: front, member-api, postgres, redis의 리소스 이름과 Secret 이름 변경
- `projects/choimory-dev/data/overlays/{stg,prod}/choimory-dev-redis-secret.yaml`, `member-api/overlays/{stg,prod}/choimory-dev-member-api-secret.yaml` 추가, 각 `kustomization.yaml`에 등록

## 겪은 문제와 해결

| 문제 | 원인 | 해결 |
|---|---|---|
| `envoy-gateway`가 한동안 `OutOfSync`, `Missing` | OCI 차트를 받아 오는 데 시간이 걸림 | 기다리니 정상 |
| `gateway`가 `Degraded` | GatewayClass가 EnvoyProxy보다 먼저 처리되어 `Accepted: False`가 남음 | 컨트롤러 재시작과 `refresh`. 재발 방지로 EnvoyProxy에 sync-wave 추가 |
| wave 7이 생성되지 않음 | `root` 동기화가 wave 6 `Degraded` 동안 재시도 5번을 모두 쓰고 `Failed`로 끝남 | `operation` 필드 patch로 수동 동기화 |
| 이름 변경 후 예전 postgres, redis가 남음 | `data`는 prune이 꺼진 보호 대상 | 직접 삭제. `Retain`으로 남은 PV와 폴더도 삭제 |

## 이어서 할 것

- [plan8.md](plan8.md) 참고
