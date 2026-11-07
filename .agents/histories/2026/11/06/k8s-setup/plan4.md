# 목차

- [개요](#개요)
- [자원 확정과 구현 순서](#자원-확정과-구현-순서)
  - [답변](#답변)
  - [구현 순서](#구현-순서)
    - [0. 사전 준비](#0-사전-준비)
    - [1. 노드 준비 (서버에서 직접)](#1-노드-준비-서버에서-직접)
    - [2. 클러스터 생성](#2-클러스터-생성)
    - [3. CNI (Cilium)](#3-cni-cilium)
    - [4. gitops 기본 구조와 ArgoCD](#4-gitops-기본-구조와-argocd)
    - [5. 공용 도구 (ArgoCD가 설치)](#5-공용-도구-argocd가-설치)
    - [6. 프로젝트 기반 (choimory-dev)](#6-프로젝트-기반-choimory-dev)
    - [7. 데이터 (stg 먼저, 그다음 prod)](#7-데이터-stg-먼저-그다음-prod)
    - [8. 앱 레포 작업](#8-앱-레포-작업)
    - [9. 배포와 연결 (stg 먼저)](#9-배포와-연결-stg-먼저)
    - [10. 마무리](#10-마무리)
  - [진행 방식](#진행-방식)
- [추가1](#추가1)
- [추가1 분석](#추가1-분석)
  - [답변](#답변-1)
  - [clone 후 kubectl apply만 하는 방식](#clone-후-kubectl-apply만-하는-방식)
  - [gitops를 먼저 작성할 때 막히는 것: SealedSecret](#gitops를-먼저-작성할-때-막히는-것-sealedsecret)
  - [순서에서 빠진 것: Actions 워크플로](#순서에서-빠진-것-actions-워크플로)
  - [Cloudflare Tunnel과 도메인을 2차로 미루는 것](#cloudflare-tunnel과-도메인을-2차로-미루는-것)
  - [조정한 순서](#조정한-순서)
- [구현 순서 확정](#구현-순서-확정)
  - [답변](#답변-2)
  - [확정 순서](#확정-순서)
  - [구현 시작 시 진행 방식](#구현-시작-시-진행-방식)

---

# 개요

- plan1(스펙), plan2(개발 방향), plan3(네임스페이스, 디렉토리, 데이터 경로, 자원)에서 결정한 내용을 바탕으로 구현 순서를 정리함
- 결정 사항 전체 요약은 [plan3의 정해진 것과 정해야 할 것 요약](./plan3.md#정해진-것과-정해야-할-것-요약)과 [plan3의 최종결정 분석](./plan3.md#최종결정-분석) 참고
- 참고 문서: [plan1.md](./plan1.md), [plan2.md](./plan2.md), [plan3.md](./plan3.md)

---

# 자원 확정과 구현 순서

> 자원은 그 값으로 확정, 구현순서 정리해줘

## 답변

- 자원 값은 plan3 최종결정 분석의 값으로 확정
- 구현 순서는 앞 단계가 끝나야 다음 단계가 가능한 순서로 정리

## 구현 순서

### 0. 사전 준비

- Docker Compose 앱 데이터의 백업 상태 확인 (1단계에서 containerd를 재시작하기 때문)
- 서버에 데이터 폴더 생성
  - `/data/dev/choimory/k8s`
  - `/data/dev/choimory-dev/k8s`
- Cloudflare Tunnel 토큰 준비
- 앱 레포 git remote 정리 (`choimory-io-*` → `choimory-dev-*`)

### 1. 노드 준비 (서버에서 직접)

- swap 끄기 (`/swap.img`)
- 커널 모듈(`overlay`, `br_netfilter`) 활성화, `ip_forward` sysctl 설정
- containerd에서 CRI를 켜고 `SystemdCgroup = true`로 변경
  - containerd를 재시작하면 Docker Compose 앱도 잠깐 재시작됨. 사용이 적은 시간에 진행
  - 재시작 후 Jellyfin, Immich 등이 정상인지 확인
- kubeadm, kubelet, kubectl 설치 후 자동 업그레이드되지 않게 버전 고정

### 2. 클러스터 생성

- `kubeadm init`을 설정 파일로 실행
  - Pod CIDR `10.244.0.0/16`, Service CIDR `10.96.0.0/12`
  - `certSANs`에 Tailscale 이름과 IP
  - kubelet 예약값(`systemReserved`, `kubeReserved`, `evictionHard`)
- control-plane taint 제거 (노드가 1대라 여기에 Pod를 띄워야 함)
- Mac에서 Tailscale로 `kubectl` 접속 확인

### 3. CNI (Cilium)

- Helm으로 직접 설치. Gateway와 Ingress 기능, Hubble은 끔
- 노드가 `Ready`가 되는지 확인
- CNI가 없으면 ArgoCD도 뜨지 않으므로 이 단계는 수동으로 진행
- 설치 값은 `bootstrap/`에 기록해 두고, 나중에 ArgoCD 관리로 넘길지는 그때 판단

### 4. gitops 기본 구조와 ArgoCD

- gitops 레포에 `bootstrap/`, `infrastructure/`, `projects/` 뼈대 생성
- ArgoCD를 Helm으로 `infrastructure` 네임스페이스에 설치 (Dex, notifications 끄고 레플리카 1개)
- root Application 등록. 이후로는 git push만 하면 ArgoCD가 반영

### 5. 공용 도구 (ArgoCD가 설치)

- 설치 순서가 중요하므로 ArgoCD sync wave로 순서를 정함

| 순서 | 도구 | 확인 |
| --- | --- | --- |
| 1 | Sealed Secrets | `kubeseal`로 테스트 Secret을 암호화하고 풀리는지 확인 |
| 2 | local-path-provisioner, StorageClass | `storageClassConfigs` 지원 여부 확인. 테스트 PVC가 의도한 폴더에 만들어지는지 확인 |
| 3 | metrics-server | `kubectl top node`가 되는지 |
| 4 | CloudNativePG 오퍼레이터 | 오퍼레이터 Pod가 Running인지 |
| 5 | Envoy Gateway, 공용 Gateway | 프록시 Service가 ClusterIP인지, ArgoCD에서 Healthy인지 |
| 6 | cloudflared | Tunnel 토큰은 SealedSecret으로. 테스트 앱을 임시 도메인으로 열어 끝까지 연결되는지 확인 |

### 6. 프로젝트 기반 (choimory-dev)

- 네임스페이스(`choimory-dev-stg`, `choimory-dev-prod`) 생성
- ResourceQuota와 LimitRange 적용
- NetworkPolicy로 stg와 prod 사이 통신 차단
- AppProject와 ApplicationSet 생성

### 7. 데이터 (stg 먼저, 그다음 prod)

- CloudNativePG Cluster 생성. DB 접속 Secret이 자동으로 생기는지 확인
- Redis StatefulSet 생성. prod만 AOF 켬
- Redis 비밀번호와 JWT secret을 SealedSecret으로 생성
- 폴더가 `/data/dev/choimory-dev/k8s/<네임스페이스>/`에 생기는지 확인

### 8. 앱 레포 작업

- member-api
  - `application-stg.yml` 추가
  - `application.yml` group에 `stg: stg, flyway` 추가
  - JVM 옵션 확인
- GitHub Actions (member-api, front)
  - GitHub Environments(stg, prod)에 환경별 변수를 둠
  - 이미지를 빌드해 GHCR에 `stg-<sha>`, `prod-<sha>` 태그로 push
  - gitops overlay의 이미지 태그를 고쳐서 커밋. 이를 위해 gitops 레포에 push할 수 있는 토큰이나 deploy key 필요
- 첫 push 후 GHCR 패키지가 public인지 확인

### 9. 배포와 연결 (stg 먼저)

- member-api와 front overlay, HTTPRoute 작성
- Cloudflare에 `staging.choimory.dev`, `staging-api.choimory.dev` 연결
- stg에서 로그인, 토큰 재발급, 로그아웃 흐름과 CORS 확인
- `stg → main` PR을 머지해 prod 자동 배포 확인 후 `choimory.dev`, `api.choimory.dev` 연결

### 10. 마무리

- 실제 사용 메모리가 예상(평소 약 4Gi)과 맞는지 확인
- 운영 메모: 인증서 1년 만료, 업그레이드는 마이너 버전 한 단계씩
- README(구조와 아키텍처) 작성, done 문서 작성, calendar를 DONE으로 변경

## 진행 방식

| 구분 | 내용 |
| --- | --- |
| 서버에서 직접 실행 (1~3단계) | 명령어를 정리해 드리면 직접 실행하거나 실행 방법을 함께 정함. 이 세션에서는 서버에 접속하지 않음 |
| gitops와 앱 레포 (4~9단계) | Claude가 파일을 작성하고, 커밋과 push는 사용자가 직접 진행 |
| 기록 | 단계 묶음마다 `task1.md`, `task2.md` ...에 기록 |

- 구현 시작 지시가 있으면 calendar를 TASK로 바꾸고 0단계부터 진행

---

# 추가1

- 구현순서에 궁금함이 있음
- 그냥 서버에 k8s, helm 설치하고 gitops 작성한 후에, gitops clone해서 kube apply 하면 안됨?
- cloudfalre tunnel은 구동 완료된 후에 도메인 구매해서 붙일거임 (2차로)
- api yml 작성 -> gitops 작성 -> 서버에 k8s 설치 -> CI/CD 구축 -> 배포 -> kube apply -> cloudflare tunnel 도메인 작업
- 이 순서는 안되는건가?

---

# 추가1 분석

> plan4 추가 1 확인

## 답변

- 큰 틀은 제안한 순서로 가능. 다만 그대로는 막히는 지점이 세 군데 있어 그 부분만 조정

## clone 후 kubectl apply만 하는 방식

- 가능은 하지만, 레포 전체를 한 번에 apply하면 아래 문제로 실패

| 문제 | 이유 |
| --- | --- |
| 순서 | Gateway, HTTPRoute, CloudNativePG Cluster, SealedSecret은 CRD가 먼저 있어야 생성됨. 한 번에 apply하면 "그런 종류 없음" 오류로 여러 번 나눠 apply 필요 |
| Helm 차트 | ArgoCD, Envoy Gateway, CloudNativePG, Sealed Secrets는 Helm 차트라 `kubectl apply`로 바로 설치되지 않고 `helm install`을 따로 해야 함 |
| 큰 CRD | Envoy Gateway CRD는 너무 커서 `--server-side` 옵션 없이는 apply 실패 |
| 자동 배포 | 손으로 apply하면 Actions가 이미지 태그를 커밋해도 클러스터에 반영되지 않음. 자동 배포에는 ArgoCD 필요 |

- 그래서 손으로 하는 apply는 처음 한 번만 함

```bash
helm install argocd ...               # ArgoCD 설치
kubectl apply -f bootstrap/root.yaml  # "이 레포를 보고 알아서 설치해" 등록
```

- 나머지 순서 문제는 ArgoCD가 sync wave로 처리. 사실상 "clone하고 apply"와 같은 방식

## gitops를 먼저 작성할 때 막히는 것: SealedSecret

- SealedSecret은 클러스터에 설치된 Sealed Secrets 컨트롤러의 키로 암호화함
- 클러스터가 생기기 전에는 만들 수 없음 (Redis 비밀번호, JWT secret, Tunnel 토큰)
- gitops를 미리 작성할 때 SealedSecret만 비워두고, 컨트롤러가 뜬 뒤 `kubeseal`로 만들어 커밋

## 순서에서 빠진 것: Actions 워크플로

- 앱이 배포되려면 GHCR에 이미지가 먼저 있어야 함
- 앱 레포의 Actions 워크플로(빌드, GHCR push, gitops 태그 커밋)가 kube apply 전에 필요

## Cloudflare Tunnel과 도메인을 2차로 미루는 것

- 가능. ArgoCD와 Actions는 모두 서버에서 바깥으로 나가는 연결만 쓰므로 도메인 없이도 CI/CD는 끝까지 동작
- 1차에서는 바깥에서 브라우저로 접속할 수 없으므로 확인 범위는 다음과 같음
  - ArgoCD에서 모든 앱이 Synced, Healthy인지
  - `kubectl port-forward`와 `curl`로 member-api 헬스체크, 로그인 API 응답 확인
  - PostgreSQL과 Redis 연결, 데이터 폴더 생성 확인
- 브라우저로 로그인 흐름이나 CORS를 확인하는 것은 2차에서. CORS와 쿠키가 실제 도메인과 https 기준으로 설정되기 때문

## 조정한 순서

| 순서 | 내용 | 원래 제안 순서와 비교 |
| --- | --- | --- |
| 1 | api yml 작성 (member-api stg 프로필) | 같음 |
| 2 | 앱 레포 Actions 워크플로 작성 (빌드, GHCR push, gitops 태그 커밋) | 추가 |
| 3 | gitops 작성 (SealedSecret 제외), GitHub에 push | 같음 (push는 사용자가 직접) |
| 4 | 서버에 k8s 설치 (노드 준비, `kubeadm init`, Cilium) | 같음 |
| 5 | ArgoCD Helm 설치, root 등록 → 나머지는 ArgoCD가 설치 | "kube apply"를 이것으로 |
| 6 | `kubeseal`로 SealedSecret 생성, 커밋 | 추가 |
| 7 | 1차 확인 (port-forward, curl) | 추가 |
| 8 | (2차) 도메인 구매, Cloudflare Tunnel 연결, 브라우저로 확인 | 같음 |

- 위 "자원 확정과 구현 순서"의 0~10단계와 내용은 같고, gitops를 먼저 다 작성하고 서버 작업을 나중에 몰아서 하는 방식으로 순서만 바뀜
- [ ] 이 순서로 확정할지

---

# 구현 순서 확정

> 이순서가 좋을거 같음 이거로 확정하자

## 답변

- 추가1 분석의 조정한 순서로 확정

## 확정 순서

| 순서 | 내용 | 하는 사람 |
| --- | --- | --- |
| 1 | member-api stg 프로필 (`application-stg.yml`, group 추가) | Claude 작성 |
| 2 | 앱 레포 Actions 워크플로 (member-api, front) | Claude 작성 |
| 3 | gitops 작성 (SealedSecret 제외) | Claude 작성, push는 사용자가 직접 |
| 4 | 서버에 k8s 설치 (노드 준비, `kubeadm init`, Cilium) | 명령어는 Claude가 정리, 실행은 서버에서 사용자가 직접 |
| 5 | ArgoCD Helm 설치, root 등록 | 서버에서 사용자가 직접 |
| 6 | `kubeseal`로 SealedSecret 생성, 커밋 | 함께 진행 |
| 7 | 1차 확인 (port-forward, curl) | 함께 진행 |
| 8 | (2차) 도메인 구매, Cloudflare Tunnel 연결 | 추후 |

## 구현 시작 시 진행 방식

1. calendar를 `TASK:k8s 구축 작업(k8s-setup)`으로 변경
2. 1단계 member-api 작업부터 시작. 작성 전에 고칠 파일과 내용을 먼저 보여주고 진행
3. 단계 묶음마다 `task1.md`부터 기록

- [x] 구현 순서: 위 순서로 확정
