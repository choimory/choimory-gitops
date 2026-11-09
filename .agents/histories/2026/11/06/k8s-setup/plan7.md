# 목차

- [개요](#개요)
- [4단계 서버 설치 명령어](#4단계-서버-설치-명령어)
  - [답변](#답변)
  - [0. 사전 확인](#0-사전-확인)
  - [1. 노드 준비](#1-노드-준비)
  - [2. 클러스터 생성](#2-클러스터-생성)
  - [3. Cilium 설치](#3-cilium-설치)
  - [4. Mac에서 접속 확인](#4-mac에서-접속-확인)
  - [끝나면 확인할 것](#끝나면-확인할-것)
  - [문제가 생기면](#문제가-생기면)
  - [결정 필요 사항](#결정-필요-사항)
- [노션 대조 후 명령어 보강](#노션-대조-후-명령어-보강)
  - [답변](#답변-1)
  - [실행 위치](#실행-위치)
  - [Mac 사전 준비](#mac-사전-준비)
  - [0번 전에 추가 점검](#0번-전에-추가-점검)
  - [2번 kubeconfig 권한](#2번-kubeconfig-권한)
  - [3번 뒤 Pod DNS와 기존 서비스 확인](#3번-뒤-pod-dns와-기존-서비스-확인)
  - [4번 kubeconfig 권한](#4번-kubeconfig-권한)
  - [재부팅 복구 확인](#재부팅-복구-확인)
  - [디스크 운영 주의](#디스크-운영-주의)
  - [RAM 확인 결과에 따른 처리](#ram-확인-결과에-따른-처리)
- [Cilium 역할과 설치 이유](#cilium-역할과-설치-이유)
  - [답변](#답변-2)
  - [이 클러스터에서 하는 일](#이-클러스터에서-하는-일)
  - [왜 Cilium인가](#왜-cilium인가)
  - [맡기지 않은 일](#맡기지-않은-일)
  - [ArgoCD로 설치하지 않는 이유](#argocd로-설치하지-않는-이유)
- [관리형 k8s와 온프레미스 설치 차이](#관리형-k8s와-온프레미스-설치-차이)
  - [답변](#답변-3)
  - [구성요소별 비교](#구성요소별-비교)
  - [관리형의 CNI](#관리형의-cni)

---

# 개요

- 구현 순서 4단계(서버에 k8s 설치)의 명령어를 정리함
- 서버에서 사용자가 직접 실행하며, 이 세션에서는 서버에 접속하지 않음
- 스펙과 자원 값은 [plan3.md](./plan3.md), 구현 순서는 [plan4.md](./plan4.md), gitops 작성 내용은 [task3.md](./task3.md) 참고

---

# 4단계 서버 설치 명령어

> 4단계 서버 설치 명령어 정리해줘

## 답변

- 서버에서 직접, 위에서부터 순서대로 실행
- 이 명령어는 정리만 했고 실제로 실행해 보지 않음
- Kubernetes는 최신 1.37이 아니라 v1.36.6으로 정함. Cilium 1.20의 지원 범위가 1.36까지임
- 사용 버전: Kubernetes v1.36.6, Cilium 1.20.4, Helm v4.3.1

## 0. 사전 확인

- Docker 앱 데이터의 백업 상태를 먼저 확인하고 사용이 적은 시간에 진행

```bash
containerd --version
cat /etc/os-release | head -3
sudo cat /etc/containerd/config.toml | head -20
docker ps --format '{{.Names}}: {{.Status}}' | tee ~/docker-before.txt
ip route get 1.1.1.1      # 나오는 src IP가 고정 IP인지 확인
tailscale status | head -3
```

## 1. 노드 준비

```bash
# 데이터 폴더
sudo mkdir -p /data/dev/choimory/k8s /data/dev/choimory-dev/k8s

# swap 끄기
sudo swapoff -a
sudo sed -i '/\/swap.img/s/^/#/' /etc/fstab

# 커널 모듈과 네트워크 설정
cat <<'EOF' | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter

cat <<'EOF' | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
sudo sysctl --system
```

### containerd 설정

- 이 단계에서 Docker 앱이 영향을 받을 수 있음

```bash
sudo cp /etc/containerd/config.toml /etc/containerd/config.toml.bak
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
grep -n 'SystemdCgroup' /etc/containerd/config.toml   # true인지 확인. 안 나오면 멈추고 확인
sudo systemctl restart containerd
sudo systemctl status containerd --no-pager | head -5
docker ps --format '{{.Names}}: {{.Status}}'          # 앱이 모두 떠 있는지 확인
```

### kubeadm, kubelet, kubectl, Helm 설치

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.36/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet

curl -fsSL https://get.helm.sh/helm-v4.3.1-linux-amd64.tar.gz | tar -xz -C /tmp
sudo install /tmp/linux-amd64/helm /usr/local/bin/helm
kubeadm version -o short; helm version --short
```

## 2. 클러스터 생성

- `<TAILSCALE_IP>`와 `<TS_NET_NAME>`을 실제 값으로 변경

```bash
cat <<'EOF' > ~/kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta4
kind: InitConfiguration
nodeRegistration:
  criSocket: unix:///run/containerd/containerd.sock
---
apiVersion: kubeadm.k8s.io/v1beta4
kind: ClusterConfiguration
kubernetesVersion: v1.36.6
networking:
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12
apiServer:
  certSANs:
    - choimory-server
    - <TS_NET_NAME>
    - <TAILSCALE_IP>
---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: systemd
systemReserved:
  memory: 10Gi
  cpu: "2"
kubeReserved:
  memory: 1Gi
  cpu: 500m
evictionHard:
  memory.available: 500Mi
  nodefs.available: 10%
  nodefs.inodesFree: 5%
  imagefs.available: 15%
EOF

sudo kubeadm init --config ~/kubeadm-config.yaml

mkdir -p ~/.kube
sudo cp /etc/kubernetes/admin.conf ~/.kube/config
sudo chown "$(id -u):$(id -g)" ~/.kube/config

# 노드가 1대라 control-plane에도 Pod를 띄운다
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

## 3. Cilium 설치

```bash
cat <<'EOF' > ~/cilium-values.yaml
ipam:
  mode: kubernetes     # kubeadm의 podSubnet(10.244.0.0/16)을 사용
operator:
  replicas: 1
hubble:
  enabled: false
gatewayAPI:
  enabled: false       # Gateway는 Envoy Gateway를 사용
ingressController:
  enabled: false
EOF

helm repo add cilium https://helm.cilium.io
helm repo update
helm install cilium cilium/cilium --version 1.20.4 -n kube-system -f ~/cilium-values.yaml

kubectl -n kube-system rollout status ds/cilium --timeout=5m
kubectl get nodes -o wide       # STATUS가 Ready인지 확인
kubectl get pods -A
```

## 4. Mac에서 접속 확인

- Mac에서 실행
- admin 권한 파일이므로 다른 곳에 공유하지 않음

```bash
scp <서버계정>@choimory-server:.kube/config ~/.kube/choimory-server
kubectl --kubeconfig ~/.kube/choimory-server config set-cluster kubernetes --server=https://choimory-server:6443
kubectl --kubeconfig ~/.kube/choimory-server get nodes
```

## 끝나면 확인할 것

- 노드가 `Ready`이고 `kube-system` Pod가 모두 Running인지
- `docker ps` 결과가 `~/docker-before.txt`와 같은지
- Mac에서 `kubectl get nodes`가 되는지

## 문제가 생기면

- `kubeadm init` 이전에 멈췄다면 containerd 설정만 되돌림. 이후라면 초기화도 함

```bash
sudo kubeadm reset -f                                                  # init 이후에만
sudo cp /etc/containerd/config.toml.bak /etc/containerd/config.toml
sudo systemctl restart containerd
```

## 결정 필요 사항

- [ ] `kubeadm-config.yaml`, `cilium-values.yaml`을 재설치용으로 gitops `bootstrap/`에 남길지 (레포가 public이므로 Tailscale 주소는 마스킹)

---

# 노션 대조 후 명령어 보강

> 노션도 문서 다 확인한다음에 기존 plan 문서랑 대조해봐

## 답변

- 노션 메인, 3차 `최종 작업 진행`, 4차·5차 작업 진행과 QNA를 대조해 빠진 절차를 추가함
- 구현 방향, gitops, 버전, Pod 자원 값은 바뀌지 않음
- 이 명령어도 실제로 실행해 보지 않음

## 실행 위치

| 보강 항목 | 넣을 위치 |
| --- | --- |
| Mac 사전 준비 | 시작 전, Mac |
| 추가 점검 | 0번 사전 확인 전, 서버 |
| kubeconfig 권한 | 2번 kubeconfig 복사 직후, 서버 |
| Pod DNS와 기존 서비스 확인 | 3번 Cilium 설치 뒤, 서버 |
| kubeconfig 권한 | 4번 `scp` 직후, Mac |
| 재부팅 복구 확인 | 4번까지 끝난 뒤, 서버 |

## Mac 사전 준비

- 기록상 Mac의 `kubectl`은 v1.32.2. 서버 v1.36과의 지원 범위(±1 마이너)를 벗어남

```bash
which kubectl
kubectl version --client
brew upgrade kubernetes-cli   # brew로 설치한 경우. v1.35 이상인지 다시 확인
```

## 0번 전에 추가 점검

```bash
free -h                    # 전체 RAM 확인. 하단 RAM 처리 참고
df -h /                    # 사용률이 80% 미만인지 확인
swapon --show              # swap 파일 위치 확인
grep -n swap /etc/fstab
sudo ss -tlnp | grep -E ':(6443|10250|10257|10259|2379|2380) '   # 아무것도 안 나와야 함
sudo ufw status            # inactive 기대. active면 멈추고 규칙 확인
tailscale ip -4            # <TAILSCALE_IP>에 사용
tailscale status --json | grep -m1 '"DNSName"'   # <TS_NET_NAME>에 사용. 끝의 점은 제외
```

- `swapon --show`가 `/swap.img`가 아니면 1번의 `sed` 패턴을 실제 경로로 바꿔서 실행

## 2번 kubeconfig 권한

- `chown` 다음에 실행

```bash
chmod 600 ~/.kube/config
```

## 3번 뒤 Pod DNS와 기존 서비스 확인

- Tailscale MagicDNS가 서버 DNS를 건드리면 Pod의 외부 도메인 조회가 실패할 수 있음

```bash
kubectl run dns-test --rm -it --restart=Never --image=busybox:1.37 -- nslookup kubernetes.default.svc.cluster.local
kubectl run dns-test --rm -it --restart=Never --image=busybox:1.37 -- nslookup github.com

# Cilium 설치 뒤 Tailscale과 Docker 앱 확인. 차이가 없어야 함
tailscale status | head -3
diff <(docker ps --format '{{.Names}}' | sort) <(cut -d: -f1 ~/docker-before.txt | sort)
```

- Uptime Kuma에서도 기존 앱이 정상인지 확인. containerd 재시작 때 잠깐 다운 알림이 올 수 있음
- `github.com` 조회가 실패하면 멈추고 아래 결과를 확인

```bash
kubectl -n kube-system logs deploy/coredns --tail=30
cat /run/systemd/resolve/resolv.conf     # 100.100.100.100이 있는지 확인
```

- 후보 조치는 `sudo tailscale set --accept-dns=false`. 서버의 Tailscale 이름 조회에 영향을 주므로 원인 확인 후 적용

## 4번 kubeconfig 권한

- `scp` 다음에 Mac에서 실행

```bash
chmod 600 ~/.kube/choimory-server
```

## 재부팅 복구 확인

- Docker 앱도 함께 재시작되므로 사용이 적은 시간에 진행

```bash
sudo reboot

# 재부팅 후 다시 접속해서 확인
swapon --show                                         # 아무것도 안 나와야 함
systemctl is-active containerd docker kubelet tailscaled
kubectl get nodes
kubectl get pods -A
diff <(docker ps --format '{{.Names}}' | sort) <(cut -d: -f1 ~/docker-before.txt | sort)
```

- Mac에서 `kubectl --kubeconfig ~/.kube/choimory-server get nodes`도 다시 확인

## 디스크 운영 주의

- `/data`, Docker 이미지, kubelet이 같은 SSD(약 465.9GiB)를 사용
- 설정상 사용률 85%(`imagefs.available: 15%`)부터 이미지 정리와 Pod 축출이 시작될 수 있고, DB Pod도 대상이 될 수 있음
- Uptime Kuma 80% 알림과 여유가 5%뿐이므로 알림을 받으면 바로 정리

## RAM 확인 결과에 따른 처리

- 5차는 32GB, 4차 QNA는 26GB로 기록되어 있어 `free -h`로 확정

| 실제 RAM | k8s 할당 가능량 | 처리 |
| --- | --- | --- |
| 32GB | 약 20.5Gi | 변경 없음 |
| 26GB | 약 14.5Gi | 예상 최대 5.6Gi에 충분하므로 설치는 그대로 진행. plan3의 계산 근거만 정정 |

---

# Cilium 역할과 설치 이유

> cilium 역할이랑 설치이유 설명좀

## 답변

- Cilium은 Pod끼리 통신하게 해 주는 네트워크 플러그인(CNI)
- kubeadm은 CNI를 포함하지 않아서, 설치 전에는 노드가 `NotReady`이고 일반 Pod도 뜨지 않음. CNI 자체는 필수

## 이 클러스터에서 하는 일

| 역할 | 내용 |
| --- | --- |
| Pod IP 할당 | `10.244.0.0/16` 대역에서 Pod마다 IP를 줌 (`ipam.mode: kubernetes`) |
| Pod 네트워크 연결 | Pod와 노드를 연결해 Pod 간 통신과 외부 통신을 가능하게 함 |
| NetworkPolicy 적용 | stg와 prod 사이를 실제로 차단 |

- gitops의 NetworkPolicy는 같은 네임스페이스와 `infrastructure`에서 오는 요청만 허용
- 이 규칙은 CNI가 지원해야만 동작함. Flannel은 NetworkPolicy를 적용하지 않아 stg에서 prod로 그대로 통신됨

## 왜 Cilium인가

- NetworkPolicy 지원으로 차단 요구사항을 충족
- 리눅스 커널에서 패킷을 처리하는 eBPF 기반이고, 실무에서 많이 쓰여 kubeadm을 고른 목적(실무 경험)과 맞음
- Calico도 같은 요구를 충족하므로 Cilium만 가능한 것은 아님. 노드가 1대라 성능 이점도 크지 않음

## 맡기지 않은 일

| 기능 | 설정 | 이유 |
| --- | --- | --- |
| Hubble (트래픽 관측) | 끔 | 메모리 절약 |
| Gateway API, Ingress | 끔 | 외부 요청 라우팅은 Envoy Gateway가 담당 |
| kube-proxy 대체 | 켜지 않음 | Service 라우팅은 kubeadm 기본 kube-proxy가 담당 |

- Cilium은 Pod 사이의 통신과 차단, Envoy Gateway는 `/member` 같은 경로로 들어온 HTTP 요청의 전달을 맡음
- 기존 Docker 앱은 Docker 자체 네트워크(`172.x`)를 그대로 쓰고 대역도 겹치지 않음

## ArgoCD로 설치하지 않는 이유

- ArgoCD도 Pod로 실행되므로 CNI가 먼저 있어야 함
- Cilium만 서버에서 Helm으로 직접 설치하고, 나머지는 ArgoCD가 설치

---

# 관리형 k8s와 온프레미스 설치 차이

> eks나 nks는 이미 설치되어있는거고 k8s 온프레미스 설치할때는 해줘야하는건가?

## 답변

- EKS나 NKS는 클러스터를 만들 때 CNI까지 준비되어 있고, kubeadm으로 직접 구축하면 설치해야 함
- 온프레미스 전체가 그런 것은 아님. k3s처럼 Flannel을 기본으로 포함한 배포판도 있음. kubeadm이 클러스터 핵심만 설치하는 도구라서 그러함

## 구성요소별 비교

| 구성요소 | EKS/NKS | kubeadm |
| --- | --- | --- |
| 컨트롤 플레인, etcd | 클라우드가 운영 | 직접 설치·백업·업그레이드 |
| 컨테이너 런타임 | 노드에 포함 | 직접 설치 (이번엔 containerd 공유) |
| CNI | 기본 제공 | 직접 설치 (Cilium) |
| CoreDNS, kube-proxy | 기본 제공 | kubeadm이 설치 |
| 스토리지 | 클라우드 디스크 연동 (EKS의 EBS CSI는 애드온 설치 필요) | 직접 설치 (local-path) |
| 외부 노출 | 클라우드 LB 연동 | 직접 구성 (Cloudflare Tunnel 예정) |
| metrics-server | 대체로 별도 설치 | 직접 설치 |

## 관리형의 CNI

- 관리형도 내부에서는 CNI가 똑같이 동작하며, 클라우드가 대신 골라서 넣어 준 것
- EKS: 기본은 AWS VPC CNI로 Pod가 VPC IP를 직접 받음. 서브넷 IP 부족 문제가 생기기 쉬움. Cilium으로 바꾸거나 함께 쓸 수도 있음
- NKS: Cilium 기반으로 알고 있으나 시점과 클러스터 유형에 따라 다를 수 있어 콘솔이나 공식 문서로 확인 필요
- 실제 클러스터에서는 `kubectl get pods -n kube-system`으로 어떤 CNI가 떠 있는지 확인 가능
- NetworkPolicy도 관리형이라고 항상 바로 동작하지는 않음. EKS의 VPC CNI는 네트워크 정책 기능을 따로 켜야 함
