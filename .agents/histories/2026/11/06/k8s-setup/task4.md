# 목차

- [개요](#개요)
- [4단계 실행 명령어](#4단계-실행-명령어)
  - [1. 시작 전 (Mac)](#1-시작-전-mac)
  - [2. 사전 점검 (서버)](#2-사전-점검-서버)
  - [3. 노드 준비 (서버)](#3-노드-준비-서버)
    - [데이터 폴더](#데이터-폴더)
    - [swap 끄기](#swap-끄기)
    - [커널 모듈과 네트워크 설정](#커널-모듈과-네트워크-설정)
    - [containerd 설정](#containerd-설정)
    - [kubeadm, kubelet, kubectl, Helm 설치](#kubeadm-kubelet-kubectl-helm-설치)
  - [4. 클러스터 생성 (서버)](#4-클러스터-생성-서버)
  - [5. Cilium 설치 (서버)](#5-cilium-설치-서버)
  - [6. DNS와 기존 서비스 확인 (서버)](#6-dns와-기존-서비스-확인-서버)
  - [7. 접속 확인 (Mac)](#7-접속-확인-mac)
  - [8. 재부팅 복구 확인 (서버)](#8-재부팅-복구-확인-서버)
- [진행](#진행)
  - [2.사전 점검](#2사전-점검)
  - [3. 노드 준비](#3-노드-준비)
  - [4. 클러스터 생성](#4-클러스터-생성)

---

# 개요

- 구현 순서 4단계(서버에 k8s 설치)에서 실행할 명령어만 순서대로 정리함
- 설명, 판단 기준, 문제 발생 시 복구 방법은 [plan7.md](./plan7.md) 참고
- 명령어는 실제로 실행해 보지 않았음
- 사전 조건: gitops 변경이 `main`에 push되어 있어야 함

---

# 4단계 실행 명령어

> 이제 명령어 하나씩 실행할건데 해야할 명령어만 task4에 작성해줘

## 1. 시작 전 (Mac)

```bash
which kubectl
kubectl version --client
brew upgrade kubernetes-cli
kubectl version --client
```

- `which kubectl`: Mac에서 실행되는 kubectl의 설치 경로를 확인한다.
- `kubectl version --client`: 업그레이드 전 kubectl 버전을 확인한다.
- `brew upgrade kubernetes-cli`: Homebrew로 설치한 kubectl을 최신 버전으로 올린다.
- `kubectl version --client`: 업그레이드 후 버전을 다시 확인한다.

## 2. 사전 점검 (서버)

```bash
free -h
df -h /
swapon --show
grep -n swap /etc/fstab
sudo ss -tlnp | grep -E ':(6443|10250|10257|10259|2379|2380) '
sudo ufw status
tailscale ip -4
tailscale status --json | grep -m1 '"DNSName"'
containerd --version
cat /etc/os-release | head -3
sudo cat /etc/containerd/config.toml | head -20
docker ps --format '{{.Names}}: {{.Status}}' | tee ~/docker-before.txt
ip route get 1.1.1.1
tailscale status | head -3
```

- `free -h`: 전체 RAM, 사용량, 여유량과 swap 사용량을 확인한다.
- `df -h /`: 루트 파일시스템의 디스크 용량과 사용률을 확인한다.
- `swapon --show`: 현재 켜져 있는 swap의 경로와 크기를 확인한다.
- `grep -n swap /etc/fstab`: 부팅 시 swap을 켜는 설정의 행 번호와 내용을 확인한다.
- `sudo ss -tlnp | grep -E ...`: k8s가 쓸 포트를 이미 사용하는 프로세스가 있는지 확인한다. 6443은 API 서버, 10250은 kubelet, 10257은 controller-manager, 10259는 scheduler, 2379와 2380은 etcd가 사용한다. 아무것도 출력되지 않으면 비어 있는 것이다.
- `sudo ufw status`: 방화벽이 켜져 있는지 확인한다.
- `tailscale ip -4`: 인증서 SAN에 넣을 서버의 Tailscale IPv4 주소를 확인한다.
- `tailscale status --json | grep -m1 '"DNSName"'`: 인증서 SAN에 넣을 서버의 Tailscale DNS 이름을 확인한다.
- `containerd --version`: containerd 버전을 확인한다. 1.x와 2.x는 설정 파일 구조가 다르다.
- `cat /etc/os-release | head -3`: Ubuntu 버전을 확인한다.
- `sudo cat /etc/containerd/config.toml | head -20`: 현재 containerd 설정과 CRI 플러그인 비활성화 여부를 확인한다.
- `docker ps --format ... | tee ~/docker-before.txt`: 실행 중인 Docker 앱 목록을 출력하고, 설치 후 비교할 수 있도록 파일로 저장한다.
- `ip route get 1.1.1.1`: 외부 통신에 쓰는 네트워크 인터페이스와 서버 IP를 확인한다. k8s 노드 IP를 예상하는 근거가 된다.
- `tailscale status | head -3`: Tailscale이 연결되어 있는지 확인한다.

## 3. 노드 준비 (서버)

### 데이터 폴더

```bash
sudo mkdir -p /data/dev/choimory/k8s /data/dev/choimory-dev/k8s
```

- `sudo mkdir -p ...`: local-path StorageClass가 PVC 데이터를 저장할 루트 폴더 2개를 만든다. `-p`는 상위 폴더가 없으면 함께 만들고, 이미 있어도 오류를 내지 않는다.

### swap 끄기

- `swapon --show`의 경로가 `/swap.img`가 아니면 `sed`의 경로를 변경

```bash
sudo swapoff -a
sudo sed -i '/\/swap.img/s/^/#/' /etc/fstab
```

- `sudo swapoff -a`: 현재 켜져 있는 swap을 모두 끈다. swap에 있던 내용은 RAM으로 다시 옮겨진다. 재부팅 전까지만 유지된다.
- `sudo sed -i ... /etc/fstab`: `/swap.img` 행을 주석 처리해 재부팅 후에도 swap이 켜지지 않게 한다.

### 커널 모듈과 네트워크 설정

```bash
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

- `cat <<'EOF' | sudo tee /etc/modules-load.d/k8s.conf`: 부팅할 때 `overlay`, `br_netfilter` 커널 모듈을 자동으로 불러오도록 설정 파일을 만든다.
- `sudo modprobe overlay`: 컨테이너 이미지 레이어를 합쳐 파일시스템으로 보여 주는 overlay 모듈을 지금 불러온다.
- `sudo modprobe br_netfilter`: 브리지를 지나는 트래픽을 iptables가 처리할 수 있게 하는 모듈을 지금 불러온다.
- `cat <<'EOF' | sudo tee /etc/sysctl.d/k8s.conf`: 브리지 트래픽의 iptables 처리와 IP 포워딩을 켜는 커널 설정 파일을 만든다.
- `sudo sysctl --system`: 모든 sysctl 설정 파일을 다시 읽어 재부팅 없이 지금 적용한다.

### containerd 설정

```bash
sudo cp /etc/containerd/config.toml /etc/containerd/config.toml.bak
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
grep -n 'SystemdCgroup' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl status containerd --no-pager | head -5
docker ps --format '{{.Names}}: {{.Status}}'
```

- `sudo cp ... config.toml.bak`: 문제가 생기면 되돌릴 수 있도록 기존 containerd 설정을 백업한다.
- `containerd config default | sudo tee ... >/dev/null`: 현재 버전의 기본 설정으로 파일을 새로 만든다. 기존 설정의 `disabled_plugins = ["cri"]`도 이때 사라져 k8s가 containerd를 쓸 수 있게 된다.
- `sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' ...`: containerd의 cgroup 드라이버를 kubelet과 같은 systemd로 맞춘다.
- `grep -n 'SystemdCgroup' ...`: 값이 `true`로 바뀌었는지 재시작 전에 확인한다.
- `sudo systemctl restart containerd`: 바뀐 설정을 적용하기 위해 containerd를 재시작한다.
- `sudo systemctl status containerd --no-pager | head -5`: containerd가 정상 실행 중인지 확인한다.
- `docker ps --format ...`: containerd 재시작 후 기존 Docker 앱이 그대로 실행 중인지 확인한다.

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

- `sudo apt-get update`: 패키지 목록을 최신 상태로 갱신한다.
- `sudo apt-get install -y apt-transport-https ca-certificates curl gpg`: HTTPS 저장소 접근과 서명 키 처리에 필요한 패키지를 설치한다.
- `sudo mkdir -p -m 755 /etc/apt/keyrings`: apt 저장소 서명 키를 보관할 폴더를 만든다.
- `curl ... Release.key | sudo gpg --dearmor -o ...`: k8s 저장소의 서명 키를 내려받아 apt가 읽을 수 있는 바이너리 형식으로 저장한다.
- `echo 'deb [...] ...' | sudo tee ...kubernetes.list`: k8s v1.36 저장소를 apt 저장소 목록에 추가한다.
- `sudo apt-get update`: 추가한 k8s 저장소의 패키지 목록을 읽어 온다.
- `sudo apt-get install -y kubelet kubeadm kubectl`: 노드 에이전트(kubelet), 클러스터 생성 도구(kubeadm), 클러스터 조작 도구(kubectl)를 설치한다.
- `sudo apt-mark hold kubelet kubeadm kubectl`: `apt upgrade` 때 k8s 패키지가 의도치 않게 올라가지 않도록 버전을 고정한다.
- `sudo systemctl enable --now kubelet`: kubelet을 지금 시작하고 부팅 시 자동 시작되도록 등록한다. `kubeadm init` 전에는 재시작을 반복하는 것이 정상이다.
- `curl ... helm-v4.3.1-linux-amd64.tar.gz | tar -xz -C /tmp`: Helm 압축 파일을 내려받아 `/tmp`에 푼다.
- `sudo install /tmp/linux-amd64/helm /usr/local/bin/helm`: Helm 실행 파일을 실행 권한과 함께 PATH 경로에 복사한다.
- `kubeadm version -o short; helm version --short`: 설치된 kubeadm과 Helm의 버전을 확인한다.

## 4. 클러스터 생성 (서버)

- `<실제_TS_NET_NAME>`, `<실제_TAILSCALE_IP>`는 2번에서 확인한 값으로 변경. DNSName 끝의 점은 제외
- 실제 값은 홈 디렉토리의 복사본에만 넣음

```bash
git clone https://github.com/choimory/choimory-gitops.git ~/choimory-gitops
cp ~/choimory-gitops/bootstrap/kubeadm-config.yaml ~/choimory-gitops/bootstrap/cilium-values.yaml ~/
sed -i 's/<TS_NET_NAME>/<실제_TS_NET_NAME>/; s/<TAILSCALE_IP>/<실제_TAILSCALE_IP>/' ~/kubeadm-config.yaml
grep -A3 certSANs ~/kubeadm-config.yaml

sudo kubeadm init --config ~/kubeadm-config.yaml

mkdir -p ~/.kube
sudo cp /etc/kubernetes/admin.conf ~/.kube/config
sudo chown "$(id -u):$(id -g)" ~/.kube/config
chmod 600 ~/.kube/config

kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

- `git clone ...`: `bootstrap/` 설정 파일을 가져오기 위해 gitops 레포를 서버에 내려받는다.
- `cp ... ~/`: 실제 Tailscale 값을 레포 파일에 넣지 않도록 설정 파일 2개를 홈 디렉토리에 복사한다.
- `sed -i 's/<TS_NET_NAME>/.../; s/<TAILSCALE_IP>/.../' ~/kubeadm-config.yaml`: 홈 디렉토리 복사본의 placeholder를 실제 값으로 바꾼다.
- `grep -A3 certSANs ~/kubeadm-config.yaml`: 인증서 SAN에 들어갈 값이 올바르게 바뀌었는지 확인한다.
- `sudo kubeadm init --config ~/kubeadm-config.yaml`: 사전 점검을 거쳐 인증서, etcd, API 서버, controller-manager, scheduler, CoreDNS, kube-proxy를 구성해 컨트롤 플레인을 만든다.
- `mkdir -p ~/.kube`: kubectl 설정 파일을 둘 폴더를 만든다.
- `sudo cp /etc/kubernetes/admin.conf ~/.kube/config`: 클러스터 관리자 인증 정보를 kubectl 기본 설정 위치로 복사한다.
- `sudo chown "$(id -u):$(id -g)" ~/.kube/config`: 일반 계정으로 kubectl을 쓸 수 있도록 복사한 파일의 소유자를 현재 계정으로 바꾼다.
- `chmod 600 ~/.kube/config`: 관리자 인증 정보이므로 소유자만 읽고 쓸 수 있게 제한한다.
- `kubectl taint nodes --all node-role.kubernetes.io/control-plane-`: 노드가 1대라 일반 Pod도 컨트롤 플레인 노드에 배치되도록 taint를 제거한다. 끝의 `-`가 제거를 뜻한다.

## 5. Cilium 설치 (서버)

```bash
helm repo add cilium https://helm.cilium.io
helm repo update
helm install cilium cilium/cilium --version 1.20.4 -n kube-system -f ~/cilium-values.yaml

kubectl -n kube-system rollout status ds/cilium --timeout=5m
kubectl get nodes -o wide
kubectl get pods -A
```

- `helm repo add cilium https://helm.cilium.io`: Cilium Helm 차트 저장소를 등록한다.
- `helm repo update`: 등록된 저장소의 차트 목록을 갱신한다.
- `helm install cilium cilium/cilium --version 1.20.4 -n kube-system -f ~/cilium-values.yaml`: 준비한 values로 Cilium 1.20.4를 `kube-system` 네임스페이스에 설치한다.
- `kubectl -n kube-system rollout status ds/cilium --timeout=5m`: Cilium DaemonSet이 준비될 때까지 최대 5분 기다린다.
- `kubectl get nodes -o wide`: 노드가 `Ready`인지와 노드 IP, 런타임 정보를 확인한다.
- `kubectl get pods -A`: 모든 네임스페이스의 Pod 상태를 확인한다.

## 6. DNS와 기존 서비스 확인 (서버)

```bash
kubectl run dns-test --rm -it --restart=Never --image=busybox:1.37 -- nslookup kubernetes.default.svc.cluster.local
kubectl run dns-test --rm -it --restart=Never --image=busybox:1.37 -- nslookup github.com

tailscale status | head -3
diff <(docker ps --format '{{.Names}}' | sort) <(cut -d: -f1 ~/docker-before.txt | sort)
```

- `kubectl run dns-test ... nslookup kubernetes.default.svc.cluster.local`: 임시 Pod를 띄워 클러스터 내부 DNS 조회를 확인한다. `--rm`으로 종료 후 Pod가 삭제되고, `--restart=Never`로 한 번만 실행된다.
- `kubectl run dns-test ... nslookup github.com`: 임시 Pod에서 외부 도메인 조회가 되는지 확인한다.
- `tailscale status | head -3`: k8s 설치 후에도 Tailscale 연결이 유지되는지 확인한다.
- `diff <(docker ps ...) <(cut ... ~/docker-before.txt ...)`: 설치 전후 Docker 앱 이름을 비교한다. 아무것도 출력되지 않으면 목록이 같은 것이다.

## 7. 접속 확인 (Mac)

- `<서버계정>`은 실제 계정으로 변경

```bash
mkdir -p ~/.kube
scp <서버계정>@choimory-server:.kube/config ~/.kube/choimory-server
chmod 600 ~/.kube/choimory-server
kubectl --kubeconfig ~/.kube/choimory-server config set-cluster kubernetes --server=https://choimory-server:6443
kubectl --kubeconfig ~/.kube/choimory-server get nodes
```

- `mkdir -p ~/.kube`: Mac에 kubeconfig를 둘 폴더를 만든다.
- `scp <서버계정>@choimory-server:.kube/config ~/.kube/choimory-server`: 서버의 kubeconfig를 Mac으로 복사한다.
- `chmod 600 ~/.kube/choimory-server`: 관리자 인증 정보이므로 소유자만 읽고 쓸 수 있게 제한한다.
- `kubectl --kubeconfig ... config set-cluster kubernetes --server=https://choimory-server:6443`: kubeconfig의 API 서버 주소를 Tailscale로 접속 가능한 서버 이름으로 바꾼다.
- `kubectl --kubeconfig ... get nodes`: Mac에서 Tailscale을 통해 클러스터에 접속되는지 확인한다.

## 8. 재부팅 복구 확인 (서버)

```bash
sudo reboot
```

- `sudo reboot`: 재부팅 후에도 설정과 서비스가 자동으로 복구되는지 확인하기 위해 서버를 재시작한다.

재부팅 후 다시 접속해서 실행

```bash
swapon --show
systemctl is-active containerd docker kubelet tailscaled
kubectl get nodes
kubectl get pods -A
diff <(docker ps --format '{{.Names}}' | sort) <(cut -d: -f1 ~/docker-before.txt | sort)
```

- `swapon --show`: 재부팅 후에도 swap이 꺼져 있는지 확인한다. 아무것도 출력되지 않아야 한다.
- `systemctl is-active containerd docker kubelet tailscaled`: 핵심 서비스 4개가 자동으로 시작되었는지 확인한다.
- `kubectl get nodes`: 노드가 다시 `Ready`가 되었는지 확인한다.
- `kubectl get pods -A`: 시스템 Pod가 다시 실행 중인지 확인한다.
- `diff <(docker ps ...) <(cut ... ~/docker-before.txt ...)`: 기존 Docker 앱이 모두 다시 올라왔는지 비교한다. 아무것도 출력되지 않으면 목록이 같은 것이다.

Mac에서 실행

```bash
kubectl --kubeconfig ~/.kube/choimory-server get nodes
```

- `kubectl --kubeconfig ... get nodes`: 재부팅 후에도 Mac에서 Tailscale을 통해 클러스터에 접속되는지 확인한다.

---

# 진행

## 2.사전 점검

```shell
choimory@choimory-server:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:            26Gi       2.6Gi       4.0Gi       331Mi        20Gi        24Gi
Swap:          8.0Gi       606Mi       7.4Gi
choimory@choimory-server:~$ df -h /
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv  466G   94G  353G  22% /
choimory@choimory-server:~$ swapon --show
NAME      TYPE SIZE   USED PRIO
/swap.img file   8G 606.9M   -1
choimory@choimory-server:~$ grep -n swap /etc/fstab
14:/swap.img    none    swap    sw      0       0
choimory@choimory-server:~$ sudo ss -tlnp | grep -E ':(6443|10250|10257|10259|2379|2380) '
[sudo: authenticate] Password:       
choimory@choimory-server:~$ sudo ufw status
Status: inactive
choimory@choimory-server:~$ tailscale ip -4
<TAILSCALE_IP>
choimory@choimory-server:~$ tailscale status --json | grep -m1 '"DNSName"'
    "DNSName": "<TS_NET_NAME>.",
choimory@choimory-server:~$ containerd --version
containerd containerd v2.4.2 b7a25261e029d68978bbf290df0a62fc819b3c51
choimory@choimory-server:~$ cat /etc/os-release | head -3
PRETTY_NAME="Ubuntu 26.04.1 LTS"
NAME="Ubuntu"
VERSION_ID="26.04"
choimory@choimory-server:~$ sudo cat /etc/containerd/config.toml | head -20
#   Copyright 2018-2022 Docker Inc.

#   Licensed under the Apache License, Version 2.0 (the "License");
#   you may not use this file except in compliance with the License.
#   You may obtain a copy of the License at

#       http://www.apache.org/licenses/LICENSE-2.0

#   Unless required by applicable law or agreed to in writing, software
#   distributed under the License is distributed on an "AS IS" BASIS,
#   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#   See the License for the specific language governing permissions and
#   limitations under the License.

disabled_plugins = ["cri"]

#root = "/var/lib/containerd"
#state = "/run/containerd"
#subreaper = true
#oom_score = 0
choimory@choimory-server:~$ docker ps --format '{{.Names}}: {{.Status}}' | tee ~/docker-before.txt
immich_server: Up 6 days (healthy)
jellyfin: Up 6 days (healthy)
uptime-kuma: Up 6 days (healthy)
homepage: Up 6 days (healthy)
dockge: Up 6 days (healthy)
immich_postgres: Up 6 days (healthy)
immich_machine_learning: Up 6 days (healthy)
immich_redis: Up 6 days (healthy)
choimory@choimory-server:~$ ip route get 1.1.1.1
1.1.1.1 via 192.168.10.1 dev enp4s0 src 192.168.10.201 uid 1000 
    cache 
choimory@choimory-server:~$ tailscale status | head -3
<TAILSCALE_IP>     choimory-server   choimory@  linux    idle; offers exit node     
<TAILSCALE_IP>   <DEVICE_NAME>  choimory@  windows  -                          
<TAILSCALE_IP>   <DEVICE_NAME>     choimory@  iOS      idle, tx 1916 rx 2884      
choimory@choimory-server:~$ 
```

## 3. 노드 준비

```shell
choimory@choimory-server:~$ sudo mkdir -p /data/dev/choimory/k8s /data/dev/choimory-dev/k8s
[sudo: authenticate] Password:       
choimory@choimory-server:~$ sudo swapoff -a
sudo sed -i '/\/swap.img/s/^/#/' /etc/fstab
choimory@choimory-server:~$ ^C
choimory@choimory-server:~$ sudo swapoff -a
choimory@choimory-server:~$ sudo sed -i '/\/swap.img/s/^/#/' /etc/fstab
choimory@choimory-server:~$ cat <<'EOF' | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
overlay
br_netfilter
choimory@choimory-server:~$ sudo modprobe overlay
choimory@choimory-server:~$ sudo modprobe br_netfilter
choimory@choimory-server:~$ cat <<'EOF' | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
choimory@choimory-server:~$ sudo sysctl --system
* Applying /usr/lib/sysctl.d/10-apparmor.conf ...
* Applying /usr/lib/sysctl.d/10-coredump-debian.conf ...
* Applying /usr/lib/sysctl.d/50-default.conf ...
* Applying /usr/lib/sysctl.d/50-pid-max.conf ...
* Applying /usr/lib/sysctl.d/55-bufferbloat.conf ...
* Applying /usr/lib/sysctl.d/55-console-messages.conf ...
* Applying /usr/lib/sysctl.d/55-ipv6-privacy.conf ...
* Applying /usr/lib/sysctl.d/55-kernel-hardening.conf ...
* Applying /usr/lib/sysctl.d/55-magic-sysrq.conf ...
* Applying /usr/lib/sysctl.d/55-map-count.conf ...
* Applying /usr/lib/sysctl.d/55-network-security.conf ...
* Applying /usr/lib/sysctl.d/55-ptrace.conf ...
* Applying /usr/lib/sysctl.d/55-zeropage.conf ...
* Applying /etc/sysctl.d/99-tailscale.conf ...
* Applying /etc/sysctl.d/k8s.conf ...
kernel.apparmor_restrict_unprivileged_userns = 1
kernel.apparmor_restrict_unprivileged_unconfined = 1
kernel.core_pattern = core
kernel.sysrq = 0x01b6
kernel.core_uses_pid = 1
net.ipv4.conf.default.rp_filter = 2
net.ipv4.conf.br-098aed937424.rp_filter = 2
net.ipv4.conf.br-451e55596833.rp_filter = 2
net.ipv4.conf.br-4a4526e479f4.rp_filter = 2
net.ipv4.conf.br-5456f25db97b.rp_filter = 2
net.ipv4.conf.br-99a7fa05193f.rp_filter = 2
net.ipv4.conf.br-fe34dab8581e.rp_filter = 2
net.ipv4.conf.docker0.rp_filter = 2
net.ipv4.conf.eno1.rp_filter = 2
net.ipv4.conf.enp4s0.rp_filter = 2
net.ipv4.conf.lo.rp_filter = 2
net.ipv4.conf.tailscale0.rp_filter = 2
net.ipv4.conf.veth0b5ba37.rp_filter = 2
net.ipv4.conf.veth30772fb.rp_filter = 2
net.ipv4.conf.veth4052b66.rp_filter = 2
net.ipv4.conf.veth544faa2.rp_filter = 2
net.ipv4.conf.veth6192b41.rp_filter = 2
net.ipv4.conf.veth851615f.rp_filter = 2
net.ipv4.conf.veth9d63dfd.rp_filter = 2
net.ipv4.conf.vetha01fca4.rp_filter = 2
net.ipv4.conf.vethc08e109.rp_filter = 2
net.ipv4.conf.vethc5b5029.rp_filter = 2
net.ipv4.conf.vethcf39227.rp_filter = 2
net.ipv4.conf.vethd3a66ff.rp_filter = 2
net.ipv4.conf.vethd3eafab.rp_filter = 2
net.ipv4.conf.wlp2s0.rp_filter = 2
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.br-098aed937424.accept_source_route = 0
net.ipv4.conf.br-451e55596833.accept_source_route = 0
net.ipv4.conf.br-4a4526e479f4.accept_source_route = 0
net.ipv4.conf.br-5456f25db97b.accept_source_route = 0
net.ipv4.conf.br-99a7fa05193f.accept_source_route = 0
net.ipv4.conf.br-fe34dab8581e.accept_source_route = 0
net.ipv4.conf.docker0.accept_source_route = 0
net.ipv4.conf.eno1.accept_source_route = 0
net.ipv4.conf.enp4s0.accept_source_route = 0
net.ipv4.conf.lo.accept_source_route = 0
net.ipv4.conf.tailscale0.accept_source_route = 0
net.ipv4.conf.veth0b5ba37.accept_source_route = 0
net.ipv4.conf.veth30772fb.accept_source_route = 0
net.ipv4.conf.veth4052b66.accept_source_route = 0
net.ipv4.conf.veth544faa2.accept_source_route = 0
net.ipv4.conf.veth6192b41.accept_source_route = 0
net.ipv4.conf.veth851615f.accept_source_route = 0
net.ipv4.conf.veth9d63dfd.accept_source_route = 0
net.ipv4.conf.vetha01fca4.accept_source_route = 0
net.ipv4.conf.vethc08e109.accept_source_route = 0
net.ipv4.conf.vethc5b5029.accept_source_route = 0
net.ipv4.conf.vethcf39227.accept_source_route = 0
net.ipv4.conf.vethd3a66ff.accept_source_route = 0
net.ipv4.conf.vethd3eafab.accept_source_route = 0
net.ipv4.conf.wlp2s0.accept_source_route = 0
net.ipv4.conf.default.promote_secondaries = 1
net.ipv4.conf.br-098aed937424.promote_secondaries = 1
net.ipv4.conf.br-451e55596833.promote_secondaries = 1
net.ipv4.conf.br-4a4526e479f4.promote_secondaries = 1
net.ipv4.conf.br-5456f25db97b.promote_secondaries = 1
net.ipv4.conf.br-99a7fa05193f.promote_secondaries = 1
net.ipv4.conf.br-fe34dab8581e.promote_secondaries = 1
net.ipv4.conf.docker0.promote_secondaries = 1
net.ipv4.conf.eno1.promote_secondaries = 1
net.ipv4.conf.enp4s0.promote_secondaries = 1
net.ipv4.conf.lo.promote_secondaries = 1
net.ipv4.conf.tailscale0.promote_secondaries = 1
net.ipv4.conf.veth0b5ba37.promote_secondaries = 1
net.ipv4.conf.veth30772fb.promote_secondaries = 1
net.ipv4.conf.veth4052b66.promote_secondaries = 1
net.ipv4.conf.veth544faa2.promote_secondaries = 1
net.ipv4.conf.veth6192b41.promote_secondaries = 1
net.ipv4.conf.veth851615f.promote_secondaries = 1
net.ipv4.conf.veth9d63dfd.promote_secondaries = 1
net.ipv4.conf.vetha01fca4.promote_secondaries = 1
net.ipv4.conf.vethc08e109.promote_secondaries = 1
net.ipv4.conf.vethc5b5029.promote_secondaries = 1
net.ipv4.conf.vethcf39227.promote_secondaries = 1
net.ipv4.conf.vethd3a66ff.promote_secondaries = 1
net.ipv4.conf.vethd3eafab.promote_secondaries = 1
net.ipv4.conf.wlp2s0.promote_secondaries = 1
net.ipv4.ping_group_range = 0 2147483647
net.core.default_qdisc = fq_codel
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
fs.protected_regular = 2
fs.protected_fifos = 1
vm.max_map_count = 1048576
kernel.pid_max = 4194304
net.core.default_qdisc = fq_codel
kernel.printk = 4 4 1 7
net.ipv6.conf.all.use_tempaddr = 2
net.ipv6.conf.default.use_tempaddr = 2
kernel.kptr_restrict = 1
kernel.sysrq = 176
vm.max_map_count = 1048576
net.ipv4.conf.default.rp_filter = 2
net.ipv4.conf.all.rp_filter = 2
kernel.yama.ptrace_scope = 1
vm.mmap_min_addr = 65536
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
choimory@choimory-server:~$ sudo cp /etc/containerd/config.toml /etc/containerd/config.toml.bak
choimory@choimory-server:~$ containerd config default | sudo tee /etc/containerd/config.toml >/dev/null
choimory@choimory-server:~$ sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
choimory@choimory-server:~$ grep -n 'SystemdCgroup' /etc/containerd/config.toml
89:            SystemdCgroup = true
choimory@choimory-server:~$ sudo systemctl restart containerd
choimory@choimory-server:~$ sudo systemctl status containerd --no-pager | head -5
● containerd.service - containerd container runtime
     Loaded: loaded (/usr/lib/systemd/system/containerd.service; enabled; preset: enabled)
     Active: active (running) since Mon 2026-11-09 22:19:33 KST; 3s ago
 Invocation: 7743bc8cee204495ad2db015fb99b2ff
       Docs: https://containerd.io
choimory@choimory-server:~$ docker ps --format '{{.Names}}: {{.Status}}'
immich_server: Up 6 days (healthy)
jellyfin: Up 6 days (healthy)
uptime-kuma: Up 6 days (healthy)
homepage: Up 6 days (healthy)
dockge: Up 6 days (healthy)
immich_postgres: Up 7 days (healthy)
immich_machine_learning: Up 7 days (healthy)
immich_redis: Up 7 days (healthy)
choimory@choimory-server:~$ 
```

- 3. 노드준비의 containerd 설정까지

```shell
choimory@choimory-server:~$ sudo apt-get install -y apt-transport-https ca-certificates curl gpg
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
ca-certificates is already the newest version (20260816ubuntu0.26.04.2).
curl is already the newest version (8.18.0-1ubuntu2.8).
gpg is already the newest version (2.4.8-4ubuntu3.1).
gpg set to manually installed.
Solving dependencies... Done
The following NEW packages will be installed:
  apt-transport-https
0 upgraded, 1 newly installed, 0 to remove and 31 not upgraded.
Need to get 26.0 kB of archives.
After this operation, 36.9 kB of additional disk space will be used.
Get:1 http://kr.archive.ubuntu.com/ubuntu resolute-updates/universe amd64 apt-transport-https all 3.2.1ubuntu0.1 [26.0 kB]
Fetched 26.0 kB in 0s (850 kB/s)               
Selecting previously unselected package apt-transport-https.
(Reading database… 141272 files and directories currently installed.)
Preparing to unpack …/apt-transport-https_3.2.1ubuntu0.1_all.deb…
Unpacking apt-transport-https (3.2.1ubuntu0.1)…
Setting up apt-transport-https (3.2.1ubuntu0.1)…
Scanning processes...                                                                                                                                                                                                                                                                         
Scanning candidates...                                                                                                                                                                                                                                                                        
Scanning processor microcode...                                                                                                                                                                                                                                                               
Scanning linux images...                                                                                                                                                                                                                                                                      

Pending kernel upgrade!
Running kernel version:
  7.0.0-38-generic
Diagnostics:
  The currently running kernel version is not the expected kernel version 7.0.0-42-generic.

Restarting the system to load the new kernel will not be handled automatically, so you should consider rebooting.

The processor microcode seems to be up-to-date.

Restarting services...

Service restarts being deferred:
 /etc/needrestart/restart.d/dbus.service
 systemctl restart networkd-dispatcher.service
 systemctl restart systemd-logind.service
 systemctl restart unattended-upgrades.service

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
choimory@choimory-server:~$ sudo mkdir -p -m 755 /etc/apt/keyrings
choimory@choimory-server:~$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.36/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
choimory@choimory-server:~$ echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /
```

```shell
choimory@choimory-server:~$ sudo apt-get update
Hit:1 http://kr.archive.ubuntu.com/ubuntu resolute InRelease
Hit:2 http://kr.archive.ubuntu.com/ubuntu resolute-updates InRelease                                                                      
Hit:3 http://kr.archive.ubuntu.com/ubuntu resolute-backports InRelease                                                                    
Hit:4 https://download.docker.com/linux/ubuntu resolute InRelease                                                                         
Get:5 https://pkgs.tailscale.com/stable/ubuntu resolute InRelease                 
Hit:6 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.36/deb  InRelease
Hit:7 http://security.ubuntu.com/ubuntu resolute-security InRelease
Fetched 6,652 B in 1s (8,035 B/s)                      
Reading package lists... Done
choimory@choimory-server:~$ sudo apt-get install -y kubelet kubeadm kubectl
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Solving dependencies... Done
The following additional packages will be installed:
  kubernetes-cni
The following NEW packages will be installed:
  kubeadm kubectl kubelet kubernetes-cni
0 upgraded, 4 newly installed, 0 to remove and 31 not upgraded.
Need to get 76.8 MB of archives.
After this operation, 292 MB of additional disk space will be used.
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.36/deb  kubeadm 1.36.6-1.1 [12.6 MB]
Get:2 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.36/deb  kubectl 1.36.6-1.1 [11.8 MB]
Get:3 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.36/deb  kubernetes-cni 1.9.1-1.1 [39.0 MB]
Get:4 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.36/deb  kubelet 1.36.6-1.1 [13.4 MB]                                                                                                                                                                   
Fetched 76.8 MB in 8s (10.0 MB/s)                                                                                                                                                                                                                                                            
Selecting previously unselected package kubeadm.
(Reading database… 141276 files and directories currently installed.)
Preparing to unpack …/kubeadm_1.36.6-1.1_amd64.deb…
Unpacking kubeadm (1.36.6-1.1)…
Selecting previously unselected package kubectl.
Preparing to unpack …/kubectl_1.36.6-1.1_amd64.deb…
Unpacking kubectl (1.36.6-1.1)…
Selecting previously unselected package kubernetes-cni.
Preparing to unpack …/kubernetes-cni_1.9.1-1.1_amd64.deb…
Unpacking kubernetes-cni (1.9.1-1.1)…
Selecting previously unselected package kubelet.
Preparing to unpack …/kubelet_1.36.6-1.1_amd64.deb…
Unpacking kubelet (1.36.6-1.1)…
Setting up kubeadm (1.36.6-1.1)…
Setting up kubectl (1.36.6-1.1)…
Setting up kubernetes-cni (1.9.1-1.1)…
Setting up kubelet (1.36.6-1.1)…
Scanning processes...                                                                                                                                                                                                                                                                         
Scanning candidates...                                                                                                                                                                                                                                                                        
Scanning processor microcode...                                                                                                                                                                                                                                                               
Scanning linux images...                                                                                                                                                                                                                                                                      

Pending kernel upgrade!
Running kernel version:
  7.0.0-38-generic
Diagnostics:
  The currently running kernel version is not the expected kernel version 7.0.0-42-generic.

Restarting the system to load the new kernel will not be handled automatically, so you should consider rebooting.

The processor microcode seems to be up-to-date.

Restarting services...

Service restarts being deferred:
 /etc/needrestart/restart.d/dbus.service
 systemctl restart networkd-dispatcher.service
 systemctl restart systemd-logind.service
 systemctl restart unattended-upgrades.service

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
choimory@choimory-server:~$ sudo apt-mark hold kubelet kubeadm kubectl
kubelet set on hold.
kubeadm set on hold.
kubectl set on hold.
choimory@choimory-server:~$ sudo systemctl enable --now kubelet
choimory@choimory-server:~$ curl -fsSL https://get.helm.sh/helm-v4.3.1-linux-amd64.tar.gz | tar -xz -C /tmp
choimory@choimory-server:~$ sudo install /tmp/linux-amd64/helm /usr/local/bin/helm
choimory@choimory-server:~$ kubeadm version -o short; helm version --short
v1.36.6
v4.3.1+gc0d08d0
```

- kubeadm, kubelet, kubectl, helm 설치

```shell
choimory@choimory-server:~$ sudo reboot
choimory@choimory-server:~$   uname -r
7.0.0-42-generic
choimory@choimory-server:~$   swapon --show
choimory@choimory-server:~$   lsmod | grep -E 'overlay|br_netfilter'
br_netfilter           32768  0
bridge                425984  1 br_netfilter
overlay               233472  8
choimory@choimory-server:~$   sysctl net.ipv4.ip_forward net.bridge.bridge-nf-call-iptables
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
choimory@choimory-server:~$   systemctl is-active containerd docker tailscaled
active
active
active
choimory@choimory-server:~$ diff <(docker ps --format '{{.Names}}' | sort) <(cut -d: -f1 ~/docker-before.txt | sort)
```

- 서버 재부팅

## 4. 클러스터 생성

```shell
choimory@choimory-server:/data/dev/choimory/repo$ git clone https://github.com/choimory/choimory-gitops.git
Cloning into 'choimory-gitops'...
remote: Enumerating objects: 45, done.
remote: Counting objects: 100% (45/45), done.
remote: Compressing objects: 100% (34/34), done.
remote: Total 45 (delta 5), reused 40 (delta 3), pack-reused 0 (from 0)
Receiving objects: 100% (45/45), 27.38 KiB | 6.84 MiB/s, done.
Resolving deltas: 100% (5/5), done.
choimory@choimory-server:/data/dev/choimory/repo$ ls -l
total 8
drwxrwsr-x 7 choimory homeserver 4096 Nov  3 14:47 choimory-docker-compose
drwxrwsr-x 5 choimory homeserver 4096 Nov  9 23:49 choimory-gitops
choimory@choimory-server:/data/dev/choimory/repo$ 
```

- git clone

```shell
choimory@choimory-server:/data/dev/choimory/repo$ cd /data/dev/choimory/repo/choimory-gitops
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ ls bootstrap
cilium-values.yaml  kubeadm-config.yaml  root.yaml
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ cp bootstrap/kubeadm-config.yaml bootstrap/cilium-values.yaml ~/
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ TS_IP=$(tailscale ip -4)
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ TS_NAME=$(tailscale status --json | grep -m1 '"DNSName"' | sed -E 's/.*"DNSName": "([^"]+)\.".*/\1/')
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ sed -i "s/<TS_NET_NAME>/$TS_NAME/; s/<TAILSCALE_IP>/$TS_IP/" ~/kubeadm-config.yaml
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ grep -A3 certSANs ~/kubeadm-config.yaml
```

- bootstrap 실행하여 설치
- `sed` 치환
- 실제 값은 홈 디렉토리의 복사본에만 들어감
- tailscale 아이피와 도메인은 비공개

```shell
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ sudo kubeadm init --config ~/kubeadm-config.yaml
[sudo: authenticate] Password:       
[init] Using Kubernetes version: v1.36.6
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action beforehand using 'kubeadm config images pull'
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ sudo kubeadm init --config ~/kubeadm-config.yaml
[sudo: authenticate] Password:       
[init] Using Kubernetes version: v1.36.6
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action beforehand using 'kubeadm config images pull'[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [choimory-server <TS_NET_NAME> kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.10.201 <TAILSCALE_IP>]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [choimory-server localhost] and IPs [192.168.10.201 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [choimory-server localhost] and IPs [192.168.10.201 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "super-admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/instance-config.yaml"
[patches] Applied patch of type "application/strategic-merge-patch+json" to target "kubeletconfiguration"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests"
[kubelet-check] Waiting for a healthy kubelet at http://127.0.0.1:10248/healthz. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 431.654µs
[control-plane-check] Waiting for healthy control plane components. This can take up to 4m0s
[control-plane-check] Checking kube-apiserver at https://192.168.10.201:6443/livez
[control-plane-check] Checking kube-controller-manager at https://127.0.0.1:10257/healthz
[control-plane-check] Checking kube-scheduler at https://127.0.0.1:10259/livez
[control-plane-check] kube-controller-manager is healthy after 2.261038ms
[control-plane-check] kube-scheduler is healthy after 2.523725ms
[control-plane-check] kube-apiserver is healthy after 1.501256578s
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node choimory-server as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node choimory-server as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: <BOOTSTRAP_TOKEN>
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Configured RBAC rules to allow the API server kubelet client certificate to access the kubelet API
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.10.201:6443 --token <BOOTSTRAP_TOKEN> \
        --discovery-token-ca-cert-hash sha256:<CA_CERT_HASH> 
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ 
```

- 클러스터 생성
- 출력 끝의 `kubeadm join` 토큰은 비공개
- join 토큰은 따로 저장할 필요는 없음

```shell
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ mkdir -p ~/.kube
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ sudo cp /etc/kubernetes/admin.conf ~/.kube/config
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ sudo chown "$(id -u):$(id -g)" ~/.kube/config
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ chmod 600 ~/.kube/config
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl taint nodes --all node-role.kubernetes.io/control-plane-
node/choimory-server untainted
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl get nodes
NAME              STATUS     ROLES           AGE    VERSION
choimory-server   NotReady   control-plane   6m3s   v1.36.6
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl get pods -A
NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE
kube-system   coredns-589f44dc88-2s5xj                  0/1     Pending   0          5m56s
kube-system   coredns-589f44dc88-42qq7                  0/1     Pending   0          5m56s
kube-system   etcd-choimory-server                      1/1     Running   0          6m3s
kube-system   kube-apiserver-choimory-server            1/1     Running   0          6m3s
kube-system   kube-controller-manager-choimory-server   1/1     Running   0          6m3s
kube-system   kube-proxy-8dsgf                          1/1     Running   0          5m56s
kube-system   kube-scheduler-choimory-server            1/1     Running   0          6m3s
```

- kubectl 설정과 taint 제거
- 확인: Cilium 설치 전이라 노드 `NotReady`, CoreDNS `Pending`이 정상

```shell
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ helm repo add cilium https://helm.cilium.io
"cilium" has been added to your repositories
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "cilium" chart repository
Update Complete. ⎈Happy Helming!⎈
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ helm install cilium cilium/cilium --version 1.20.4 -n kube-system -f ~/cilium-values.yaml
NAME: cilium
LAST DEPLOYED: Tue Nov 10 00:09:24 2026
NAMESPACE: kube-system
STATUS: deployed
REVISION: 1
DESCRIPTION: Install complete
TEST SUITE: None
NOTES:
You have successfully installed Cilium.

Your release version is 1.20.4.

For any further help, visit https://docs.cilium.io/en/v1.20/gettinghelp
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl -n kube-system rollout status ds/cilium --timeout=5m
Waiting for daemon set "cilium" rollout to finish: 0 of 1 updated pods are available...
daemon set "cilium" successfully rolled out
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl get nodes -o wide
NAME              STATUS   ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION             CONTAINER-RUNTIME
choimory-server   Ready    control-plane   11m   v1.36.6   192.168.10.201   <none>        Ubuntu 26.04.1 LTS   7.0.0-42-generic (amd64)   containerd://2.4.2
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl get pods -A
NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE
kube-system   cilium-envoy-f852f                        1/1     Running   0          88s
kube-system   cilium-kzfj8                              1/1     Running   0          88s
kube-system   cilium-operator-6c4c574b46-pfvdj          1/1     Running   0          88s
kube-system   coredns-589f44dc88-2s5xj                  1/1     Running   0          11m
kube-system   coredns-589f44dc88-42qq7                  1/1     Running   0          11m
kube-system   etcd-choimory-server                      1/1     Running   0          11m
kube-system   kube-apiserver-choimory-server            1/1     Running   0          11m
kube-system   kube-controller-manager-choimory-server   1/1     Running   0          11m
kube-system   kube-proxy-8dsgf                          1/1     Running   0          11m
kube-system   kube-scheduler-choimory-server            1/1     Running   0          11m
```

- Cilium 설치
- 확인: 노드 `Ready`, CoreDNS `Running`

```shell
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl run dns-test --rm -it --restart=Never --image=busybox:1.37 -- nslookup kubernetes.default.svc.cluster.local
Server:         10.96.0.10
Address:        10.96.0.10:53

Name:   kubernetes.default.svc.cluster.local
Address: 10.96.0.1


All commands and output from this session will be recorded in container logs, including credentials and sensitive information passed through the command prompt.
If you don't see a command prompt, try pressing enter.
warning: couldn't attach to pod/dns-test, falling back to streaming logs: Internal error occurred: Internal error occurred: error attaching to container: failed to load task: no running task found: not found
Server:         10.96.0.10
Address:        10.96.0.10:53

Name:   kubernetes.default.svc.cluster.local
Address: 10.96.0.1


pod "dns-test" deleted from default namespace
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ kubectl run dns-test --rm -it --restart=Never --image=busybox:1.37 -- nslookup github.com
Server:         10.96.0.10
Address:        10.96.0.10:53

Non-authoritative answer:

Non-authoritative answer:
Name:   github.com
Address: 20.200.245.247

All commands and output from this session will be recorded in container logs, including credentials and sensitive information passed through the command prompt.
If you don't see a command prompt, try pressing enter.
warning: couldn't attach to pod/dns-test, falling back to streaming logs: Internal error occurred: Internal error occurred: error attaching to container: failed to load task: no running task found: not found
Server:         10.96.0.10
Address:        10.96.0.10:53

Non-authoritative answer:

Non-authoritative answer:
Name:   github.com
Address: 20.200.245.247

pod "dns-test" deleted from default namespace
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ tailscale status | head -3
<TAILSCALE_IP>     choimory-server   choimory@  linux    idle; offers exit node     
<TAILSCALE_IP>   <DEVICE_NAME>  choimory@  windows  offline, last seen 5h ago  
<TAILSCALE_IP>   <DEVICE_NAME>     choimory@  iOS      -                          
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ diff <(docker ps --format '{{.Names}}' | sort) <(cut -d: -f1 ~/docker-before.txt | sort)
```

- DNS와 기존 서비스 확인
- 확인: 두 `nslookup` 모두 주소가 나오고 `diff` 출력이 없어야 함

```shell
choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ sudo reboot
choimory@choimory-server:~$ swapon --show
choimory@choimory-server:~$ systemctl is-active containerd docker kubelet tailscaled
active
active
active
active
choimory@choimory-server:~$ kubectl get nodes
NAME              STATUS   ROLES           AGE   VERSION
choimory-server   Ready    control-plane   24m   v1.36.6
choimory@choimory-server:~$ kubectl get pods -A
NAMESPACE     NAME                                      READY   STATUS    RESTARTS      AGE
kube-system   cilium-envoy-f852f                        1/1     Running   1 (10m ago)   15m
kube-system   cilium-kzfj8                              1/1     Running   1 (10m ago)   15m
kube-system   cilium-operator-6c4c574b46-pfvdj          1/1     Running   1 (10m ago)   15m
kube-system   coredns-589f44dc88-2s5xj                  1/1     Running   1 (10m ago)   24m
kube-system   coredns-589f44dc88-42qq7                  1/1     Running   1 (10m ago)   24m
kube-system   etcd-choimory-server                      1/1     Running   1 (10m ago)   24m
kube-system   kube-apiserver-choimory-server            1/1     Running   1 (10m ago)   24m
kube-system   kube-controller-manager-choimory-server   1/1     Running   1 (10m ago)   24m
kube-system   kube-proxy-8dsgf                          1/1     Running   1 (10m ago)   24m
kube-system   kube-scheduler-choimory-server            1/1     Running   1 (10m ago)   24m
choimory@choimory-server:~$ diff <(docker ps --format '{{.Names}}' | sort) <(cut -d: -f1 ~/docker-before.txt | sort)
choimory@choimory-server:~$ 
```

- 재부팅 복구 확인
- 확인: `swapon --show`와 `diff`는 출력이 없고, 서비스 4개는 `active`, 노드는 `Ready`
- 재부팅 직후에는 노드가 `Ready`로 돌아오기까지 1~2분 걸릴 수 있음

---
