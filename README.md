# choimory-gitops

홈서버 k8s 클러스터의 GitOps 저장소. ArgoCD가 이 저장소의 `main` 브랜치를 보고 클러스터 구성 전체를 관리한다.

---

## 구성

| 항목 | 내용 |
|---|---|
| 클러스터 | kubeadm `v1.36.6`, 노드 1대 (컨트롤 플레인 taint 제거). Docker Compose 앱과 같은 서버에서 containerd를 공유 |
| CNI | Cilium `1.20.4` (Helm 직접 설치, ArgoCD 대상 아님). Hubble, Gateway API, Ingress 기능은 끔 |
| CD | ArgoCD (Helm 차트 `10.17.0`). 최초 설치만 Helm으로 하고, 이후에는 ArgoCD가 자기 자신을 포함해 전체를 관리 |
| CI | 각 앱 저장소의 GitHub Actions. 이미지를 GHCR에 올리고 이 저장소의 이미지 태그를 커밋 |
| 이미지 저장소 | GHCR (`ghcr.io/choimory/<앱>`, public) |
| 매니페스트 | 앱은 Kustomize(base + overlays), 공용 도구는 Helm 차트 + values |
| Gateway | Envoy Gateway (Gateway API). 공용 Gateway `gateway`를 모든 프로젝트가 같이 사용. Envoy 프록시 Deployment, Service 이름은 `gateway-proxy`로 고정 |
| 외부 접속 | Cloudflare Tunnel `choimory-tunnel`. 클러스터 안의 cloudflared가 Cloudflare로 나가는 연결을 맺고 요청을 `gateway-proxy`로 전달. 서버 인바운드 포트 없음 |
| 인프라 도구 접속 | tailnet 전용 Gateway `infra-gateway` (`*.infra.choimory.dev`). Tailscale Operator가 프록시 Service `infra-gateway-proxy`를 tailnet 기기 `choimory-infra-gateway`로 노출. 인터넷, LAN에는 열리지 않음 |
| 인증서 | 공개 앱은 Cloudflare Universal SSL. `infra-gateway`는 cert-manager가 Let's Encrypt 와일드카드 인증서를 Cloudflare DNS-01로 발급, 자동 갱신 |
| 비밀값 | Sealed Secrets. 암호화된 SealedSecret만 저장소에 커밋 |
| DB | PostgreSQL은 CloudNativePG, Redis는 공식 이미지 StatefulSet. 프로젝트·환경별로 하나씩 |
| 스토리지 | local-path-provisioner, `reclaimPolicy: Retain` |

---

## 디렉토리 구조

```
bootstrap/                      # 서버에서 직접 적용하는 최초 설치 파일
├── kubeadm-config.yaml         # kubeadm init 설정 (Tailscale 값은 placeholder)
├── cilium-values.yaml          # Cilium Helm values
└── root.yaml                   # ArgoCD root Application (infrastructure/apps를 바라봄)

infrastructure/                 # 여러 프로젝트가 같이 쓰는 공용 도구 (네임스페이스 choimory-infra)
├── apps/                       # root가 만드는 Application 목록 (sync wave로 설치 순서 지정)
├── argocd/                     # ArgoCD values
├── sealed-secrets/             # Sealed Secrets values
├── local-path-provisioner/     # provisioner와 StorageClass
├── metrics-server/             # metrics-server values
├── cloudnative-pg/             # CloudNativePG 오퍼레이터 values
├── envoy-gateway/              # Envoy Gateway values
├── gateway/                    # EnvoyProxy, GatewayClass, 공용 Gateway
├── cloudflared/                # cloudflared 설정(Tunnel 라우팅), Deployment, 인증 파일 SealedSecret
├── cert-manager/               # cert-manager values
├── tailscale-operator/         # Tailscale Operator values, OAuth SealedSecret
└── infra-gateway/              # 인프라 도구 전용 EnvoyProxy, GatewayClass, Gateway, ClusterIssuer, Certificate, Cloudflare API 토큰 SealedSecret

projects/<프로젝트명>/           # 프로젝트별 구성
├── argocd/                     # AppProject, ApplicationSet
├── namespaces/                 # Namespace, ResourceQuota, LimitRange, NetworkPolicy
├── postgres/                   # CloudNativePG Cluster (tailnet용 추가 Service 포함)
├── redis/                      # Redis StatefulSet, Service, tailnet용 Service, SealedSecret
└── <앱>/                        # Deployment, Service, HTTPRoute, SealedSecret
    ├── base/
    └── overlays/{stg,prod}/
```

- 디렉토리 `infrastructure/`와 네임스페이스 `choimory-infra`는 이름이 다르다. 디렉토리는 역할, 네임스페이스는 소유자 기준 이름
- 환경은 브랜치가 아니라 overlay로 구분한다. 이 저장소의 브랜치는 `main` 하나만 ArgoCD가 본다

---

## 공용 도구와 설치 순서

`bootstrap/root.yaml`이 `infrastructure/apps`의 Application을 만들고, 아래 sync wave 순서로 설치된다.

| wave | Application | 버전 |
|---|---|---|
| -1 | Namespace `choimory-infra` | |
| 0 | argocd | 차트 `10.17.0` |
| 1 | sealed-secrets | 차트 `2.22.0` |
| 2 | local-path-provisioner | `v0.0.37` |
| 3 | metrics-server | 차트 `3.14.0` |
| 4 | cloudnative-pg | 차트 `0.29.1` |
| 5 | envoy-gateway | `v1.10.0` |
| 5 | cert-manager | 차트 `v1.21.3` |
| 5 | tailscale-operator | 차트 `1.104.7` |
| 6 | gateway (EnvoyProxy, GatewayClass, `gateway`) | |
| 6 | infra-gateway (EnvoyProxy, GatewayClass, `infra-gateway`, ClusterIssuer, Certificate) | |
| 7 | choimory-dev (프로젝트 AppProject, ApplicationSet) | |
| 7 | choimory-cloudflared | `2026.11.0` |

- ArgoCD 기본 동작은 자식 Application의 Health를 기다리지 않으므로, `infrastructure/argocd/values.yaml`에 Application Health 판단을 추가해 wave 순서를 지킨다
- EnvoyProxy는 GatewayClass보다 먼저 만들어져야 해서 `sync-wave: "-1"`을 둔다
- Envoy 프록시 이름은 자동 생성(`envoy-<네임스페이스>-<Gateway>-<해시>`) 대신 EnvoyProxy에서 `gateway-proxy`로 고정한다. 이 설정은 GatewayClass 단위라 같은 GatewayClass로 Gateway를 두 개 이상 만들면 이름이 겹친다
- 그래서 `infra-gateway`는 GatewayClass `envoy-infra`, EnvoyProxy `envoy-proxy-infra`를 따로 두고 프록시 이름을 `infra-gateway-proxy`로 고정한다

---

## 인프라 도구 접속 (infra-gateway)

```
[공개]  브라우저 → Cloudflare → Tunnel → cloudflared → gateway-proxy → HTTPRoute → 앱
[내부]  내 기기(Tailscale) → *.infra.choimory.dev (DNS: 100.x) → infra-gateway-proxy(tailnet 기기) → HTTPRoute → 인프라 도구
```

| 항목 | 값 |
|---|---|
| 도메인 | `*.infra.choimory.dev`. Cloudflare DNS에 A 레코드 → tailnet IP `100.x` (DNS only, 회색 구름) |
| Gateway | `infra-gateway` (HTTPS 443) / GatewayClass `envoy-infra` / EnvoyProxy `envoy-proxy-infra` / 프록시 `infra-gateway-proxy` |
| tailnet 기기 | `choimory-infra-gateway` (태그 `tag:k8s`), Operator `choimory-k8s-operator` (태그 `tag:k8s-operator`) |
| 인증서 | ClusterIssuer `letsencrypt`, Certificate `infra-choimory-dev` → Secret `infra-choimory-dev-tls` |
| 허용 범위 | `infra-gateway-access: "true"` 라벨이 있는 네임스페이스의 HTTPRoute (`choimory-infra`에 적용) |

- 도메인 규칙: 서버 전체에 하나인 도구는 `{도구}.infra.choimory.dev`, 환경별 도구는 `staging-` 접두어 (공개 앱과 같은 규칙). 와일드카드 인증서가 한 단계만 덮으므로 하이픈을 쓴다
- 도구 추가는 HTTPRoute 하나로 끝난다 (`parentRefs`에 `infra-gateway`, 네임스페이스에 라벨). 인증서와 DNS는 바꾸지 않는다

| 도구 | 주소 | HTTPRoute |
|---|---|---|
| ArgoCD | `https://argocd.infra.choimory.dev` | 차트의 `server.httproute` (`infrastructure/argocd/values.yaml`). `server.insecure: true`로 TLS는 Gateway가 처리 |

- 공개 DNS에 `100.x`가 보여도 tailnet 밖에서는 닿지 않는다. tailnet IP는 프록시 Service를 지우지 않는 한 유지된다
- Tailscale 정책 파일 `tagOwners`에 `tag:k8s-operator`(소유자 `autogroup:admin`), `tag:k8s`(소유자 `tag:k8s-operator`)가 있어야 한다

---

## 프로젝트: choimory-dev

### 네임스페이스와 Application

| 네임스페이스 | 환경 | 앱 저장소 브랜치 |
|---|---|---|
| `choimory-dev-stg` | stg | `stg` |
| `choimory-dev-prod` | prod | `main` |

| ApplicationSet | Application (환경마다) | 디렉토리 | prune |
|---|---|---|---|
| `choimory-dev-apps` | `choimory-dev-{env}-front`, `choimory-dev-{env}-member-api` | `choimory-dev-front`, `choimory-dev-member-api` | 켬 |
| `choimory-dev-protected` | `choimory-dev-{env}-namespaces`, `choimory-dev-{env}-postgres`, `choimory-dev-{env}-redis` | `namespaces`, `postgres`, `redis` | 끔 |

- ApplicationSet 요소의 `component`는 Application 이름, `dir`은 디렉토리 이름. 둘을 분리해 디렉토리 이름을 바꿔도 Application 이름이 유지된다
- `choimory-dev-protected`는 `preserveResourcesOnDeletion: true`. Application이나 ApplicationSet이 삭제되어도 DB, Redis, PVC는 남는다
- 프로젝트 Application의 배포 대상은 AppProject로 `choimory-dev-*` 네임스페이스만 허용한다

### 도메인

| 앱 | stg | prod |
|---|---|---|
| front | `staging.choimory.dev` | `choimory.dev` |
| member-api | `staging-member-api.choimory.dev` | `member-api.choimory.dev` |

- API는 서비스별 서브도메인을 쓰고, 경로는 앱 경로를 그대로 전달한다 (예: `member-api.choimory.dev/v1/member/join`)
- Cloudflare Universal SSL이 한 단계 서브도메인만 덮으므로 `staging-member-api`처럼 하이픈을 쓴다
- 도메인은 HTTPRoute에 지정한다. Cloudflare Tunnel이 `choimory.dev`, `*.choimory.dev`를 모두 `gateway-proxy`로 넘기므로 도메인을 추가할 때 Cloudflare 설정은 바꾸지 않는다

### 자원

| 대상 | prod (요청 / 상한) | stg (요청 / 상한) |
|---|---|---|
| member-api | 384Mi / 768Mi | 256Mi / 512Mi |
| front | 128Mi / 256Mi | 96Mi / 192Mi |
| PostgreSQL | 128Mi / 512Mi | 64Mi / 256Mi |
| Redis | 32Mi / 128Mi (AOF 켬) | 16Mi / 64Mi (AOF 끔) |

- ResourceQuota `limits.memory`: prod 3Gi, stg 2Gi. LimitRange 기본값 64Mi / 256Mi
- CPU는 요청만 두고 상한은 두지 않는다
- NetworkPolicy로 같은 네임스페이스와 `choimory-infra`에서 오는 요청만 허용해 stg와 prod 사이 통신을 막는다

### 데이터 저장 위치

| StorageClass | 경로 |
|---|---|
| `local-path` (기본) | `/data/dev/choimory/k8s/<네임스페이스>/<PVC 이름>/` |
| `local-path-choimory-dev` | `/data/dev/choimory-dev/k8s/<네임스페이스>/<PVC 이름>/` |

---

## 배포 흐름

```
앱 저장소 push (dev, stg, main)
  → Actions test job만 자동 실행

Actions 탭 → deploy → Run workflow (브랜치 stg 또는 main)
  → test
  → 이미지 빌드, GHCR push (태그 stg-<커밋 SHA> / prod-<커밋 SHA>)
  → 이 저장소 main의 overlays/{stg,prod} 이미지 태그를 github-actions[bot]이 커밋
  → ArgoCD가 변경을 감지해 클러스터에 반영
```

- 수동 실행은 `stg`, `main` 브랜치만 허용한다. 다른 브랜치를 고르면 deploy job이 실패한다
- Actions가 이 저장소에 push할 때는 앱 저장소별 Deploy Key를 쓴다. 공개키는 이 저장소의 Deploy keys(쓰기 권한), 개인키는 앱 저장소의 Repository secret `GITOPS_DEPLOY_KEY`
- Actions가 `main`에 태그를 커밋하므로, 이 저장소를 수정하기 전에 `main`을 먼저 받아온다
- front는 `NEXT_PUBLIC_MEMBER_API_BASE_URL`이 빌드 시 번들에 고정되므로 GitHub Environments(`stg`, `prod`)의 variable로 환경별 이미지를 빌드한다

---

## 비밀값 (Sealed Secrets)

| Secret | 네임스페이스 | 용도 |
|---|---|---|
| `choimory-dev-redis-secret` | `choimory-dev-{stg,prod}` | Redis ACL 사용자 `username`, `password` (redis, member-api가 같이 사용) |
| `choimory-dev-member-api-secret` | `choimory-dev-{stg,prod}` | member-api JWT 서명 키 |
| `choimory-dev-postgres-member` | `choimory-dev-{stg,prod}` | PostgreSQL member 도메인 계정 (basic-auth `username`, `password`). CNPG `initdb`와 member-api가 같이 사용 |
| `choimory-cloudflared-credentials` | `choimory-infra` | Cloudflare Tunnel 인증 파일 |
| `cloudflare-api-token` | `choimory-infra` | cert-manager DNS-01용 Cloudflare API 토큰 (`choimory.dev`, DNS Edit + Zone Read) |
| `operator-oauth` | `choimory-infra` | Tailscale Operator OAuth 클라이언트 (Devices Core, Auth Keys, Services, 태그 `tag:k8s-operator`) |

새 SealedSecret 만들기 (서버):

```bash
kubectl create secret generic <이름> -n <네임스페이스> \
  --from-literal=<키>="$(openssl rand -hex 32)" \
  --dry-run=client -o yaml \
| kubeseal --controller-namespace choimory-infra --format yaml > <이름>.yaml
```

- SealedSecret은 이름과 네임스페이스에 묶여 암호화되므로, 둘 중 하나를 바꾸면 다시 만들어야 한다
- 컨트롤러 개인키는 저장소 밖에 백업한다. 클러스터를 다시 설치할 때 이 키를 먼저 복원해야 기존 SealedSecret이 풀린다

---

## 처음부터 설치하기

1. 서버 준비: swap 끄기, `overlay`·`br_netfilter` 모듈과 sysctl, containerd CRI 활성화(`SystemdCgroup = true`), kubeadm·kubelet·kubectl 설치와 버전 고정
2. `bootstrap/kubeadm-config.yaml`을 홈 디렉토리에 복사해 Tailscale 값을 채우고 `kubeadm init`, 컨트롤 플레인 taint 제거
3. Cilium 설치: `helm install cilium cilium/cilium --version 1.20.4 -n kube-system -f bootstrap/cilium-values.yaml`
4. `choimory-infra` 네임스페이스 생성. 기존 SealedSecret을 쓰려면 백업한 sealed-secrets 키를 이 네임스페이스에 먼저 복원
5. ArgoCD 설치: `helm install argocd argo/argo-cd --version 10.17.0 -n choimory-infra -f infrastructure/argocd/values.yaml` (릴리스 이름은 반드시 `argocd`)
6. root 등록: `kubectl apply -f bootstrap/root.yaml`. 이후 공용 도구와 프로젝트는 ArgoCD가 wave 순서로 설치

- 저장소 전체를 손으로 `kubectl apply`하지 않는다. CRD 순서와 Helm 차트를 ArgoCD가 처리한다

---

## 운영

### 관리 화면과 DB 접속

ArgoCD, PostgreSQL, Redis는 Tailscale을 켠 기기에서만 접속한다. 인터넷, 공유기 LAN에는 열려 있지 않다.

| 대상 | 접속 주소 | 방식 |
|---|---|---|
| ArgoCD | `https://argocd.infra.choimory.dev` | infra-gateway |
| PostgreSQL | `choimory-dev-postgres-{stg,prod}:5432` (DB `member`, 사용자 `choimory_dev_member`, SSL `require`) | CNPG `Cluster`의 `managed.services.additional`이 만드는 `choimory-dev-postgres-tailnet` Service를 Tailscale Operator가 노출 |
| Redis | `choimory-dev-redis-{stg,prod}:6379` (사용자 `choimory-dev`, member는 database `1`) | `choimory-dev-redis-tailnet` Service를 Tailscale Operator가 노출 |

- 접속 주소는 tailnet 기기 이름(MagicDNS)이다. 앱(member-api)은 기존 `-rw`, `choimory-dev-redis` Service를 그대로 쓴다
- 계정 규칙: PostgreSQL은 도메인별 계정(`choimory_dev_{도메인}`), Redis는 공통 계정 `choimory-dev`에 도메인별 database 번호(0: api-gateway 예약, 1: member). 비밀번호는 위 SealedSecret에 있다
- Redis는 ACL로 이름 없는 `default` 사용자를 꺼 두어 username 없이 접속할 수 없다 (`redis-cli -h <주소> --user choimory-dev --askpass`)
- tailnet 멤버는 함께 개발, 운영하는 사람만 초대한다. Tailscale 정책은 전체 허용이라 초대가 곧 ArgoCD, DB 접근 권한이다. 빠지는 사람은 Users에서 제거하고, 역할이 나뉘면 `grants`로 좁힌다
- prod DB는 클라이언트에서 Read-only로 연결한다

Tailscale 경로에 문제가 있을 때는 서버에서 port-forward로 열고, 원격에서는 SSH 터널로 접속한다.

| 대상 | port-forward (서버) |
|---|---|
| ArgoCD | `kubectl -n choimory-infra port-forward svc/argocd-server 8080:80` (`server.insecure`라 HTTP) |
| PostgreSQL | `kubectl -n choimory-dev-{env} port-forward svc/choimory-dev-postgres-rw <포트>:5432 --address 127.0.0.1` |
| Redis | `kubectl -n choimory-dev-{env} port-forward svc/choimory-dev-redis <포트>:6379 --address 127.0.0.1` |
| member-api actuator | `kubectl -n choimory-dev-{env} port-forward deploy/choimory-dev-member-api <포트>:8081` (`/actuator/health/{liveness,readiness}`) |

- 서버의 3000, 8080 등은 Docker Compose 앱이 쓰고 있을 수 있어 겹치지 않는 포트를 쓴다
- 비밀번호는 각 Secret에서 확인한다 (`kubectl get secret <이름> -o jsonpath='{.data.<키>}' | base64 -d`)

### PostgreSQL 다시 만들기

`initdb`(DB 이름, 소유 계정)는 `Cluster`를 처음 만들 때만 적용된다. 바꾸려면 `Cluster`를 지워 다시 만든다. 데이터는 사라진다.

1. 바뀐 매니페스트와 SealedSecret을 push하고 Secret이 풀렸는지 확인한다
2. `kubectl -n choimory-dev-{env} delete cluster choimory-dev-postgres`. `selfHeal`이 켜져 있어 ArgoCD가 곧바로 다시 만든다
3. 새 `Cluster`가 `Cluster in healthy state`가 될 때까지 확인한다. 예전 인스턴스가 정리되기 전에 다시 만들어지면 상태가 꼬일 수 있다
   - `Cluster is unrecoverable and needs manual intervention`: PVC가 남지 않은 것을 확인하고 `Cluster`를 한 번 더 지운다
   - 예전 `choimory-dev-postgres-1` Pod, PVC가 Terminating에서 멈춤: Pod를 `--grace-period=0 --force`로 지우고, PVC가 사라진 뒤 `Cluster`를 한 번 더 지운다
4. `kubectl -n choimory-dev-{env} rollout restart deploy/choimory-dev-member-api`

- 다른 도메인 DB를 추가할 때는 `Cluster`를 다시 만들지 않는다. 계정은 `managed.roles`(비밀번호는 basic-auth SealedSecret), DB는 CNPG `Database` 리소스로 추가한다
- 같은 도메인의 다른 앱(예: batch)은 같은 DB를 쓰고, 그 앱만 쓰는 데이터는 별도 스키마(예: `batch`)에 둔다

### 주의

- `choimory-dev-protected`의 앱(namespaces, postgres, redis)은 prune이 꺼져 있어, 매니페스트에서 리소스 이름을 바꾸거나 지우면 예전 리소스를 직접 정리해야 한다
- front, member-api Application에는 ApplicationSet 컨트롤러가 `resources-finalizer`를 붙인다. Application을 지우면 연결된 리소스도 지워지므로, 리소스를 남겨야 하면 finalizer를 먼저 제거한다
- 클러스터 인증서는 1년 만료(업그레이드 시 갱신). 업그레이드는 마이너 버전 하나씩 진행한다
- Kubernetes 버전은 Cilium의 지원 범위에 맞춰 올린다
