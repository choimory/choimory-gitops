# 목차

- [개요](#개요)
- [고려사항 1차 분석](#고려사항-1차-분석)
  - [답변](#답변)
  - [클러스터: k3s vs k8s(kubeadm)](#클러스터-k3s-vs-k8skubeadm)
  - [CI 선택](#ci-선택)
  - [CD 선택](#cd-선택)
  - [CR 선택](#cr-선택)
  - [GitOps 레이어 구조](#gitops-레이어-구조)
  - [대상 레포 분석 중 추가 고려사항](#대상-레포-분석-중-추가-고려사항)
  - [결정 필요 사항](#결정-필요-사항)
- [추가1](#추가1)
- [추가1 분석](#추가1-분석)
  - [답변](#답변-1)
  - [Kubernetes 선택: k3s에서 못 하는 것](#kubernetes-선택-k3s에서-못-하는-것)
  - [CI 선택: 인바운드 문제](#ci-선택-인바운드-문제)
  - [CD 스택 비교](#cd-스택-비교)
  - [확인 필요 사항](#확인-필요-사항)
- [추가2](#추가2)
- [추가2 분석](#추가2-분석)
  - [답변](#답변-2)
  - [Kubernetes: kubeadm 선택 시 고려사항](#kubernetes-kubeadm-선택-시-고려사항)
  - [CI/CD 확정](#cicd-확정)
  - [CR: GHCR 인바운드 여부](#cr-ghcr-인바운드-여부)
  - [이름 정리](#이름-정리-1)
  - [남은 결정 사항](#남은-결정-사항)
- [최종 스펙](#최종-스펙)
  - [답변](#답변-3)
  - [확정 스펙](#확정-스펙)
  - [미정 항목](#미정-항목)

---

# 개요

- https://app.notion.com/p/PC-3fe87e6bf6528071ad2af10621fab4dc?source=copy_link
- 홈서버에 개발 환경 구축 작업 진행 (노션문서의 5차)
- 해야할일
  - 홈서버에 k8s 클러스터 설치
  - CI/CD 구축
  - /Users/choimory/choimory_workspace/IdeaProjects/choimory-dev/choimory-dev-front, choimory-dev-member-api를 1차 대상으로 진행
- 고려사항
  - 홈서버에 k3s 설치할지, k8s 설치할지
  - CI: Jenkins, Github Actions, Argo workflows, CircleCI 중에 무엇을 선택할지
  - CD: ArgoCD, FluxCD, GoCD 중에 무엇을 선택할지
  - CR: GHCR, Dockerhub 중에 무엇을 선택할지
  - GitOps: GitOps 사용할 예정이며 Kubernetes, Kustomize > Helm (CD Bootstrap) > CD stack 으로 관리할 예정
---

# 고려사항 1차 분석

> plan1 확인

## 답변

- 참고 자료: plan1 개요, 노션 5차 작업 진행 문서, 노션 4차 작업 진행 문서(⑤ GitOps 계획), choimory-dev-front, choimory-dev-member-api 레포

| 항목 | 추천 | 이유 |
| --- | --- | --- |
| 클러스터 | k3s | 미니PC 노드 1대에 Docker 앱들과 같이 쓰기 좋고, 노션 계획(설치 명령, 저장 경로)이 이미 k3s 기준 |
| CI | GitHub Actions | 코드가 GitHub에 있고 홈서버 자원을 쓰지 않음. `GITHUB_TOKEN`만으로 GHCR에 푸시 가능 |
| CD | ArgoCD | 클러스터가 Git을 가져가는 pull 방식이라 6443 포트를 외부에 열 필요가 없음. 노션 4차의 App of Apps 계획과 같음 |
| CR | GHCR | Actions와 인증이 바로 연결됨. Docker Hub 무료 플랜은 pull 횟수 제한과 private 저장소 1개 제약이 있음 |

## 클러스터: k3s vs k8s(kubeadm)

- 쿠버네티스 API는 똑같아서 매니페스트, Kustomize, Helm, ArgoCD를 다루는 경험에는 차이가 없음
- kubeadm은 CNI, Ingress, 스토리지, 인증서 갱신을 전부 직접 구성해야 함. 노드 1대 홈서버에서는 이 부담이 이득보다 큼
- k3s는 Traefik, local-path 스토리지, CoreDNS가 기본으로 들어 있고, 노션에 정리한 `--default-local-storage-path /data/dev/choimory-dev/k8s`를 그대로 쓸 수 있음
- 클러스터 내부 구조 자체를 공부하는 게 목적이 아니라면 k3s 추천

## CI 선택

- Jenkins: 미니PC에 상주해야 하고 JVM이라 메모리를 많이 씀. 이미 Docker 앱이 여러 개 돌고 있어 부담
- Argo Workflows: ArgoCD와 별개 프로젝트. 빌드가 클러스터 안에서 돌아 자원을 쓰고, GitHub webhook을 받으려면 클러스터 입구를 열어야 함
- CircleCI: 외부 서비스가 하나 더 늘 뿐 Actions보다 나은 점이 없음
- GitHub Actions: 기본 러너가 amd64라 서버 CPU(7735HS)와 아키텍처가 같음 → 추천

## CD 선택

- 노션 5차에서는 "A안(Actions + Tailscale)으로 먼저 → 나중에 B안(ArgoCD)"을 추천했었음
- 지금은 gitops 레포를 따로 만들었고 GitOps로 가겠다고 정했으므로, 처음부터 ArgoCD로 가는 게 두 번 일하지 않는 길
- FluxCD도 괜찮지만 UI가 없어 처음 배울 때는 ArgoCD가 상태를 확인하기 편함
- GoCD는 쿠버네티스 GitOps 전용 도구가 아니라 제외

### 배포 흐름

1. 앱 레포 push
2. Actions가 테스트 → 이미지 빌드 → `ghcr.io/choimory/<앱>:<커밋SHA>` 푸시
3. Actions가 gitops 레포의 이미지 태그를 수정해서 커밋
4. ArgoCD가 변경을 감지해 배포

### 이미지 태그 갱신 방식

- Actions가 다른 레포(gitops)에 커밋하는 구조라 gitops 레포용 Deploy Key 또는 Fine-grained PAT가 필요
- 대안으로 ArgoCD Image Updater가 있지만, 처음에는 Actions 커밋 방식이 흐름이 잘 보여서 더 나음

## CR 선택

- GHCR: Actions의 `GITHUB_TOKEN`(`permissions: packages: write`)으로 별도 토큰 없이 푸시 가능
- 패키지가 private이면 k3s에서 pull하기 위해 네임스페이스마다 `imagePullSecret` 필요 (`read:packages` 권한만 있는 토큰으로 생성)
- Docker Hub: 무료 플랜은 pull 횟수 제한과 private 저장소 1개 제약이 있음

## GitOps 레이어 구조

- 개요의 "Kubernetes, Kustomize > Helm (CD Bootstrap) > CD stack"을 아래 구조로 해석함 (확인 필요)

```txt
bootstrap/          # Helm으로 ArgoCD 최초 설치 (values, 차트 버전 고정) + root-app.yaml (한 번만 수동으로 apply)
platform/           # 클러스터 공용 (ArgoCD 자신, sealed-secrets, cloudflared, cloudnative-pg, redis)
apps/
  member-api/
    base/           # Kustomize base (Deployment, Service, Ingress)
    overlays/dev/   # 환경별 이미지 태그, 환경변수, replicas
  front/
    base/
    overlays/dev/
```

## 대상 레포 분석 중 추가 고려사항

### front의 NEXT_PUBLIC 빌드 인자

- `NEXT_PUBLIC_MEMBER_API_BASE_URL`은 빌드할 때 번들에 고정됨
- dev와 prod가 같은 이미지를 쓸 수 없어 환경별로 이미지를 따로 빌드(태그 분리)하거나, 런타임에 주입하는 방식으로 바꿔야 함

### member-api 의존성

- PostgreSQL, Redis, JWT secret이 필요
- DB: CloudNativePG로 클러스터 안에 둘지 결정 필요
- Redis: StatefulSet으로 띄우는 방안
- 비밀값 관리: Sealed Secrets(노션 계획)와 External Secrets 중 선택 필요

### 도메인과 환경 범위

- member-api CORS 설정에 `dev.choimory.dev`(dev), `choimory.dev`(prod)가 있음
- 이번 작업이 dev 환경만인지 prod까지인지 범위를 정해야 함

### 이름 정리

- 로컬 디렉토리는 `choimory-dev-*`, 원격 레포는 `choimory-io-*`, 이 레포 README 제목은 `choimory-dev-manifests`
- GHCR 이미지 이름을 정할 때 함께 맞추면 좋음

## 결정 필요 사항

- [ ] 앱 레포 두 개의 public/private 여부 (GHCR pull secret 필요 여부)
- [ ] 이번 범위: dev 환경만 / prod까지
- [ ] DB와 Redis를 k3s 안에 둘지
- [ ] 추천 방향(k3s, GitHub Actions, ArgoCD, GHCR)으로 진행할지

---

# 추가1

- Kubernetes 선택하기
  - k3s에서 사용하지 못하는것들이 무엇인지를 확인해야함
  - 이중에 내가 하고 싶은것들을 k3s에서 못한다면 k8s도 고려할거임 (이를테면 HPA)
  - 반대로 k8s가 서버스펙에 부담스러우면 포기할수도 있음
  - k8s 경험 목적이 크므로 프로젝트만을 고려한 선택을 하지는 않음
- CI 선택
  - 홈서버는 현재 상태로는 아웃바운드는 가능하지만 인바운드는 불가능할것으로 예상됨
  - 이것을 수정하던지, 아니면 온프레미스로 가야함
  - 기술적으로 큰 무리가 없으면 서버에 저장용량, 램에 부담을 덜주는 Github Actions가 좋아보이긴 함
- CD
  - 주요 CD 스택들의 각각의 장단점과 내 현재 상황에 적용했을때 어떨지 확인 필요

---

# 추가1 분석

> 추가 1 확인

## 답변

- 추가1의 Kubernetes 선택, CI 선택, CD 항목을 분석함
- 서버 스펙(사용자 확인): Beelink EQR6, AMD Ryzen 7 7735HS(노션 기록 기준, Radeon 내장 그래픽) 8코어 16스레드, RAM 32GB

## Kubernetes 선택: k3s에서 못 하는 것

- k3s는 CNCF 공식 인증을 받은 쿠버네티스 배포판이라 API와 리소스가 표준과 같음
- HPA는 k3s에서 가능. HPA에 필요한 metrics-server가 k3s에는 기본으로 포함되어 있음

### k3s에서도 그대로 되는 것

| 기능 | 비고 |
| --- | --- |
| HPA, VPA, PDB | metrics-server 기본 포함 |
| RBAC, ServiceAccount, NetworkPolicy | NetworkPolicy 컨트롤러 내장 |
| Ingress, Gateway API | Traefik 기본. 빼고 ingress-nginx로 교체 가능 |
| CRD, Operator | ArgoCD, CloudNativePG 등 전부 사용 가능 |
| Helm, Kustomize | 클라이언트 쪽 도구라 차이 없음 |
| 멀티 노드 | 나중에 미니PC를 늘리면 agent로 붙이면 됨 |

### k3s에서 다르거나 못 하는 것

- 컨트롤 플레인 운영 경험: k3s는 apiserver, scheduler, controller-manager, etcd를 프로세스 하나로 묶어 실행함. 그래서 아래 작업들을 표준 방식으로 연습할 수 없음
  - `kube-system`에서 static pod로 보고 고치기
  - `kubeadm upgrade`
  - 인증서 갱신
  - etcdctl 백업과 복구
- 데이터 저장소: 기본이 SQLite. `--cluster-init` 옵션으로 내장 etcd로 바꿀 수 있음
- 기본 구성요소: CNI(flannel), 로드밸런서(ServiceLB), 스토리지(local-path)가 미리 정해져 있음. 옵션으로 빼고 교체 가능
- 클라우드 연동: 클라우드 프로바이더와 in-tree 스토리지 드라이버가 빠져 있음. 홈서버에서는 상관없음
- 워크로드와 플랫폼 운영(HPA, Ingress, GitOps, RBAC, 모니터링 등)은 k3s로 전부 경험 가능. 못 하는 건 클러스터 자체를 설치하고 관리하는 경험(CKA 시험 범위)뿐

### 서버 스펙 부담

| | k3s | kubeadm |
| --- | --- | --- |
| 컨트롤 플레인 유휴 메모리 | 약 0.5~1GB | 약 1.5~2GB (etcd, apiserver 등 개별 실행) |
| 사전 작업 | 거의 없음 | swap 끄기, containerd CRI 설정, CNI 직접 설치 |
| Docker와 공존 | 자체 containerd를 써서 깔끔함 | Docker가 깔아둔 containerd는 CRI 기능이 꺼져 있어 설정을 바꿔야 함. Docker 앱에 영향을 줄 수 있어 주의 필요 |

- RAM 32GB 기준: 기존 Docker 앱(Immich 머신러닝, Jellyfin 등)을 넉넉히 잡아도 kubeadm 컨트롤 플레인 2GB는 충분히 수용 가능
- CPU(8코어 16스레드)도 둘 다 여유 있음
- 서버 스펙은 k3s/kubeadm 선택의 제약이 아님. 남은 판단 기준은 경험하고 싶은 범위와 Docker 공존 작업 부담

### 중간 선택지: 조립형 k3s

- k3s를 쓰되 기본 구성요소를 빼고 직접 조립하는 방법
- `--disable traefik,servicelb,local-storage --flannel-backend=none --cluster-init`로 설치
- Cilium, ingress-nginx, MetalLB, 스토리지를 직접 설치
- 가볍게 유지하면서 CNI, Ingress, 로드밸런서, etcd를 직접 구성하는 경험을 얻을 수 있음

### 정리

- 컨트롤 플레인 운영(CKA 수준)까지 경험하고 싶으면 → kubeadm (스펙은 충분, Docker containerd 공존 설정은 주의)
- 그게 아니라면 → k3s (필요하면 조립형)

## CI 선택: 인바운드 문제

- GitHub Actions + ArgoCD 조합은 홈서버로 들어오는 인바운드가 필요 없음

```txt
[GitHub Actions 러너] ── 빌드·푸시 ──▶ [GHCR]           (전부 GitHub 쪽에서 일어남)
[Actions] ── 이미지 태그 커밋 ──▶ [gitops 레포]

[홈서버 ArgoCD] ── 3분마다 조회 ──▶ [gitops 레포]      (아웃바운드)
[홈서버 k3s]    ── 이미지 pull  ──▶ [GHCR]             (아웃바운드)
[cloudflared]   ── 터널 연결   ──▶ [Cloudflare]        (아웃바운드, 사용자 트래픽이 이 터널로 들어옴)
```

- 홈서버는 밖으로 나가는 연결만 하므로 현재 상태 그대로 가능
- 오히려 서버에 직접 설치하는 CI(Jenkins, Argo Workflows)가 인바운드를 더 필요로 함. GitHub push webhook을 받으려면 Cloudflare Tunnel 등으로 입구를 열거나, 주기적으로 레포를 조회하는 방식으로 우회해야 함
- ArgoCD에 webhook을 붙여 즉시 배포하고 싶을 때만 인바운드가 필요. 선택 사항이며 기본 3분 주기 조회로 충분
- 비용: public 레포는 Actions 무제한 무료, private 레포는 월 2,000분 무료. Gradle 빌드와 Next 빌드가 회당 5~10분 정도라 넉넉함
- 서버 저장공간과 RAM을 아끼는 GitHub Actions에 기술적으로 걸리는 문제는 없음

## CD 스택 비교

| | ArgoCD | FluxCD | GoCD |
| --- | --- | --- | --- |
| 성격 | 쿠버네티스 GitOps 전용 (CNCF 졸업) | 쿠버네티스 GitOps 전용 (CNCF 졸업) | 범용 CI/CD 파이프라인 서버 |
| 동작 방식 | Pull (클러스터가 Git을 감시하고 맞춤) | Pull | Push (에이전트가 배포 명령 실행) |
| 실제 상태가 Git과 달라지면 | 감지 후 자동 복구 | 감지 후 자동 복구 | 없음 |
| 메모리 | 약 0.5~1GB (안 쓰는 dex, notifications를 끄면 줄어듦) | 약 0.2~0.4GB | Java 서버 + 에이전트로 무거움 |
| UI | 강력함 (리소스 트리, diff, 동기화, 롤백) | 공식 UI 없음. Weave GitOps는 Weaveworks 폐업 후 정체되어 CLI 위주 | 파이프라인 UI 있음 |
| Helm 처리 | 템플릿으로 렌더링 (`helm list`에 안 보임) | 실제 Helm release로 설치 | 스크립트로 직접 실행 |
| 이미지 태그 자동 갱신 | Image Updater (별도 설치, 비교적 덜 성숙) | Image Automation 내장, 성숙함 | 파이프라인으로 직접 구현 |
| 부트스트랩 | Helm 설치 → root app (개요의 "Helm (CD Bootstrap)"과 그대로 맞음) | `flux bootstrap github` CLI가 레포에 직접 커밋하는 방식 | 서버 설치 |
| 국내 채용 공고 노출 | 가장 많음 | 적음 | 거의 없음 |

### 현재 상황에 대입

- ArgoCD: 인바운드 없이 동작하고, 개요의 Helm 부트스트랩 구조와 노션 4차의 App of Apps 계획에 그대로 맞음. UI로 Sync, Diff, Health 상태를 보며 배울 수 있어 k8s 경험 목적에 가장 잘 맞음. 단점은 Flux보다 메모리를 더 쓴다는 것뿐이며, RAM 32GB라 문제 되지 않음
- FluxCD: 가장 가볍고 이미지 자동 갱신이 내장됨. UI가 없어 처음 배우는 단계에서는 상태 파악이 어렵고, 부트스트랩 방식이 개요의 구조와 조금 다름
- GoCD: Push 방식이라 서버 접근 자격증명이 필요하고, 실제 상태가 Git과 달라져도 감지하지 못함. GitOps 목적에 맞지 않고 무거워서 제외
- 추천: ArgoCD

## 확인 필요 사항

- [x] 서버 RAM 용량: 32GB (kubeadm도 수용 가능)
- [ ] k8s 경험 범위: 컨트롤 플레인 운영까지(kubeadm) / 워크로드와 플랫폼 운영(k3s, 필요하면 조립형)

---

# 추가2

- Kubernetes: 서버 스펙에 큰 문제 없나? 그럼 그냥 실무에서 쓰이는 k8s로 가고 싶은 마음이 좀 더 크다
- CI/CD: Github Actions, ArgoCD로 가자
- CR: GHCR도 인바운드 필요없나?
- 그 외: 디렉토리명이 현재 가장 정확한 이름임. git 정보는 내가 한번 git repo명을 변경했는데 아직 예전 정보로 되어있는것. 아마 클론 새로 하면 바뀔텐데 내가 안해놨음

---

# 추가2 분석

> 추가2 확인

## 답변

- 추가2의 Kubernetes, CI/CD, CR, 그 외(이름) 항목을 분석함

## Kubernetes: kubeadm 선택 시 고려사항

- 서버 스펙(RAM 32GB, 8코어 16스레드)은 문제없음. kubeadm 컨트롤 플레인(약 2GB)과 기존 Docker 앱을 같이 돌려도 여유 있음
- 실무 쿠버네티스는 대부분 EKS, GKE 같은 관리형이라 컨트롤 플레인이 숨겨져 있음. kubeadm은 그보다 한 단계 아래인 온프레미스 실무와 가깝고 CKA 시험 범위와도 같아 경험 목적에 좋은 선택
- kubeadm으로 가면 k3s가 대신 해주던 것들을 직접 챙겨야 함

| 항목 | 내용 |
| --- | --- |
| Docker와 공존 | Docker가 설치한 containerd에는 CRI 기능이 꺼져 있음(`disabled_plugins = ["cri"]`). 이걸 켜고 `SystemdCgroup = true`로 설정해 같이 쓰는 방식 추천. Docker는 `moby`, k8s는 `k8s.io` 네임스페이스를 써서 컨테이너가 섞이지 않음. containerd 재시작 시 Docker 앱이 잠깐 영향받을 수 있어 작업 시간을 정해 진행. (cri-dockerd는 비추천) |
| swap | Ubuntu 서버 기본 `/swap.img`를 꺼야 함 |
| 커널 설정 | `overlay`, `br_netfilter` 모듈과 `ip_forward` sysctl. IP 포워딩은 노션 기준 이미 켜져 있음 |
| 대역 충돌 확인 | Pod `10.244.0.0/16`, Service `10.96.0.0/12`(기본값)는 Docker(`172.x`), LAN(`192.168.10.0/24`), Tailscale(`100.64.0.0/10`)과 겹치지 않아 그대로 사용 가능 |
| 노드 1대 구성 | 컨트롤 플레인 taint를 제거해 앱 파드도 같은 노드에 뜨게 함 |
| CNI | Calico 또는 Cilium. Cilium이 요즘 실무 채택이 늘고 있고 NetworkPolicy와 관측 기능도 강함 |
| Ingress | ingress-nginx는 2026년 3월에 유지보수 종료. 새로 시작한다면 Gateway API 구현체(Envoy Gateway, Cilium Gateway, Traefik) 추천 |
| 외부 노출 | Cloudflare Tunnel이 클러스터 안의 Gateway Service로 바로 연결하므로 MetalLB 같은 LoadBalancer는 지금은 불필요 |
| 스토리지 | local-path-provisioner를 직접 설치하고 경로를 `/data/dev/choimory-dev/k8s`로 지정 |
| HPA | metrics-server 직접 설치. kubelet 인증서 처리 방법(`serverTLSBootstrap` 또는 `--kubelet-insecure-tls`) 결정 필요 |
| 맥북에서 kubectl | `kubeadm init` 설정의 apiserver `certSANs`에 `choimory-server`, ts.net 주소, Tailscale IP 포함 |
| 운영 | 버전 고정. 업그레이드는 마이너 버전 하나씩만 가능. 인증서는 1년마다 만료되며 업그레이드 시 자동 갱신. etcd 스냅샷 백업 필요 |

- k3s 대비 설치 단계는 늘지만, 이 과정 자체가 원하는 "실무 k8s 경험"에 해당함

## CI/CD 확정

- CI: GitHub Actions
- CD: ArgoCD

## CR: GHCR 인바운드 여부

- GHCR도 인바운드 불필요
- 이미지 push: Actions 러너(GitHub 서버)가 `ghcr.io`로 올림. 홈서버는 관여하지 않음
- 이미지 pull: 홈서버의 containerd가 `ghcr.io`, `pkg-containers.githubusercontent.com`에 HTTPS(443)로 접속해서 받아옴. 아웃바운드라 현재 상태 그대로 가능
- private 패키지: 네임스페이스마다 `imagePullSecret`만 있으면 됨
- Docker Hub도 같은 구조. 인바운드가 필요해지는 경우는 Harbor처럼 레지스트리를 홈서버에 직접 설치할 때뿐 (Actions가 홈서버로 push해야 하므로)
- 결국 홈서버로 들어오는 경로는 Cloudflare Tunnel(사용자 트래픽)과 Tailscale(관리 접속) 두 개뿐. 둘 다 서버가 밖으로 먼저 연결하는 방식이라 공유기 포트포워딩 0개 유지 가능

## 이름 정리

- 디렉토리명(`choimory-dev-front`, `choimory-dev-member-api`)이 정확한 이름
- 이미지 이름: `ghcr.io/choimory/choimory-dev-front`, `ghcr.io/choimory/choimory-dev-member-api`
- git remote는 레포명 변경 전 정보로 남아 있음. 새로 클론하지 않아도 아래 명령으로 변경 가능 (GitHub가 예전 이름도 새 이름으로 연결해 줘서 당장 문제는 없음)

```bash
git remote set-url origin git@github.com:choimory/choimory-dev-front.git
git remote set-url origin git@github.com:choimory/choimory-dev-member-api.git
```

- 이 gitops 레포는 원격이 `choimory-gitops`인데 README 제목은 `choimory-dev-manifests`. 추후 README 정리 시 맞추기

## 남은 결정 사항

- [x] Kubernetes: kubeadm
- [x] CI/CD: GitHub Actions, ArgoCD
- [x] CR: GHCR
- [ ] 앱 레포의 public/private 여부 (pull secret 필요 여부)
- [ ] 범위: dev 환경만 / prod까지
- [ ] DB와 Redis를 클러스터 안에 둘지
- [ ] CNI(Calico 또는 Cilium)와 Gateway 구현체 선택. 따로 정하지 않으면 Cilium + Cilium Gateway 조합 추천 (구성요소 하나로 CNI와 Gateway를 같이 해결)

---

# 최종 스펙

> 일단 스펙은 정해졌지?

## 답변

- plan1은 스펙 정하기 문서로 마무리하고, 개발 방향은 plan2에서 진행
- 핵심 기술 스택은 확정, 클러스터 세부 구성과 운영 범위는 미정

## 확정 스펙

| 항목 | 결정 |
| --- | --- |
| 서버 | 홈서버 미니PC 노드 1대 (Beelink EQR6, Ryzen 7 7735HS 8코어 16스레드, RAM 32GB) |
| Kubernetes | kubeadm |
| CI | GitHub Actions |
| CD | ArgoCD (pull 방식, 인바운드 필요 없음) |
| CR | GHCR (`ghcr.io/choimory/choimory-dev-front`, `ghcr.io/choimory/choimory-dev-member-api`) |
| GitOps 구조 | 앱 매니페스트는 Kustomize, ArgoCD 최초 설치는 Helm, 그 뒤로는 ArgoCD가 클러스터 구성 전체 관리 |
| 외부 노출 | Cloudflare Tunnel (사용자 트래픽), Tailscale (관리 접속), 포트포워딩 0개 |
| 1차 배포 대상 | choimory-dev-front, choimory-dev-member-api |

## 미정 항목

| 항목 | 상태 |
| --- | --- |
| CNI, Gateway | 미정 (추천: Cilium + Cilium Gateway) |
| DB, Redis 위치 | 미정 (추천: 클러스터 안. DB는 CloudNativePG, Redis는 StatefulSet) |
| 비밀값 관리 | 미정 (노션 계획대로면 Sealed Secrets) |
| 환경 범위 | 미정 (dev만 / prod까지) |
| GHCR 패키지 공개 여부 | 미정 (앱 레포의 public/private에 따라 결정) |
