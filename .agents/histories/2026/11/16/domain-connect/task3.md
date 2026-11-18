# 목차

- [개요](#개요)
- [Envoy 프록시 이름 고정](#envoy-프록시-이름-고정)
  - [답변](#답변)
  - [변경 파일](#변경-파일)
  - [검증](#검증)
  - [push 후 확인](#push-후-확인)
  - [다음 작업](#다음-작업)
- [cloudflared 설치와 로그인 (서버)](#cloudflared-설치와-로그인-서버)
  - [답변](#답변-1)
  - [설치 결과](#설치-결과)
  - [로그인 결과](#로그인-결과)
- [Tunnel 생성 중단](#tunnel-생성-중단)
  - [답변](#답변-2)
- [Tunnel 생성](#tunnel-생성)
  - [답변](#답변-3)
  - [생성 결과](#생성-결과)
- [인증 파일 암호화 실패](#인증-파일-암호화-실패)
  - [답변](#답변-4)
  - [원인](#원인)
- [인증 파일 암호화](#인증-파일-암호화)
  - [답변](#답변-5)
  - [암호화 결과](#암호화-결과)
  - [서버 파일 정리 시점](#서버-파일-정리-시점)
  - [다음 작업 (5번)](#다음-작업-5번)
- [cloudflared gitops 작성](#cloudflared-gitops-작성)
  - [답변](#답변-6)
  - [변경 파일 (5번)](#변경-파일-5번)
  - [구성 내용](#구성-내용)
  - [검증 (5번)](#검증-5번)
  - [push 후 확인 (5번)](#push-후-확인-5번)
  - [다음 작업 (6번)](#다음-작업-6번)
- [push 후 1차 확인](#push-후-1차-확인)
  - [답변](#답변-7)
  - [확인 결과](#확인-결과)
  - [로그에서 볼 것](#로그에서-볼-것)
- [cloudflared QUIC 연결 실패](#cloudflared-quic-연결-실패)
  - [답변](#답변-8)
  - [로그 분석](#로그-분석)
  - [재시작 반복 원인](#재시작-반복-원인)
  - [조치: HTTP/2 고정](#조치-http2-고정)
- [HTTP/2 고정 작업](#http2-고정-작업)
  - [답변](#답변-9)
  - [변경 파일 (HTTP/2)](#변경-파일-http2)
  - [검증 (HTTP/2)](#검증-http2)
  - [push 후 확인 (HTTP/2)](#push-후-확인-http2)
- [HTTP/2 push 후 확인 명령](#http2-push-후-확인-명령)
  - [답변](#답변-10)
- [cloudflared Tunnel 연결 확인](#cloudflared-tunnel-연결-확인)
  - [답변](#답변-11)
  - [연결 결과](#연결-결과)
  - [다음 작업 (6번 DNS, 접속 확인, 인증 파일 삭제)](#다음-작업-6번-dns-접속-확인-인증-파일-삭제)
- [DNS 레코드 생성과 도메인 접속 확인](#dns-레코드-생성과-도메인-접속-확인)
  - [답변](#답변-12)
  - [DNS 레코드](#dns-레코드)
  - [접속 확인](#접속-확인)
  - [cf-ray LAX](#cf-ray-lax)
  - [남은 마무리 (2단계)](#남은-마무리-2단계)

---

# 개요

- plan3에서 확정한 2단계(Cloudflare Tunnel, cloudflared)를 gitops 방식으로 진행함
- 이 문서는 2단계 1번 작업(Envoy 프록시 이름을 `gateway-proxy`로 고정)부터 기록함
- task1의 Gateway 이름 변경은 push 후 `gateway` `PROGRAMMED True` 확인됨
- 참고 문서: [plan3.md](./plan3.md)(추가2 분석, 2단계 진행 순서)

---

# Envoy 프록시 이름 고정

> 응 작업 시작해줘

## 답변

- EnvoyProxy에 Service, Deployment 이름을 `gateway-proxy`로 지정
- 자동 생성 이름(`envoy-choimory-infra-gateway-<해시>`)에는 해시가 들어가 cloudflared 연결 주소가 불안정하므로 고정
- Envoy Gateway v1.10 API의 `envoyService.name`, `envoyDeployment.name` 필드 사용

## 변경 파일

| 파일 | 변경 |
|---|---|
| `infrastructure/gateway/gateway.yaml` | EnvoyProxy에 `envoyService.name: gateway-proxy`, `envoyDeployment.name: gateway-proxy` 추가. GatewayClass 단위 설정이라 Gateway가 여러 개면 이름이 겹친다는 주석 추가. "2차 작업의 cloudflared" 주석을 "cloudflared"로 정리 |
| `README.md` | 구성 표 Gateway 항목에 프록시 이름 추가, sync wave 설명에 고정 이름과 주의사항 추가 |

- 커밋과 push는 직접 진행

## 검증

- `kubectl kustomize infrastructure/gateway` 빌드 결과에서 EnvoyProxy의 `envoyDeployment.name`, `envoyService.name`이 `gateway-proxy`로 나오는 것 확인

## push 후 확인

- `main` push 후 ArgoCD가 EnvoyProxy를 갱신하면 Envoy Gateway가 프록시를 `gateway-proxy` 이름으로 새로 만들고 기존 `envoy-choimory-infra-gateway-*`를 정리함
- 서버에서 확인

```bash
kubectl -n choimory-infra get deploy,svc,pod | grep -E "gateway-proxy|envoy-choimory-infra"
kubectl -n choimory-infra get gateway gateway
kubectl get httproute -A -o custom-columns='NS:.metadata.namespace,NAME:.metadata.name,ACCEPTED:.status.parents[0].conditions[?(@.type=="Accepted")].status'
```

- 기대 결과
  - Deployment, Service `gateway-proxy` 존재, Pod Running
  - 기존 `envoy-choimory-infra-gateway-*`는 사라짐 (남아 있으면 잠시 후 재확인)
  - Gateway `gateway` `PROGRAMMED True`, HTTPRoute 4개 `ACCEPTED True`
- 클러스터 안에서 프록시 응답 확인 (선택)

```bash
kubectl -n choimory-infra run curl-test --rm -it --image=curlimages/curl --restart=Never -- \
  curl -s -o /dev/null -w "%{http_code}\n" -H "Host: staging.choimory.dev" http://gateway-proxy.choimory-infra.svc.cluster.local
```

- 200이면 front stg까지 연결됨

## 다음 작업

- 2단계 3번: Mac에서 `cloudflared` 설치, `cloudflared tunnel login`, `cloudflared tunnel create choimory-tunnel`
- 2단계 4번: 인증 파일을 SealedSecret `choimory-cloudflared-credentials`로 암호화
- 실행 위치는 Mac이 아닌 홈서버로 변경 (plan3 [cloudflared CLI 실행 위치](./plan3.md#cloudflared-cli-실행-위치))

---

# cloudflared 설치와 로그인 (서버)

> ```
> choimory@choimory-server:~$ dpkg --print-architecture
> amd64
> ```

> ```
> choimory@choimory-server:~$ 
> curl -L -o /tmp/cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
> sudo dpkg -i /tmp/cloudflared.deb && rm /tmp/cloudflared.deb
> cloudflared --version
> ...
> Unpacking cloudflared (2026.11.0)…
> Setting up cloudflared (2026.11.0)…
> Processing triggers for man-db (2.13.1-1build1)…
> cloudflared version 2026.11.0 (built 2026-11-12-12:17 UTC)
> choimory@choimory-server:~$ cloudflared tunnel login
> Please open the following URL and log in with your Cloudflare account:
>
> https://dash.cloudflare.com/argotunnel?aud=&callback=<MASKED>
>
> Leave cloudflared running to download the cert automatically.
> 2026-11-16T11:23:02Z INF Waiting for login...
> 2026-11-16T11:23:55Z INF Waiting for login...
> 2026-11-16T11:24:30Z INF You have successfully logged in.
> If you wish to copy your credentials to a server, they have been saved to:
> /home/choimory/.cloudflared/cert.pem
>
> choimory@choimory-server:~$ # 3. Tunnel 생성 → ~/.cloudflared/<UUID>.json 생성, UUID 출력
> cloudflared tunnel create choimory-tunnel
> chmod 600 ~/.cloudflared/*^C
> ```

## 답변

- 서버 아키텍처 amd64 확인 후 `.deb`로 설치, 로그인까지 정상 완료
- Tunnel 생성 명령은 `^C`로 중단

## 설치 결과

| 항목 | 값 |
|---|---|
| 아키텍처 | amd64 |
| 설치 방식 | GitHub 릴리스 `.deb` (`cloudflared-linux-amd64.deb`) |
| 버전 | `2026.11.0` (Deployment 이미지 태그 `cloudflare/cloudflared:2026.11.0`로 사용) |
| 용도 | CLI 전용. `cloudflared service install`은 실행하지 않음 |

## 로그인 결과

- `cloudflared tunnel login` 출력 URL을 Mac 브라우저에서 열어 `choimory.dev` 선택, Authorize
- `/home/choimory/.cloudflared/cert.pem` 저장
- 로그인 URL의 callback 값은 사용이 끝나 효력이 없음 (문서에는 마스킹)

---

# Tunnel 생성 중단

> 내가 취소한거임 3번 명령어 진행하면 됨?

## 답변

- 진행하면 됨. 안전하게 `cloudflared tunnel list`로 `choimory-tunnel`이 없는지 먼저 확인 후 생성
- 주석 줄과 함께 붙여 넣지 말고 한 줄씩 실행
- 생성 중 끊겨 Tunnel만 있고 인증 파일이 없다면 `cloudflared tunnel delete choimory-tunnel` 후 재생성

---

# Tunnel 생성

> ```
> choimory@choimory-server:~$ 
> cloudflared tunnel create choimory-tunnel
> chmod 600 ~/.cloudflared/*
> Tunnel credentials written to /home/choimory/.cloudflared/<TUNNEL_ID>.json. cloudflared chose this file based on where your origin certificate was found. Keep this file secret. To revoke these credentials, delete the tunnel.
>
> Created tunnel choimory-tunnel with id <TUNNEL_ID>
> ```

## 답변

- Tunnel 생성 완료

## 생성 결과

| 항목 | 값 |
|---|---|
| Tunnel 이름 | `choimory-tunnel` |
| Tunnel ID | `<TUNNEL_ID>` (마스킹. ConfigMap에는 실제 값 사용) |
| 인증 파일 | `/home/choimory/.cloudflared/<TUNNEL_ID>.json` (비밀값, 권한 600) |
| 로그인 인증서 | `/home/choimory/.cloudflared/cert.pem` (권한 600) |

- 남은 2단계 작업: 4번 인증 파일 암호화, 5번 gitops 작성, 6번 DNS 연결, 7번 확인

---

# 인증 파일 암호화 실패

> ```
> choimory@choimory-server:~$ kubectl create secret generic choimory-cloudflared-credentials \
>   -n choimory-infra \
>   --from-file=credentials.json=$HOME/.cloudflared/<TUNNEL_ID>.json \
>   --dry-run=client -o yaml \
> | kubeseal --controller-namespace choimory-infra -o yaml \
> > ~/choimory-cloudflared-credentials.yaml
>
> cat ~/choimory-cloudflared-credentials.yaml
> -bash: TUNNEL_ID: No such file or directory
> error: no secrets found. Ensure the input is valid and UTF-8 encoded
> ```

## 답변

- 문서(plan3)의 마스킹된 명령을 그대로 복사해 실행해서 실패

## 원인

- bash가 `<TUNNEL_ID>`를 "TUNNEL_ID 파일에서 입력을 읽으라"는 리다이렉션으로 해석 → 빈 입력이 `kubeseal`로 전달
- 실패하면서 빈 `~/choimory-cloudflared-credentials.yaml`이 생성됨. 다음 실행에서 덮어써서 문제 없음
- 재발 방지: 실행할 명령은 문서가 아닌 채팅에서 받은 실제 값이 들어간 명령 사용

---

# 인증 파일 암호화

> ```
> choimory@choimory-server:~$ kubectl create secret generic choimory-cloudflared-credentials \
>     -n choimory-infra \
>     --from-file=credentials.json=$HOME/.cloudflared/<TUNNEL_ID>.json \
>     --dry-run=client -o yaml \
>   | kubeseal --controller-namespace choimory-infra -o yaml \
>   > ~/choimory-cloudflared-credentials.yaml
>   cat ~/choimory-cloudflared-credentials.yaml
> ---
> apiVersion: bitnami.com/v1alpha1
> kind: SealedSecret
> metadata:
>   name: choimory-cloudflared-credentials
>   namespace: choimory-infra
> spec:
>   encryptedData:
>     credentials.json: <ENCRYPTED>
>   template:
>     metadata:
>       name: choimory-cloudflared-credentials
>       namespace: choimory-infra
> ```

- 실제 실행은 `<TUNNEL_ID>` 자리에 실제 Tunnel ID를 넣은 명령. 문서에는 Tunnel ID와 암호문 모두 마스킹

## 답변

- SealedSecret 정상 생성

## 암호화 결과

| 항목 | 확인 |
|---|---|
| kind | `SealedSecret` |
| 이름, 네임스페이스 | `choimory-cloudflared-credentials` / `choimory-infra` (gitops와 일치) |
| 암호화 데이터 키 | `credentials.json` (Deployment에서 이 이름으로 마운트) |
| 결과 파일 | 서버 `~/choimory-cloudflared-credentials.yaml` |

- 클러스터의 sealed-secrets 키로만 풀리므로 저장소 커밋 가능
- 이름, 네임스페이스에 묶여 암호화되므로 gitops에서도 같은 이름, 네임스페이스 사용

## 서버 파일 정리 시점

- `~/.cloudflared/<TUNNEL_ID>.json`(인증 파일 원본): cloudflared가 Healthy로 연결된 것을 확인한 뒤 삭제. 그 전에 문제가 생기면 재암호화가 필요할 수 있음
- `~/choimory-cloudflared-credentials.yaml`: 암호문이라 삭제, 보관 모두 무관

## 다음 작업 (5번)

| 파일 | 내용 |
|---|---|
| `infrastructure/apps/cloudflared.yaml` | Application `choimory-cloudflared` (sync wave 6) |
| `infrastructure/cloudflared/kustomization.yaml` | 네임스페이스 `choimory-infra`, 리소스 목록 |
| `infrastructure/cloudflared/config.yaml` | ConfigMap: Tunnel ID, `choimory.dev`, `*.choimory.dev` → `http://gateway-proxy.choimory-infra.svc.cluster.local:80`, 나머지 404 |
| `infrastructure/cloudflared/deployment.yaml` | `cloudflare/cloudflared:2026.11.0`, 인증 파일과 설정 마운트, `/ready` probe, 메모리 32Mi/128Mi |
| `infrastructure/cloudflared/choimory-cloudflared-credentials.yaml` | 서버에서 만든 SealedSecret |
| `README.md` | 구성 표, 디렉토리 구조, sync wave 표 |

- 1번(`gateway-proxy`)과 같이 push하면 cloudflared가 먼저 떠서 잠깐 연결 오류가 날 수 있으나 Service가 생기면 자동 해결

---

# cloudflared gitops 작성

> 응 작업 시작해줘

## 답변

- plan3의 5번 계획대로 cloudflared를 gitops로 작성. sync wave는 7

## 변경 파일 (5번)

| 파일 | 변경 |
|---|---|
| `infrastructure/apps/cloudflared.yaml` | 신규. Application `choimory-cloudflared`, sync wave 7, prune, selfHeal |
| `infrastructure/cloudflared/config.yaml` | 신규. cloudflared 설정 (Tunnel ID, 인증 파일 경로, metrics, 라우팅 규칙) |
| `infrastructure/cloudflared/kustomization.yaml` | 신규. 네임스페이스 `choimory-infra`, `configMapGenerator`로 `choimory-cloudflared-config` 생성 |
| `infrastructure/cloudflared/deployment.yaml` | 신규. Deployment `choimory-cloudflared` |
| `infrastructure/cloudflared/choimory-cloudflared-credentials.yaml` | 신규. 서버에서 만든 SealedSecret + 설명 주석 |
| `README.md` | 구성 표에 외부 접속 항목, 디렉토리 구조에 `cloudflared/`, sync wave 표에 `7 | choimory-cloudflared`, 도메인 항목 문구 수정 |

- 커밋과 push는 직접 진행

## 구성 내용

- 라우팅 규칙 (`config.yaml`)

| hostname | service |
|---|---|
| `choimory.dev` | `http://gateway-proxy.choimory-infra.svc.cluster.local:80` |
| `*.choimory.dev` | `http://gateway-proxy.choimory-infra.svc.cluster.local:80` |
| 그 외 | `http_status:404` |

- Deployment

| 항목 | 값 |
|---|---|
| 이미지 | `cloudflare/cloudflared:2026.11.0` |
| 실행 | `tunnel --config /etc/cloudflared/config/config.yaml run` |
| 마운트 | ConfigMap → `/etc/cloudflared/config`, Secret → `/etc/cloudflared/creds` (읽기 전용) |
| probe | liveness(초기 대기 10초), readiness 모두 `GET /ready :2000` |
| 자원 | 요청 cpu 10m, 메모리 32Mi / 상한 메모리 128Mi |
| 레플리카 | 1 |

- Tunnel ID는 `config.yaml`에 실제 값으로 들어감 (비밀값 아님). 문서에는 `<TUNNEL_ID>`로 마스킹
- 설정 파일 주석에 SealedSecret 재생성 방법(이 문서 "인증 파일 암호화")과 Tunnel 재생성 시 ID 변경 필요를 남김

## 검증 (5번)

- `kubectl kustomize infrastructure/cloudflared` 빌드 확인
  - ConfigMap 이름이 해시가 붙은 `choimory-cloudflared-config-<해시>`로 생성되고 Deployment 볼륨이 같은 이름을 참조
  - ConfigMap의 `tunnel` 값이 생성한 Tunnel ID와 일치
  - Secret 볼륨이 `choimory-cloudflared-credentials`를 참조, SealedSecret 이름, 네임스페이스 일치

## push 후 확인 (5번)

```bash
kubectl -n choimory-infra get application choimory-cloudflared
kubectl -n choimory-infra get pod -l app=choimory-cloudflared
kubectl -n choimory-infra logs deploy/choimory-cloudflared | grep -i "registered tunnel connection"
cloudflared tunnel info choimory-tunnel
```

- 기대 결과: Application Synced/Healthy, Pod Running(Ready 1/1), 로그에 연결 등록 메시지, `tunnel info`에 연결 표시
- 연결이 안 되면 로그에서 QUIC 실패 여부 확인 후 `config.yaml`에 `protocol: http2` 추가 검토
- 확인 후 서버의 인증 파일 원본 `~/.cloudflared/<TUNNEL_ID>.json` 삭제

## 다음 작업 (6번)

- 서버에서 DNS 레코드 생성 (1회)

```bash
cloudflared tunnel route dns choimory-tunnel choimory.dev
cloudflared tunnel route dns choimory-tunnel "*.choimory.dev"
```

- 생성 후 Cloudflare DNS → Records에서 두 CNAME이 Proxied인지 확인

---

# push 후 1차 확인

> ```
> choimory@choimory-server:~$ kubectl -n choimory-infra get deploy,svc | grep gateway-proxy
> kubectl -n choimory-infra get pod -l app=choimory-cloudflared
> kubectl -n choimory-infra logs deploy/choimory-cloudflared | grep -i "registered tunnel connection"
> cloudflared tunnel info choimory-tunnel
> deployment.apps/gateway-proxy                      1/1     1            1           85s
> service/gateway-proxy                       ClusterIP   10.110.92.127    <none>        80/TCP                                             85s
> NAME                                   READY   STATUS    RESTARTS   AGE
> choimory-cloudflared-7b7b485b8-f5sxb   0/1     Running   0          39s
> Your tunnel <TUNNEL_ID> does not have any active connection.
> ```

## 답변

- `gateway-proxy`는 정상. cloudflared는 Running이지만 Ready가 아니고 Tunnel 연결 없음

## 확인 결과

| 항목 | 상태 | 판단 |
|---|---|---|
| `gateway-proxy` Deployment, Service | 1/1, ClusterIP | 정상 (2단계 1번 반영 확인) |
| cloudflared Pod | Running, 0/1, 39초 | 연결 중이거나 실패 중 |
| Tunnel | 연결 없음 | Pod가 Tunnel에 붙지 못함 |

## 로그에서 볼 것

| 로그 내용 | 원인 | 조치 |
|---|---|---|
| `Registered tunnel connection` | 정상 (늦게 연결) | 없음 |
| `failed to dial to edge with quic`, `timeout` | UDP 7844 아웃바운드 차단 | `config.yaml`에 `protocol: http2` |
| `credentials`, `no such file`, `invalid` | 인증 파일 마운트, 내용 문제 | Secret 생성 여부 확인 |
| `Unauthorized`, `tunnel not found` | Tunnel ID와 인증 파일 불일치 | ID 대조 |
| `error parsing YAML config` | 설정 파일 형식 오류 | `config.yaml` 수정 |

---

# cloudflared QUIC 연결 실패

> ```
> choimory@choimory-server:~$   kubectl -n choimory-infra get pod -l app=choimory-cloudflared
> NAME                                   READY   STATUS    RESTARTS      AGE
> choimory-cloudflared-7b7b485b8-f5sxb   0/1     Running   4 (11s ago)   2m22s
> choimory@choimory-server:~$   kubectl -n choimory-infra logs deploy/choimory-cloudflared --tail=50
> 2026-11-16T11:43:12Z INF Starting tunnel tunnelID=<TUNNEL_ID>
> 2026-11-16T11:43:12Z INF Version 2026.11.0 (Checksum fc355cd506df8fbe45bde396d17ca7ecfee625020df768749669c49a7392899b)
> 2026-11-16T11:43:12Z INF GOOS: linux, GOVersion: go1.27.4, GoArch: amd64
> 2026-11-16T11:43:12Z INF Settings: map[config:/etc/cloudflared/config/config.yaml cred-file:/etc/cloudflared/creds/credentials.json credentials-file:/etc/cloudflared/creds/credentials.json metrics:0.0.0.0:2000]
> 2026-11-16T11:43:12Z INF Generated Connector ID: <CONNECTOR_ID>
> 2026-11-16T11:43:12Z INF Initial protocol quic
> 2026-11-16T11:43:12Z INF ICMP proxy will use 10.244.0.24 as source for IPv4
> 2026-11-16T11:43:12Z INF ICMP proxy will use fe80::dc0f:7bff:fea7:f030 in zone eth0 as source for IPv6
> 2026-11-16T11:43:12Z INF ICMP proxy will use 10.244.0.24 as source for IPv4
> 2026-11-16T11:43:12Z INF ICMP proxy will use fe80::dc0f:7bff:fea7:f030 in zone eth0 as source for IPv6
> 2026-11-16T11:43:12Z INF Starting metrics server on [::]:2000/metrics
> 2026-11-16T11:43:12Z INF Tunnel connection curve preferences: [X25519MLKEM768 CurveID(65074) CurveP256] connIndex=0 event=0 ip=198.41.192.67
> 2026-11-16T11:43:22Z INF +-----------------------------------------------------------------------------------------------+
> 2026-11-16T11:43:22Z INF |                                    CONNECTIVITY PRE-CHECKS                                    |
> 2026-11-16T11:43:22Z INF +-----------------------------------------------------------------------------------------------+
> 2026-11-16T11:43:22Z INF |  COMPONENT         TARGET                     STATUS  DETAILS                                 |
> 2026-11-16T11:43:22Z INF |  DNS Resolution    region1.v2.argotunnel.com  PASS    DNS Resolved successfully               |
> 2026-11-16T11:43:22Z INF |  DNS Resolution    region2.v2.argotunnel.com  PASS    DNS Resolved successfully               |
> 2026-11-16T11:43:22Z INF |  UDP Connectivity  region1.v2.argotunnel.com  FAIL    QUIC connection failed                  |
> 2026-11-16T11:43:22Z INF |  UDP Connectivity  region2.v2.argotunnel.com  FAIL    QUIC connection failed                  |
> 2026-11-16T11:43:22Z INF |  TCP Connectivity  region1.v2.argotunnel.com  PASS    HTTP/2 connection successful            |
> 2026-11-16T11:43:22Z INF |  TCP Connectivity  region2.v2.argotunnel.com  PASS    HTTP/2 connection successful            |
> 2026-11-16T11:43:22Z INF |  Cloudflare API    api.cloudflare.com:443     PASS    API is reachable                        |
> 2026-11-16T11:43:22Z INF |  WARNING: Allow outbound QUIC traffic on port 7844 or use HTTP2.                              |
> 2026-11-16T11:43:22Z INF |  WARNING: Allow outbound QUIC traffic on port 7844 or use HTTP2.                              |
> 2026-11-16T11:43:22Z INF |                                                                                               |
> 2026-11-16T11:43:22Z INF |  SUMMARY: Environment ready with degraded transport. cloudflared will proceed using 'http2'.  |
> 2026-11-16T11:43:22Z INF +-----------------------------------------------------------------------------------------------+
> 2026-11-16T11:43:22Z INF precheck component="DNS Resolution" details="DNS Resolved successfully" run_id=14a01a34-109a-4a88-9974-7ed252587204 status=pass target=region1.v2.argotunnel.com
> 2026-11-16T11:43:22Z INF precheck component="DNS Resolution" details="DNS Resolved successfully" run_id=14a01a34-109a-4a88-9974-7ed252587204 status=pass target=region2.v2.argotunnel.com
> 2026-11-16T11:43:22Z INF precheck component="UDP Connectivity" details="QUIC connection failed" run_id=14a01a34-109a-4a88-9974-7ed252587204 status=fail target=region1.v2.argotunnel.com
> 2026-11-16T11:43:22Z INF precheck component="UDP Connectivity" details="QUIC connection failed" run_id=14a01a34-109a-4a88-9974-7ed252587204 status=fail target=region2.v2.argotunnel.com
> 2026-11-16T11:43:22Z INF precheck component="TCP Connectivity" details="HTTP/2 connection successful" run_id=14a01a34-109a-4a88-9974-7ed252587204 status=pass target=region1.v2.argotunnel.com
> 2026-11-16T11:43:22Z INF precheck component="TCP Connectivity" details="HTTP/2 connection successful" run_id=14a01a34-109a-4a88-9974-7ed252587204 status=pass target=region2.v2.argotunnel.com
> 2026-11-16T11:43:22Z INF precheck component="Cloudflare API" details="API is reachable" run_id=14a01a34-109a-4a88-9974-7ed252587204 status=pass target=api.cloudflare.com:443
> 2026-11-16T11:43:22Z INF precheck complete hard_fail=false run_id=14a01a34-109a-4a88-9974-7ed252587204 suggested_protocol=http2
> 2026-11-16T11:43:22Z ERR Failed to dial a quic connection error="failed to dial to edge with quic: timeout: handshake did not complete in time" connIndex=0 event=0 ip=198.41.192.67
> 2026-11-16T11:43:22Z INF Retrying connection in up to 2s connIndex=0 event=0 ip=198.41.192.67
> 2026-11-16T11:43:23Z INF Tunnel connection curve preferences: [X25519MLKEM768 CurveID(65074) CurveP256] connIndex=0 event=0 ip=198.41.200.233
> ```

## 답변

- 원룸 네트워크에서 UDP 7844 아웃바운드가 막혀 cloudflared 기본 방식인 QUIC 연결이 실패

## 로그 분석

| 항목 | 결과 |
|---|---|
| 인증 파일, 설정 파일 | 정상 (`Starting tunnel`, Settings에 경로 정상) |
| DNS 조회 | PASS |
| UDP (QUIC, 7844) | FAIL (`QUIC connection failed`, `handshake did not complete in time`) |
| TCP (HTTP/2, 443) | PASS |
| Cloudflare API | PASS |
| 사전 점검 결론 | `suggested_protocol=http2` |

## 재시작 반복 원인

1. 사전 점검은 HTTP/2를 권하지만 실제 연결은 `Initial protocol quic`로 시작해 QUIC 재시도
2. 연결이 없어 `/ready` 실패
3. liveness probe 실패로 Pod 재시작 → 다시 QUIC부터 시도
- HTTP/2로 넘어가기 전에 재시작이 반복됨 (RESTARTS 4)

## 조치: HTTP/2 고정

- `infrastructure/cloudflared/config.yaml`에 추가 예정

```yaml
# 원룸 네트워크에서 UDP 7844(QUIC)가 막혀 TCP 443(HTTP/2)로 연결한다.
protocol: http2
```

- `configMapGenerator`라 push하면 ConfigMap 해시가 바뀌어 Pod 자동 재시작
- 영향: 이 설정은 Cloudflare와 서버 사이 연결 방식만 바꿈. 브라우저와 Cloudflare 사이는 그대로라 체감 차이 거의 없음
- 원룸 네트워크의 UDP 차단은 바꿀 수 없으므로 HTTP/2 고정
- liveness probe는 유지. HTTP/2는 몇 초 안에 연결되어 initialDelay 10초 안에 Ready 예상. 그래도 재시작이 반복되면 startupProbe 추가
- push 후 확인: `READY 1/1`, 로그에 `Registered tunnel connection ... protocol=http2`

---

# HTTP/2 고정 작업

> 응 작업 시작해줘

## 답변

- cloudflared 연결 방식을 HTTP/2로 고정

## 변경 파일 (HTTP/2)

| 파일 | 변경 |
|---|---|
| `infrastructure/cloudflared/config.yaml` | `protocol: http2` 추가. UDP 7844가 막힌 이유와 기본값으로 두면 재시작이 반복된다는 주석 |

- 커밋과 push는 직접 진행

## 검증 (HTTP/2)

- `kubectl kustomize infrastructure/cloudflared` 빌드에 `protocol: http2` 포함 확인
- ConfigMap 이름 해시가 바뀌고(`choimory-cloudflared-config-mg5t96hg9b` → `choimory-cloudflared-config-ht5h2bf9tk`) Deployment도 새 이름을 참조 → push 시 Pod 자동 재시작

## push 후 확인 (HTTP/2)

```bash
kubectl -n choimory-infra get pod -l app=choimory-cloudflared
kubectl -n choimory-infra logs deploy/choimory-cloudflared | grep -i "registered tunnel connection"
cloudflared tunnel info choimory-tunnel
```

- 기대 결과: `READY 1/1`, RESTARTS 증가 없음, 로그에 `Registered tunnel connection ... protocol=http2`, `tunnel info`에 연결 표시
- 확인 후 6번 DNS 레코드 생성으로 진행

---

# HTTP/2 push 후 확인 명령

> 일단 변경사항 푸시했음 확인명령어좀

## 답변

```bash
kubectl -n choimory-infra get application choimory-cloudflared
kubectl -n choimory-infra get pod -l app=choimory-cloudflared
kubectl -n choimory-infra logs deploy/choimory-cloudflared | grep -iE "protocol|registered tunnel connection"
cloudflared tunnel info choimory-tunnel
```

| 명령 | 기대 결과 |
|---|---|
| application | `Synced`, `Healthy` |
| pod | 새 Pod, `READY 1/1`, RESTARTS 0 |
| logs | `Initial protocol http2`, `Registered tunnel connection ... protocol=http2` (보통 4줄) |
| tunnel info | Connector와 연결된 엣지 위치 표시 |

- 예전 Pod가 보이면 ArgoCD가 Git을 다시 읽기 전(기본 3분 주기). 즉시 반영은 hard refresh

```bash
kubectl -n choimory-infra annotate application choimory-cloudflared argocd.argoproj.io/refresh=hard --overwrite
kubectl -n choimory-infra get pod -l app=choimory-cloudflared -w
```

---

# cloudflared Tunnel 연결 확인

> ```
> choimory@choimory-server:~$   kubectl -n choimory-infra get application choimory-cloudflared
> NAME                   SYNC STATUS   HEALTH STATUS
> choimory-cloudflared   Synced        Healthy
> choimory@choimory-server:~$   kubectl -n choimory-infra get pod -l app=choimory-cloudflared
> NAME                                    READY   STATUS    RESTARTS   AGE
> choimory-cloudflared-7fb5d945df-gxng4   1/1     Running   0          22s
> choimory@choimory-server:~$   kubectl -n choimory-infra logs deploy/choimory-cloudflared | grep -iE "protocol|registered tunnel connection"
> 2026-11-16T11:48:21Z INF Settings: map[config:/etc/cloudflared/config/config.yaml cred-file:/etc/cloudflared/creds/credentials.json credentials-file:/etc/cloudflared/creds/credentials.json metrics:0.0.0.0:2000 p:http2 protocol:http2]
> 2026-11-16T11:48:21Z INF Initial protocol http2
> 2026-11-16T11:48:22Z INF Registered tunnel connection connIndex=0 connection=<CONNECTION_ID> event=0 ip=198.41.192.7 location=icn06 protocol=http2
> 2026-11-16T11:48:22Z INF Registered tunnel connection connIndex=1 connection=<CONNECTION_ID> event=0 ip=198.41.200.13 location=icn07 protocol=http2
> 2026-11-16T11:48:23Z INF Registered tunnel connection connIndex=2 connection=<CONNECTION_ID> event=0 ip=198.41.192.57 location=icn06 protocol=http2
> 2026-11-16T11:48:24Z INF Registered tunnel connection connIndex=3 connection=<CONNECTION_ID> event=0 ip=198.41.200.43 location=icn01 protocol=http2
> 2026-11-16T11:48:31Z INF precheck complete hard_fail=false run_id=9162a908-433f-40bd-9619-ab2c976b41c6 suggested_protocol=http2
> choimory@choimory-server:~$   cloudflared tunnel info choimory-tunnel
> NAME:     choimory-tunnel
> ID:       <TUNNEL_ID>
> CREATED:  2026-11-16 11:26:01.620979 +0000 UTC
>
> CONNECTOR ID                         CREATED              ARCHITECTURE VERSION   ORIGIN IP       EDGE                      
> <CONNECTOR_ID>                       2026-11-16T11:48:22Z linux_amd64  2026.11.0 <ORIGIN_IP>     1xicn01, 2xicn06, 1xicn07 
> ```

## 답변

- cloudflared가 HTTP/2로 Tunnel에 정상 연결

## 연결 결과

| 항목 | 결과 |
|---|---|
| Application | `Synced`, `Healthy` |
| Pod | 새 Pod, `READY 1/1`, RESTARTS 0 |
| 연결 방식 | `Initial protocol http2` |
| Tunnel 연결 | 4개 등록 (connIndex 0~3), 서울 엣지 `icn01`, `icn06`, `icn07` |
| `tunnel info` | Connector 1개 (`linux_amd64`, `2026.11.0`) |

- `tunnel info`의 ORIGIN IP는 집 회선 공인 IP라 문서에 마스킹. Tunnel 사용 시 외부에 노출되지 않음
- HTTP/2 고정으로 QUIC 연결 실패 문제 해결 확인

## 다음 작업 (6번 DNS, 접속 확인, 인증 파일 삭제)

- 6번 DNS 레코드 생성 (서버)

```bash
cloudflared tunnel route dns choimory-tunnel choimory.dev
cloudflared tunnel route dns choimory-tunnel "*.choimory.dev"
```

- 정상이면 `Added CNAME choimory.dev which will route to this tunnel` 형태. 이미 레코드가 있다는 오류면 Cloudflare DNS 화면 확인
- 접속 확인 (Mac, DNS 반영에 수 초~1분)

```bash
dig +short choimory.dev
curl -sI https://choimory.dev | head -1
curl -sI https://staging.choimory.dev | head -1
curl -sI https://member-api.choimory.dev | head -1
curl -sI https://staging-member-api.choimory.dev | head -1
```

| 대상 | 정상 응답 |
|---|---|
| `dig` | Cloudflare IP |
| front (`choimory.dev`, `staging.`) | `HTTP/2 200` |
| member-api | 루트 경로라 `404`, `401` 등 앱 응답이면 정상. Cloudflare 오류(`502`, `530`)면 문제 |

- 인증 파일 원본 삭제 (서버): 연결이 확인되어 이제 삭제 가능. `rm ~/.cloudflared/<TUNNEL_ID>.json` 후 `cert.pem`만 남는지 확인

---

# DNS 레코드 생성과 도메인 접속 확인

> ```
> choimory@choimory-server:~$ 
> cloudflared tunnel route dns choimory-tunnel choimory.dev
> cloudflared tunnel route dns choimory-tunnel "*.choimory.dev"
> 2026-11-16T11:55:14Z INF Added CNAME choimory.dev which will route to this tunnel tunnelID=<TUNNEL_ID>
> 2026-11-16T11:55:16Z INF Added CNAME *.choimory.dev which will route to this tunnel tunnelID=<TUNNEL_ID>
> ```

> ```
> ╭─choimory@moryui-MacBookPro ~ 
> ╰─$   
> curl -I https://choimory.dev
> curl -I https://staging.choimory.dev
> curl -I https://staging-member-api.choimory.dev
> HTTP/2 200 
> date: Mon, 16 Nov 2026 11:56:12 GMT
> content-type: text/html; charset=utf-8
> cache-control: private, no-cache, no-store, max-age=0, must-revalidate
> link: </_next/static/media/797e433ab948586e-s.p.0r6juujl39pe6.woff2>; rel=preload; as="font"; crossorigin=""; type="font/woff2", </_next/static/media/caa3a2e1cccd8315-s.p.0wgildi0cnwt9.woff2>; rel=preload; as="font"; crossorigin=""; type="font/woff2"
> vary: rsc, next-router-state-tree, next-router-prefetch, next-router-segment-prefetch, Accept-Encoding
> x-powered-by: Next.js
> cf-cache-status: DYNAMIC
> nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
> report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=<MASKED>"}]}
> server: cloudflare
> cf-ray: a5b64f9f6b2523fc-LAX
> alt-svc: h3=":443"; ma=86400
>
> HTTP/2 200 
> date: Mon, 16 Nov 2026 11:56:16 GMT
> content-type: text/html; charset=utf-8
> cache-control: private, no-cache, no-store, max-age=0, must-revalidate
> link: </_next/static/media/797e433ab948586e-s.p.0r6juujl39pe6.woff2>; rel=preload; as="font"; crossorigin=""; type="font/woff2", </_next/static/media/caa3a2e1cccd8315-s.p.0wgildi0cnwt9.woff2>; rel=preload; as="font"; crossorigin=""; type="font/woff2"
> vary: rsc, next-router-state-tree, next-router-prefetch, next-router-segment-prefetch, Accept-Encoding
> x-powered-by: Next.js
> cf-cache-status: DYNAMIC
> nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
> report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=<MASKED>"}]}
> server: cloudflare
> cf-ray: a5b64fb81f3adbc6-LAX
> alt-svc: h3=":443"; ma=86400
>
> HTTP/2 401 
> date: Mon, 16 Nov 2026 11:56:18 GMT
> content-type: application/json;charset=UTF-8
> content-length: 71
> cache-control: no-cache, no-store, max-age=0, must-revalidate
> expires: 0
> pragma: no-cache
> vary: Origin,Access-Control-Request-Method,Access-Control-Request-Headers
> x-content-type-options: nosniff
> x-frame-options: DENY
> x-xss-protection: 0
> cf-cache-status: DYNAMIC
> nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
> report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=<MASKED>"}]}
> server: cloudflare
> cf-ray: a5b64fc5bf1a4961-LAX
> alt-svc: h3=":443"; ma=86400
> ```

## 답변

- 2단계 완료. 도메인으로 클러스터 앱 접속 확인
- 요청 경로: Cloudflare → Tunnel → cloudflared → `gateway-proxy` → HTTPRoute → 앱

## DNS 레코드

| 레코드 | 결과 |
|---|---|
| `choimory.dev` | CNAME 추가 → Tunnel |
| `*.choimory.dev` | CNAME 추가 → Tunnel |

## 접속 확인

| 도메인 | 응답 | 판단 |
|---|---|---|
| `https://choimory.dev` (front prod) | `HTTP/2 200`, `x-powered-by: Next.js` | 정상 |
| `https://staging.choimory.dev` (front stg) | `HTTP/2 200`, `x-powered-by: Next.js` | 정상 |
| `https://staging-member-api.choimory.dev` | `HTTP/2 401`, `application/json` | 정상. 인증 없이 루트를 호출해 Spring Security가 응답 (앱까지 도달) |

- `server: cloudflare`, HTTPS 인증서 오류 없음 → Universal SSL 동작 확인

## cf-ray LAX

- 응답을 처리한 Cloudflare 엣지가 로스앤젤레스(LAX)라는 뜻
- 한국에서 Cloudflare 무료 플랜은 국내 회선 사업자와의 연결 비용 문제로 해외 엣지(LAX, NRT 등)로 가는 경우가 흔함
- 접속은 정상이며 응답이 수십~100ms 더 걸릴 수 있는 정도. cloudflared 연결은 서울 엣지(`icn`)지만 사용자 요청 엣지는 사용자 회선이 결정
- 토이 프로젝트라 그대로 둠 (유료 플랜 외 변경 방법이 거의 없음)

## 남은 마무리 (2단계)

1. 서버 인증 파일 원본 삭제: `rm ~/.cloudflared/<TUNNEL_ID>.json` 후 `cert.pem`만 남는지 확인
2. prod member-api 확인 (선택): `curl -sI https://member-api.choimory.dev | head -1` → `401`이면 정상
3. 1단계 남은 설정: Universal SSL Active(접속이 되므로 이미 Active), Always Use HTTPS 켜기, AI Crawl Control Block

- 다음: 3단계 브라우저 흐름(front 화면, 회원가입, 로그인, CORS, 쿠키), 4단계 ArgoCD 도메인과 Cloudflare Access
