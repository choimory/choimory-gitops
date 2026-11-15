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
- [cilium-secrets 네임스페이스](#cilium-secrets-네임스페이스)
  - [답변](#답변-2)
  - [용도](#용도)
  - [비어 있는 이유](#비어-있는-이유)
  - [처리](#처리)
- [Tailscale 내부망에서 ArgoCD 도메인 접속](#tailscale-내부망에서-argocd-도메인-접속)
  - [답변](#답변-3)
  - [선택지](#선택지)
  - [A안을 고르면 할 일](#a안을-고르면-할-일)
  - [확인할 것](#확인할-것-1)
- [인프라 도구 접속은 이후로 미룸](#인프라-도구-접속은-이후로-미룸)
  - [답변](#답변-4)
  - [진행 순서](#진행-순서)
  - [3번 때 미리 알아둘 점](#3번-때-미리-알아둘-점)
- [ArgoCD 초기 비밀번호 유지](#argocd-초기-비밀번호-유지)
  - [답변](#답변-5)
  - [지금 그대로 두어도 되는 이유](#지금-그대로-두어도-되는-이유)
  - [주의할 점](#주의할-점)
- [도메인 연결 후 초기 비밀번호 처리](#도메인-연결-후-초기-비밀번호-처리)
  - [답변](#답변-6)
  - [그때 지킬 순서](#그때-지킬-순서)
  - [참고](#참고)

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

---

# cilium-secrets 네임스페이스

> ```
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl get ns
> NAME                STATUS   AGE
> choimory-dev-prod   Active   5d3h
> choimory-dev-stg    Active   5d3h
> choimory-infra      Active   41s
> cilium-secrets      Active   6d
> default             Active   6d
> kube-node-lease     Active   6d
> kube-public         Active   6d
> kube-system         Active   6d
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl get all cilium-secrets
> error: you must specify only one resource
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl get all -n cilium-secrets 
> No resources found in cilium-secrets namespace.
> ```
>
> cilium secrets은 뭐임 ?

## 답변

- Cilium을 설치할 때 Cilium Helm 차트가 기본으로 만드는 네임스페이스
- 이번 네임스페이스 변경 작업과는 관계없고, 4단계에서 Cilium 설치 시 생김 (AGE 6d)
- 출력에서 `infrastructure`는 없고 `choimory-infra`만 있어 네임스페이스 전환은 정상

## 용도

- Cilium의 L7 기능(Cilium Ingress, Cilium Gateway API, L7 네트워크 정책의 TLS 처리)이 쓰는 TLS 인증서(Secret)를 모아두는 전용 보관 장소
- Cilium은 다른 네임스페이스의 Secret을 직접 읽지 않고 필요한 인증서를 이 네임스페이스로 복사해서 씀. Cilium에 클러스터 전체 Secret 읽기 권한을 주지 않으려는 설계

## 비어 있는 이유

- `bootstrap/cilium-values.yaml`에서 `gatewayAPI.enabled: false`, `ingressController.enabled: false`로 해당 기능을 모두 끔
- 외부 요청 처리와 TLS는 Envoy Gateway와 Cloudflare가 담당
- 그래서 네임스페이스만 있고 안은 비어 있음

## 처리

- 그대로 두는 것을 추천. 빈 네임스페이스라 자원을 쓰지 않음
- 지워도 Cilium Helm 차트가 관리하므로 Cilium 업그레이드 시 다시 생김
- 아예 만들지 않게 하려면 cilium-values에 설정을 추가하고 Cilium을 `helm upgrade`해야 함. CNI 재설정이라 얻는 것에 비해 위험이 커서 비추천

---

# Tailscale 내부망에서 ArgoCD 도메인 접속

> tailscale 내부망에서는 도메인으로 접속할수 있으면 좋을거 같은데

## 답변

- 가능함. k8s 안에서 Git으로 관리하려면 Tailscale Kubernetes Operator(A안) 추천
- 아직 파일은 수정하지 않음
- 현재 접속 방법(port-forward + SSH 터널)은 [task8.md ArgoCD 접속 방법](./task8.md#argocd-접속-방법) 참고

## 선택지

| 방법 | 접속 주소 | 동작 방식 | 장점 | 단점 |
|---|---|---|---|---|
| A. Tailscale Kubernetes Operator (추천) | `https://argocd.<TS_NET_NAME>.ts.net` | 클러스터에 operator를 설치하고, ArgoCD Ingress에 `tailscale` 클래스를 지정하면 tailnet에 `argocd` 기기가 생김 | tailnet 안에서만 접속. 정식 HTTPS 인증서 자동 발급. gitops로 관리. 이후 Grafana 같은 내부 도구도 같은 방식으로 추가 | 설치할 것이 늘어남 (operator Pod, 메모리 약 50~100Mi). Tailscale 관리 화면에서 OAuth 클라이언트 발급 필요 |
| B. 서버에서 `tailscale serve` | `https://choimory-server.<TS_NET_NAME>.ts.net` | 서버 호스트의 Tailscale이 HTTPS를 받아 ArgoCD로 넘김. ArgoCD는 NodePort로 열어야 함 | 설치할 것 없음. 인증서 자동 발급 | 서버에서 직접 설정해 gitops 밖에 있음. 서버 이름 하나를 ArgoCD가 차지. NodePort는 공유기 내부망에도 열림 |
| C. NodePort만 사용 | `https://choimory-server:30443` | ArgoCD Service를 NodePort로 변경 | 가장 간단. gitops로 관리 | 도메인이라기보다 서버 이름과 포트. 자체 서명 인증서 경고 계속. 공유기 내부망에도 열림 |

- `argocd.choimory.dev`처럼 내 도메인을 쓰는 방법도 있지만, Cloudflare DNS에 Tailscale IP를 넣어야 해 내부 IP가 공개 DNS에 노출되고 인증서도 따로 발급해야 하므로 비추천

## A안을 고르면 할 일

| 순서 | 작업 | 담당 |
|---|---|---|
| 1 | Tailscale 관리 화면에서 tag(`tag:k8s-operator`, `tag:k8s`)와 OAuth 클라이언트 발급 | 직접 |
| 2 | OAuth client ID, secret을 SealedSecret으로 생성 | 직접 실행, 명령어는 Claude |
| 3 | gitops에 operator Application 추가 (`choimory-infra`, Helm 차트) | Claude |
| 4 | ArgoCD values 수정: `server.insecure: true`, `tailscale` Ingress 추가 | Claude |
| 5 | 반영 후 Mac에서 `https://argocd.<TS_NET_NAME>.ts.net` 접속 확인 | 직접 |

- 4번에서 ArgoCD가 HTTP로 동작하도록 `server.insecure`를 켬. HTTPS는 Tailscale이 처리하며 클러스터 밖으로 노출되지 않음
- Tailscale 무료 플랜의 기기 수 제한 안에서 충분함. 서비스 하나당 기기 1개 증가
- 문서에는 tailnet 이름을 `<TS_NET_NAME>`으로 마스킹

## 확인할 것

1. 4차에서 설치한 홈서버 앱(Uptime Kuma, Homepage, Dockge 등)의 현재 접속 방식. 이미 쓰는 방식이 있으면 맞추는 것이 좋음
2. A안으로 진행할지
3. 지금 할지, 7단계(앱 배포)를 먼저 끝내고 할지. 7단계 먼저 추천. 그동안 ArgoCD는 port-forward로 접속

---

# 인프라 도구 접속은 이후로 미룸

> 인프라 도구들 (argocd, grafana, prometheus 등) 접속처리는 나중에하자 일단 앱 배포하고 cloudflare tunnel까지 해놓고 그 이후에 구매한 도메인이용해서 인프라도 도메인 활용해서 작업해주면 될듯

## 답변

- 결정: 인프라 도구(ArgoCD, Grafana, Prometheus 등) 접속 처리는 앱 배포와 Cloudflare Tunnel 작업 이후로 미룸
- 위 [Tailscale 내부망에서 ArgoCD 도메인 접속](#tailscale-내부망에서-argocd-도메인-접속)의 선택지(A·B·C)는 진행하지 않음
- 그때까지 ArgoCD는 port-forward와 SSH 터널로 접속 ([task8.md ArgoCD 접속 방법](./task8.md#argocd-접속-방법))

## 진행 순서

| 순서 | 작업 |
|---|---|
| 1 | 7단계: 앱 배포 (앱 레포 머지 → stg Run workflow → GHCR public → stg 확인 → prod) |
| 2 | 2차: Cloudflare Tunnel로 앱 도메인 연결 (`choimory.dev`, `staging.choimory.dev`, `member-api.choimory.dev`, `staging-member-api.choimory.dev`) |
| 3 | 인프라 도구(ArgoCD, Grafana, Prometheus 등)를 구매한 도메인으로 연결 |

## 3번 때 미리 알아둘 점

- 인프라 도구를 같은 Cloudflare Tunnel로 도메인에 연결하면 인터넷에 공개되므로, 앞단에 로그인 관문을 두는 것이 사실상 필수
- Cloudflare Access(Zero Trust, 50명까지 무료)로 `argocd.choimory.dev` 등에 접속하면 먼저 Cloudflare 로그인(이메일 인증 등)을 거치게 함
- 2번 Tunnel 구성 시 확장을 고려함. 예: `*.choimory.dev` 와일드카드로 연결하면 3번에서 도메인 추가 시 Tunnel 설정을 다시 건드릴 필요 없음
- 자세한 계획은 2번 이후 새 plan 문서에서 정리

---

# ArgoCD 초기 비밀번호 유지

> 그럼 argocd는 아직 초기 비밀번호 냅둔상태지

## 답변

- 맞음. 아직 로그인하지 않았으므로 `argocd-initial-admin-secret`이 그대로 있고 비밀번호도 초기값

## 지금 그대로 두어도 되는 이유

- ArgoCD는 외부에 공개되지 않음. 접속하려면 서버 SSH 접속과 `kubectl` 권한이 필요함
- 그 권한이 있으면 클러스터 전체를 다룰 수 있으므로, 초기 비밀번호 Secret이 남아 있어도 위험이 늘지 않음
- 인프라 도구 접속 작업(도메인 연결) 때 함께 처리해도 됨

## 주의할 점

- 비밀번호를 바꾸기 전에 Secret만 지우면 안 됨. Secret을 지워도 비밀번호는 초기값 그대로이고, 값을 다시 확인할 방법만 사라짐. 순서는 "로그인 → 비밀번호 변경 → Secret 삭제"
- 도메인으로 공개하기 전에는 반드시 변경. 인프라 도구 접속 작업의 첫 단계로 넣음
- 7단계를 ArgoCD 화면으로 보면서 진행하고 싶으면 지금 port-forward와 SSH 터널로 처리해도 됨 ([task8.md ArgoCD 접속 방법](./task8.md#argocd-접속-방법))

---

# 도메인 연결 후 초기 비밀번호 처리

> 초기비밀번호 secret 남겨뒀으니까 나중에 argocd 도메인 붙인다음에 그거 확인해서 계정 접속하고 초기 비밀번호 secret 지워도되지 지금 안하고

## 답변

- 그렇게 해도 됨. 초기 비밀번호 Secret은 지우지 않는 한 남아 있어 나중에 확인해서 로그인하면 됨
- 결정: 초기 비밀번호 처리는 인프라 도구 도메인 연결 작업 때 진행

## 그때 지킬 순서

1. Cloudflare Access(로그인 관문)를 먼저 걸고 ArgoCD 도메인 연결
2. 초기 비밀번호를 확인해 로그인

```bash
kubectl -n choimory-infra get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d; echo
```

3. User Info → Update Password로 비밀번호 변경
4. 초기 비밀번호 Secret 삭제

```bash
kubectl -n choimory-infra delete secret argocd-initial-admin-secret
```

- Cloudflare Access 없이 도메인부터 열면 초기 비밀번호 상태의 로그인 화면이 공개되므로 1번이 먼저
- 1번을 미루게 되면, 도메인을 열기 전에 port-forward로 3번까지 먼저 끝냄

## 참고

- ArgoCD를 업그레이드하거나 설정을 바꿔도 이 Secret은 다시 만들어지거나 바뀌지 않음. 지우기 전까지 유지
- ArgoCD를 다시 설치하면 새 비밀번호로 바뀌므로 그때 새로 확인
