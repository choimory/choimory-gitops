# 목차

- [개요](#개요)
- [4단계 작업 시작 (A-1 Cloudflare API 토큰)](#4단계-작업-시작-a-1-cloudflare-api-토큰)
  - [답변](#답변)
  - [A-1. Cloudflare API 토큰 발급](#a-1-cloudflare-api-토큰-발급)
  - [A-4 (토큰 부분). 서버에서 검증과 암호화](#a-4-토큰-부분-서버에서-검증과-암호화)
  - [다음 작업 (A-2, A-3)](#다음-작업-a-2-a-3)
- [추가1](#추가1)
- [추가1 분석](#추가1-분석)
  - [답변](#답변-1)
  - [기존 토큰 (img.png)](#기존-토큰-imgpng)
  - [템플릿 선택 (img_1.png)](#템플릿-선택-img_1png)
  - [입력 값 (img_2.png)](#입력-값-img_2png)
  - [새 policy 선택 (img_3.png)](#새-policy-선택-img_3png)
  - [기존 policy 편집 (img_4.png)](#기존-policy-편집-img_4png)
  - [수정 확인 (img_5.png, img_6.png)](#수정-확인-img_5png-img_6png)
  - [Summary 확인 (img_7.png)](#summary-확인-img_7png)
  - [Create Token 후 진행](#create-token-후-진행)
  - [캡처 이메일 노출](#캡처-이메일-노출)
- [토큰 생성과 서버 명령](#토큰-생성과-서버-명령)
  - [답변](#답변-2)
  - [1. 토큰 입력과 검증](#1-토큰-입력과-검증)
  - [2. 암호화와 확인](#2-암호화와-확인)
- [명령 실행 위치](#명령-실행-위치)
  - [답변](#답변-3)
- [토큰 검증 실패 (Authorization header)](#토큰-검증-실패-authorization-header)
  - [답변](#답변-4)
  - [변수 상태 확인](#변수-상태-확인)
  - [다시 입력](#다시-입력)
  - [토큰 화면을 닫은 경우](#토큰-화면을-닫은-경우)
- [변수 이름을 토큰 값으로 바꾼 경우](#변수-이름을-토큰-값으로-바꾼-경우)
  - [답변](#답변-5)
  - [셸 기록에서 토큰 삭제](#셸-기록에서-토큰-삭제)
- [토큰 검증과 암호화 결과](#토큰-검증과-암호화-결과)
  - [답변](#답변-6)
  - [파일 확인과 gitops 반영 방법](#파일-확인과-gitops-반영-방법)
- [SealedSecret 값 노출 여부](#sealedsecret-값-노출-여부)
  - [답변](#답변-7)
  - [안전한 이유](#안전한-이유)
  - [지켜야 할 것](#지켜야-할-것)
- [cloudflare-api-token SealedSecret 작성과 A-2 안내](#cloudflare-api-token-sealedsecret-작성과-a-2-안내)
  - [답변](#답변-8)
  - [작성 파일](#작성-파일)
  - [A-2. Tailscale 태그 소유자 추가](#a-2-tailscale-태그-소유자-추가)
  - [접근 제한 정책 확인](#접근-제한-정책-확인)
- [Tailscale 기본 정책 확인](#tailscale-기본-정책-확인)
  - [답변](#답변-9)
  - [수정할 곳](#수정할-곳)
  - [그대로 둬도 되는 이유](#그대로-둬도-되는-이유)
- [tagOwners 저장 확인과 A-3 안내](#tagowners-저장-확인과-a-3-안내)
  - [답변](#답변-10)
  - [A-3. OAuth 클라이언트 발급](#a-3-oauth-클라이언트-발급)
  - [A-4 (OAuth 부분). 서버에서 확인과 암호화](#a-4-oauth-부분-서버에서-확인과-암호화)
- [OAuth Generate 버튼 비활성](#oauth-generate-버튼-비활성)
  - [답변](#답변-11)
  - [확인할 것](#확인할-것)
- [OAuth 확인과 암호화 결과](#oauth-확인과-암호화-결과)
  - [답변](#답변-12)
  - [작성 파일 (OAuth)](#작성-파일-oauth)
  - [A단계 결과](#a단계-결과)
  - [다음 작업 (B. gitops 1차)](#다음-작업-b-gitops-1차)
- [B단계 gitops 1차 작성](#b단계-gitops-1차-작성)
  - [답변](#답변-13)
  - [버전](#버전)
  - [변경 파일 (B단계)](#변경-파일-b단계)
  - [검증 (B단계)](#검증-b단계)
  - [참고 (B단계)](#참고-b단계)
  - [push 후 확인 (C단계)](#push-후-확인-c단계)
- [C단계 첫 확인 (Application 없음)](#c단계-첫-확인-application-없음)
  - [답변](#답변-14)
  - [원인 후보](#원인-후보)
  - [확인과 새로고침](#확인과-새로고침)
- [서버 kubectl alias](#서버-kubectl-alias)
  - [답변](#답변-15)
- [C단계 확인 결과](#c단계-확인-결과)
  - [답변](#답변-16)
  - [DNS 전 사전 확인 (선택)](#dns-전-사전-확인-선택)
  - [다음 작업 (D. DNS)](#다음-작업-d-dns)
- [DNS 전 사전 확인 결과](#dns-전-사전-확인-결과)
  - [답변](#답변-17)
- [DNS 레코드 화면 확인 (추가2)](#dns-레코드-화면-확인-추가2)
  - [답변](#답변-18)
  - [참고 (추가2)](#참고-추가2)
- [D단계 dig 확인과 E단계 계획](#d단계-dig-확인과-e단계-계획)
  - [답변](#답변-19)
  - [E단계 작성 계획](#e단계-작성-계획)
  - [push 후 확인과 F단계 (계획)](#push-후-확인과-f단계-계획)
- [블로그 IP 노출과 tailnet IP 고정 여부](#블로그-ip-노출과-tailnet-ip-고정-여부)
  - [답변](#답변-20)
  - [1. 블로그에 IP가 보여도 되나](#1-블로그에-ip가-보여도-되나)
  - [2. tailnet IP 고정 여부](#2-tailnet-ip-고정-여부)
- [E단계 gitops 2차 작성](#e단계-gitops-2차-작성)
  - [답변](#답변-21)
  - [변경 파일 (E단계)](#변경-파일-e단계)
  - [검증 (E단계)](#검증-e단계)
  - [push 후 확인 (E단계)](#push-후-확인-e단계)
  - [F단계 (확인과 계정 정리)](#f단계-확인과-계정-정리)
- [ArgoCD 접속 확인](#argocd-접속-확인)
  - [답변](#답변-22)
  - [남은 것 (F단계 3~5)](#남은-것-f단계-35)
  - [그다음](#그다음)

---

# 개요

- plan5에서 확정한 4단계(ArgoCD 내부 접속, 계정 정리)를 E2 방식으로 진행함
- 확정 값: 내부 도메인 `*.infra.choimory.dev`, Gateway `infra-gateway`, 인증서 Let's Encrypt(cert-manager, Cloudflare DNS-01), Tailscale Operator
- 5단계(DB)는 4단계 완료 후 끊고 따로 진행
- 이 문서는 A-1(Cloudflare API 토큰 발급)부터 기록함
- 참고 문서: [plan5.md](./plan5.md)(4단계 최종 작업계획)

---

# 4단계 작업 시작 (A-1 Cloudflare API 토큰)

> 응 작업 시작해줘

## 답변

- A-1에서 Cloudflare API 토큰을 발급하고, 서버에서 바로 SealedSecret으로 암호화(A-4의 토큰 부분)까지 진행
- 토큰 원문은 채팅, 문서에 남기지 않음

## A-1. Cloudflare API 토큰 발급

1. Cloudflare 대시보드 오른쪽 위 프로필 → My Profile → API Tokens → Create Token
2. Edit zone DNS 템플릿의 Use template 선택
3. 설정

| 항목 | 값 |
|---|---|
| Token name | `cert-manager-choimory-dev` (자유) |
| Permissions 1 | `Zone` · `DNS` · `Edit` (템플릿 기본값) |
| Permissions 2 | + Add more → `Zone` · `Zone` · `Read` |
| Zone Resources | `Include` · `Specific zone` · `choimory.dev` |
| Client IP Address Filtering | 비워 둠 |
| TTL | 비워 둠 (만료 없음. 만료되면 인증서 갱신 실패) |

4. Continue to summary → Create Token
5. 토큰 값은 이 화면에서 한 번만 보임. 채팅이나 문서에 붙이지 않고 아래 서버 명령에 바로 넣고 버림

## A-4 (토큰 부분). 서버에서 검증과 암호화

- 입력값이 화면과 셸 기록에 남지 않게 `read -rs`로 받음

```bash
# 1. 토큰 입력 (붙여 넣고 Enter, 화면에 안 보임)
read -rs CF_API_TOKEN; echo

# 2. 토큰 검증: "status": "active", success: true면 정상
curl -s https://api.cloudflare.com/client/v4/user/tokens/verify -H "Authorization: Bearer $CF_API_TOKEN"

# 3. SealedSecret으로 암호화
kubectl create secret generic cloudflare-api-token \
  -n choimory-infra \
  --from-literal=api-token="$CF_API_TOKEN" \
  --dry-run=client -o yaml \
| kubeseal --controller-namespace choimory-infra -o yaml \
> ~/cloudflare-api-token.yaml

# 4. 셸 변수 삭제, 결과 확인
unset CF_API_TOKEN
cat ~/cloudflare-api-token.yaml
```

- `cat` 결과는 암호화된 값이라 채팅에 붙여도 됨. 그 내용으로 gitops 파일 작성, 문서에는 마스킹
- `api-token`은 cert-manager가 읽을 Secret 키 이름

## 다음 작업 (A-2, A-3)

- A-2: Tailscale 정책 파일에 `tagOwners` 추가 (Tailscale 공식 quickstart 기준)

```json
"tagOwners": {
  "tag:k8s-operator": ["autogroup:admin"],
  "tag:k8s":          ["tag:k8s-operator"],
},
```

- A-3: OAuth 클라이언트 발급

| 항목 | 값 |
|---|---|
| 권한 | Devices › Core, Keys › Auth Keys, General › Services (모두 Read, Write) |
| 태그 | `tag:k8s-operator` |

- A-1 완료 후 A-2, A-3 화면 안내 진행

---

# 추가1 


## 답변

- Cloudflare 토큰 화면이 바뀌어(Policies 방식) 새 화면 기준으로 안내
- 최종 Summary 확인 완료. Create Token 진행

## 기존 토큰 (img.png)

- `Cloudflare Tunnel API Token for choimory.dev ...`는 2단계에서 서버의 `cloudflared tunnel login` 실행 시 자동 생성된 토큰
- 서버 `~/.cloudflared/cert.pem` 안에 들어 있음. 19시간 전 사용은 `cloudflared tunnel route dns`로 DNS 레코드를 만들 때
- 삭제하지 않음. 지우면 서버에서 Tunnel, DNS 관리 CLI를 쓸 수 없게 됨 (이미 떠 있는 Tunnel 연결에는 영향 없음)
- cert-manager용으로 재사용하지 않음. Tunnel 권한까지 있어 권한이 넓고, 용도별로 나눠야 문제 시 하나만 폐기 가능 → DNS 전용 토큰 새로 생성

## 템플릿 선택 (img_1.png)

- Edit zone DNS 선택

## 입력 값 (img_2.png)

| 항목 | 입력 |
|---|---|
| Token name | `choimory-dev-cert-manager` (사용자가 지정) |
| Policies | 템플릿의 Zone policy 편집 (대상 영역, Zone Read 추가) |
| Token expiration | No expiration |
| Client IP address filtering | 비워 둠 |

## 새 policy 선택 (img_3.png)

- 먼저 기존 Zone policy의 연필(편집) 아이콘으로 수정 시도. 정책 하나에 권한 두 개가 깔끔함
- 편집에서 권한 추가가 안 되면 Add new Zone policy (대상 `choimory.dev`, 권한 Zone Read)
- Account policy(계정 전체 권한), User policy(사용자 정보 권한)는 불필요

## 기존 policy 편집 (img_4.png)

- 편집 화면 하나에서 전부 설정 가능. 새 policy 불필요
- 대상 영역: `Include` · `Specific zone` · `choimory.dev` (비어 있으면 어느 영역에도 권한 없는 토큰)
- 권한 (DNS & Zones 그룹)

| 항목 | Read | Edit |
|---|---|---|
| DNS | ☐ | ☑ (Edit에 Read 포함) |
| Zone | ☑ | ☐ |
| 나머지 | ☐ | ☐ |

## 수정 확인 (img_5.png, img_6.png)

- `img_5.png`: Zone이 Edit으로 체크되어 있어 수정 필요
  - Zone Edit은 zone management 쓰기 권한(영역 설정 변경, 삭제 가능)
  - cert-manager는 영역 ID 조회만 필요해 Read로 충분
- `img_6.png`: 수정 후 확인. 전부 맞음

| 항목 | 값 | 판단 |
|---|---|---|
| Token name | `choimory-dev-cert-manager` | ✅ |
| 대상 영역 | `Include` · `Specific zone` · `choimory.dev` | ✅ |
| DNS | Edit | ✅ |
| Zone | Read ("Grants read access to zone management") | ✅ |
| 그룹 합계 | `2/12` | ✅ |

## Summary 확인 (img_7.png)

| 항목 | 값 | 판단 |
|---|---|---|
| Name | `choimory-dev-cert-manager` | ✅ |
| Expiration | No expiration | ✅ |
| 대상 영역 | `choimory.dev` | ✅ |
| 권한 | DNS: Edit, Zone: Read | ✅ |
| IP 제한 | 없음 (All IP addresses allowed) | ✅ |

## Create Token 후 진행

1. 토큰은 한 번만 표시. 채팅, 문서에 붙이지 않음
2. 서버에서 `read -rs CF_API_TOKEN`으로 입력 후 `tokens/verify`로 `"status": "active"` 확인
3. `kubectl create secret ... | kubeseal ...`로 `~/cloudflare-api-token.yaml` 생성, `unset CF_API_TOKEN`, `cat`으로 확인
   - 명령 전체는 [A-4 (토큰 부분)](#a-4-토큰-부분-서버에서-검증과-암호화) 참고
4. `curl`, `cat` 결과는 토큰 원문이 아니라 채팅에 붙여도 됨

## 캡처 이메일 노출

- `img.png`, `img_7.png`에 계정 이메일이 보임. 토큰 값은 Cloudflare가 앞뒤 일부만 보여 문제없음
- 저장소가 public이므로 커밋 전 이메일 부분을 가리거나 이미지를 제외

---

# 토큰 생성과 서버 명령

> 이제 만들면 되지?

## 답변

- Summary가 모두 맞아 Create Token 진행
- 토큰이 표시되면 바로 서버에서 아래 명령을 순서대로 실행
- 토큰 화면을 닫기 전에 검증까지 확인하면 안전 (실패 시 화면에서 다시 복사 가능)

## 1. 토큰 입력과 검증

```bash
# 토큰 입력 (붙여 넣고 Enter, 화면과 셸 기록에 남지 않음)
read -rs CF_API_TOKEN; echo

# 검증: "status": "active"면 정상
curl -s https://api.cloudflare.com/client/v4/user/tokens/verify -H "Authorization: Bearer $CF_API_TOKEN"
```

## 2. 암호화와 확인

- 검증이 `"status": "active"`일 때 실행

```bash
kubectl create secret generic cloudflare-api-token \
  -n choimory-infra \
  --from-literal=api-token="$CF_API_TOKEN" \
  --dry-run=client -o yaml \
| kubeseal --controller-namespace choimory-infra -o yaml \
> ~/cloudflare-api-token.yaml

unset CF_API_TOKEN
cat ~/cloudflare-api-token.yaml
```

- `curl`, `cat` 결과는 토큰 원문이 아니라 채팅에 붙여도 됨. 문서에는 암호화 값 마스킹

---

# 명령 실행 위치

> 서버에서해 맥에서해?

## 답변

- 서버에서 실행. `kubectl`, `kubeseal`은 홈서버에만 있음 (2단계 Tunnel 인증 파일 암호화도 서버에서 진행)
- 토큰은 Mac 브라우저에서 복사해 서버 SSH 터미널에 붙여 넣음. 클립보드만 거치고 파일로 남지 않음

1. Mac 브라우저에서 Create Token, 토큰 복사
2. Mac 터미널에서 서버로 SSH 접속
3. 서버에서 `read -rs CF_API_TOKEN; echo` 실행 후 붙여 넣고 Enter
4. 서버에서 `curl` 검증 → 암호화 → `unset` → `cat`

---

# 토큰 검증 실패 (Authorization header)

> ```
> {"success":false,"errors":[{"code":6003,"message":"Invalid request headers","error_chain":[{"code":6111,"message":"Invalid format for Authorization header"}]}],"messages":[],"result":null}choimory@choimory-server:~$
> ```

## 답변

- `CF_API_TOKEN`이 비어 있거나 공백, 줄바꿈이 섞였을 때 나는 오류
- 토큰 자체가 잘못된 경우에는 `Invalid API Token`이 나옴
- 빈 값이 암호화되지 않도록 암호화 명령은 실행하지 않음

## 변수 상태 확인

```bash
# 토큰 값은 보이지 않고 글자 수만 출력
echo ${#CF_API_TOKEN}
```

| 결과 | 의미 |
|---|---|
| `0` | 비어 있음. `read`와 `curl` 두 줄을 한꺼번에 붙여 넣으면 `read`가 다음 줄을 입력으로 가져감 |
| 40~60 정도 | 정상 길이지만 공백 등이 섞였을 수 있음 |

## 다시 입력

- 한 줄씩 따로 실행. `read -rs CF_API_TOKEN; echo` 실행 후 커서가 멈추면 토큰 붙여 넣고 Enter
- 토큰 복사 시 앞뒤 공백 주의
- 이후 `echo ${#CF_API_TOKEN}`, `curl` 검증

## 토큰 화면을 닫은 경우

- 같은 토큰은 다시 볼 수 없음
- API Tokens 목록의 `choimory-dev-cert-manager` → `...` → Roll로 같은 권한의 새 값 발급 후 다시 진행

---

# 변수 이름을 토큰 값으로 바꾼 경우

> 내가 명령에서 CF_API_TOKEN을 실제 토큰값으로 바꿨는데 바꿀필요 없는건가?

## 답변

- 바꿀 필요 없음. `CF_API_TOKEN`은 그대로 두는 변수 이름
- `read -rs CF_API_TOKEN`: 입력을 받아 `CF_API_TOKEN` 변수에 저장. 실제 토큰은 실행 후 커서가 멈췄을 때 붙여 넣음
- `$CF_API_TOKEN`: 저장된 값을 꺼내 쓰는 자리
- 실패 원인: `$` 뒤에 토큰 값을 넣어 셸이 그 값을 변수 이름으로 해석 → 없는 변수라 빈 값 → 헤더가 `Bearer `로만 전송
- 다시 진행할 때는 명령을 수정 없이 그대로 복사해 실행

## 셸 기록에서 토큰 삭제

- 명령에 토큰을 직접 넣어 서버 bash 기록에 원문이 남음

```bash
history | tail -10
# 토큰이 들어간 줄 번호 확인 후 삭제 (여러 줄이면 큰 번호부터)
history -d <번호>
history -w
```

- 본인 서버 기록이라 위험은 낮음. 찜찜하면 Roll로 새 값 발급 후 그 값으로 진행

---

# 토큰 검증과 암호화 결과

> ```
> choimory@choimory-server:~$ read -rs CF_API_TOKEN; echo
>
> choimory@choimory-server:~$ curl -s https://api.cloudflare.com/client/v4/user/tokens/verify -H "Authorization: Bearer $CF_API_TOKEN"
> {"result":{"id":"<MASKED>","status":"active"},"success":true,"errors":[],"messages":[{"code":10000,"message":"This API Token is valid and active","type":null}]}
> choimory@choimory-server:~$ kubectl create secret generic cloudflare-api-token \
>   -n choimory-infra \
>   --from-literal=api-token="$CF_API_TOKEN" \
>   --dry-run=client -o yaml \
> | kubeseal --controller-namespace choimory-infra -o yaml \
> > ~/cloudflare-api-token.yaml
> choimory@choimory-server:~$ unset CF_API_TOKEN
> cat ~/cloudflare-api-token.yaml
> ---
> apiVersion: bitnami.com/v1alpha1
> kind: SealedSecret
> metadata:
>   name: cloudflare-api-token
>   namespace: choimory-infra
> spec:
>   encryptedData:
>     api-token: <MASKED>
> ```

## 답변

- 토큰 검증 `"status":"active"` 성공, 암호화 명령도 오류 없이 완료
- 붙여 넣은 `cat` 결과가 중간에 잘려 있음. `api-token` 값은 수백 자 한 줄이고, 아래에 `template` 부분도 있어야 함

## 파일 확인과 gitops 반영 방법

```bash
# 대략 700바이트 이상, tail에 name, namespace가 보이면 정상
wc -c ~/cloudflare-api-token.yaml
tail -5 ~/cloudflare-api-token.yaml
```

| 방법 | 내용 |
|---|---|
| 채팅에 전체 붙이기 | 암호화된 값이라 노출돼도 됨. Claude가 파일 작성 |
| Mac으로 `scp` | `infrastructure/infra-gateway/`로 직접 복사. 잘림 걱정이 없어 더 확실 |

---

# SealedSecret 값 노출 여부

> ```
>   template:
>     metadata:
>       name: cloudflare-api-token
>       namespace: choimory-infra
> ```
>
> api-token 부분은 실제 토큰값 유출될까봐 내가 자른거임. 암호화라 상관없나?

## 답변

- 상관없음. 원래 public 저장소에 커밋하라고 만든 형식

## 안전한 이유

- `kubeseal`은 클러스터 안 Sealed Secrets 컨트롤러의 공개키로 암호화. 풀 수 있는 개인키는 클러스터 안에만 있음
- `choimory-infra` 네임스페이스와 `cloudflare-api-token` 이름에서만 풀리게 묶여 있어 다른 곳에 복사해 쓸 수 없음
- 같은 방식인 `choimory-cloudflared-credentials.yaml`(Tunnel 인증 파일)이 이미 public 저장소에 커밋되어 있음

## 지켜야 할 것

- 원문: `read`로 입력한 토큰, 셸 기록에 남은 토큰
- 클러스터의 Sealed Secrets 개인키: `choimory-infra`의 `sealed-secrets-key...` Secret
- 문서에는 습관대로 암호화 값도 `<MASKED>` 처리

---

# cloudflare-api-token SealedSecret 작성과 A-2 안내

> (`cat ~/cloudflare-api-token.yaml` 전체 출력. 암호화 값은 `<MASKED>`)
>
> ```
> ---
> apiVersion: bitnami.com/v1alpha1
> kind: SealedSecret
> metadata:
>   name: cloudflare-api-token
>   namespace: choimory-infra
> spec:
>   encryptedData:
>     api-token: <MASKED>
>   template:
>     metadata:
>       name: cloudflare-api-token
>       namespace: choimory-infra
> ```

## 답변

- A-1, A-4(토큰 부분) 완료. 전체 출력으로 gitops 파일 작성
- 서버의 `~/cloudflare-api-token.yaml`은 암호화 파일이라 남아도 위험 없음. 정리하려면 `rm ~/cloudflare-api-token.yaml`
- 명령에 토큰 원문을 넣었던 셸 기록 줄은 삭제 필요

## 작성 파일

| 파일 | 내용 |
|---|---|
| `infrastructure/infra-gateway/cloudflare-api-token.yaml` | SealedSecret `cloudflare-api-token` (키 `api-token`). 맨 위에 용도와 다시 만드는 방법 주석 (기존 `choimory-cloudflared-credentials.yaml`과 같은 형식) |

- 같은 폴더의 나머지 파일(Gateway, Issuer, Certificate, kustomization)은 B단계에서 작성. 1차 push 때 함께 올림

## A-2. Tailscale 태그 소유자 추가

1. Tailscale 관리 화면 → Access controls
2. 정책 파일(JSON 편집기)에서 `"tagOwners"` 확인. 있으면 안에 두 줄 추가, 없으면 최상위에 블록째 추가

```json
"tagOwners": {
  "tag:k8s-operator": ["autogroup:admin"],
  "tag:k8s":          ["tag:k8s-operator"],
},
```

| 태그 | 소유자 | 의미 |
|---|---|---|
| `tag:k8s-operator` | 관리자(본인) | Operator 기기에 붙는 태그 |
| `tag:k8s` | `tag:k8s-operator` | Operator가 만드는 프록시 기기(`choimory-infra-gateway`, 나중에 DB)에 붙는 태그 |

3. Save. 문법 오류가 있으면 저장 전에 빨간 표시

## 접근 제한 정책 확인

- 정책 파일에 `acls`나 `grants`로 접근을 제한하는지 확인
  - 기본 정책(전체 허용)이면 내 기기에서 `tag:k8s` 기기로 바로 접속
  - 제한 중이면 `tag:k8s`로 가는 허용 규칙 추가 필요

---

# Tailscale 기본 정책 확인

> (Tailscale Access controls 정책 파일 전체. 기본 정책 상태)
>
> ```jsonc
> // Example/default ACLs for unrestricted connections.
> {
>     // "tagOwners": {
>     //      "tag:example": ["autogroup:admin"],
>     // },
>     "grants": [
>         {"src": ["*"], "dst": ["*"], "ip": ["*"]},
>     ],
>     "ssh": [
>         {
>             "action": "check",
>             "src":    ["autogroup:member"],
>             "dst":    ["autogroup:self"],
>             "users":  ["autogroup:nonroot", "root"],
>         },
>     ],
>     "nodeAttrs": [
>         {
>             "target": ["autogroup:member"],
>             "attr":   ["funnel"],
>         },
>     ],
> }
> ```
>
> (주석 처리된 groups, postures, tests 등은 생략)

## 답변

- 기본 정책. `grants`가 전체 허용이라 `tagOwners`만 추가하면 됨. 접근 허용 규칙 추가 불필요

## 수정할 곳

- 주석 처리된 `tagOwners`를 주석 해제하고 내용 변경. 나머지(`grants`, `ssh`, `nodeAttrs`)는 그대로 두고 Save

```jsonc
    // Define the tags which can be applied to devices and by which users.
    "tagOwners": {
        "tag:k8s-operator": ["autogroup:admin"],
        "tag:k8s":          ["tag:k8s-operator"],
    },
```

## 그대로 둬도 되는 이유

| 항목 | 태그 기기(`tag:k8s`)에 미치는 영향 |
|---|---|
| `grants` 전체 허용 | 내 Mac → `choimory-infra-gateway` 접속 가능. 추가 규칙 불필요 |
| `ssh` (`autogroup:member` → `autogroup:self`) | 태그 기기는 사용자 소유가 아니라 해당 없음. Operator 프록시는 SSH 불필요 |
| `nodeAttrs` funnel (`autogroup:member`) | 태그 기기는 대상 아님. 내부 Gateway가 Funnel로 인터넷에 공개될 여지가 없어 오히려 좋음 |

- 나중에 기기가 늘어 "infra Gateway, DB는 내 기기에서만"처럼 제한하려면 `grants`를 좁힘. 이번 작업 범위 밖

---

# tagOwners 저장 확인과 A-3 안내

> (`tagOwners`를 추가해 저장한 정책 파일 전체)
>
> ```jsonc
>     // Define the tags which can be applied to devices and by which users.
>     "tagOwners": {
>         "tag:k8s-operator": ["autogroup:admin"],
>         "tag:k8s":          ["tag:k8s-operator"],
>     },
> ```
>
> (나머지는 기본 정책과 동일하여 생략)

## 답변

- `tagOwners` 두 줄이 정확히 들어가고 나머지도 그대로. A-2 완료
- 다음은 A-3 OAuth 클라이언트 발급과 A-4 OAuth 부분 암호화

## A-3. OAuth 클라이언트 발급

1. Tailscale 관리 화면 → Settings → Trust credentials (예전 이름 OAuth clients)
2. 새 자격 증명 만들기(+ Credential 등) → 종류 OAuth
3. 설명(Description): `choimory-k8s-operator` 등 자유
4. 권한(Scopes): 아래 세 개만 Read, Write 모두 체크

| 그룹 | 항목 | 권한 |
|---|---|---|
| Devices | Core | Read, Write |
| Keys | Auth Keys | Read, Write |
| General | Services | Read, Write |

5. Tags: `tag:k8s-operator` 추가 (Write 체크 시 태그 선택 칸이 나타남)
6. Generate → Client ID, Client secret 표시. secret은 한 번만 보임. 채팅, 문서에 붙이지 않고 서버 명령에 바로 입력

## A-4 (OAuth 부분). 서버에서 확인과 암호화

- 명령은 수정 없이 그대로, 한 줄씩 실행. `read` 줄 실행 후 커서가 멈추면 값을 붙여 넣고 Enter

```bash
# 1. Client ID 입력
read -rs TS_CLIENT_ID; echo

# 2. Client secret 입력
read -rs TS_CLIENT_SECRET; echo

# 3. 입력 확인: 두 숫자 모두 0이 아니면 정상
echo ${#TS_CLIENT_ID} ${#TS_CLIENT_SECRET}

# 4. 자격 증명 확인: 200이면 정상 (발급 토큰은 버리고 상태 코드만 출력)
curl -s -o /dev/null -w '%{http_code}\n' \
  -d client_id="$TS_CLIENT_ID" -d client_secret="$TS_CLIENT_SECRET" \
  https://api.tailscale.com/api/v2/oauth/token

# 5. 암호화 (200일 때)
kubectl create secret generic operator-oauth \
  -n choimory-infra \
  --from-literal=client_id="$TS_CLIENT_ID" \
  --from-literal=client_secret="$TS_CLIENT_SECRET" \
  --dry-run=client -o yaml \
| kubeseal --controller-namespace choimory-infra -o yaml \
> ~/operator-oauth.yaml

# 6. 변수 삭제, 결과 확인
unset TS_CLIENT_ID TS_CLIENT_SECRET
cat ~/operator-oauth.yaml
```

- Secret 이름 `operator-oauth`, 키 `client_id`, `client_secret`은 Tailscale Operator 차트가 기본으로 찾는 이름
- `cat` 결과 전체를 받아 `infrastructure/tailscale-operator/operator-oauth.yaml`로 작성

---

# OAuth Generate 버튼 비활성

> generate 버튼이 활성화가 안됨

## 답변

- 대부분 필수 항목이 빠진 경우

## 확인할 것

1. 태그 선택 (가장 흔한 원인)
   - Devices › Core, Keys › Auth Keys에 Write를 체크하면 권한마다 태그 선택이 필수
   - 생긴 Tags 칸에서 `tag:k8s-operator` 선택. 권한마다 칸이 따로 있으면 각각 모두 선택 (Services에도 있으면 선택)
2. 태그 목록에 `tag:k8s-operator`가 없으면
   - 정책 저장 전에 화면을 열었을 수 있음 → 새로고침 후 다시 설정
   - Access controls에서 정책이 실제로 저장됐는지 확인
3. 그 외
   - Description 필수인 경우 → `choimory-k8s-operator` 입력
   - Scopes 미체크, 또는 Read만 체크하고 태그 없이 둔 경우

---

# OAuth 확인과 암호화 결과

> ```
> choimory@choimory-server:~$ read -rs TS_CLIENT_ID; echo
>
> choimory@choimory-server:~$ read -rs TS_CLIENT_SECRET; echo
>
> choimory@choimory-server:~$ echo ${#TS_CLIENT_ID} ${#TS_CLIENT_SECRET}
> 17 63
> choimory@choimory-server:~$ curl -s -o /dev/null -w '%{http_code}\n' \
>   -d client_id="$TS_CLIENT_ID" -d client_secret="$TS_CLIENT_SECRET" \
>   https://api.tailscale.com/api/v2/oauth/token
> 200
> choimory@choimory-server:~$ kubectl create secret generic operator-oauth \
>   -n choimory-infra \
>   --from-literal=client_id="$TS_CLIENT_ID" \
>   --from-literal=client_secret="$TS_CLIENT_SECRET" \
>   --dry-run=client -o yaml \
> | kubeseal --controller-namespace choimory-infra -o yaml \
> > ~/operator-oauth.yaml
> choimory@choimory-server:~$ unset TS_CLIENT_ID TS_CLIENT_SECRET
> cat ~/operator-oauth.yaml
> ---
> apiVersion: bitnami.com/v1alpha1
> kind: SealedSecret
> metadata:
>   name: operator-oauth
>   namespace: choimory-infra
> spec:
>   encryptedData:
>     client_id: <MASKED>
>     client_secret: <MASKED>
>   template:
>     metadata:
>       name: operator-oauth
>       namespace: choimory-infra
> ```

## 답변

- A단계(사전 준비) 전부 완료

| 확인 | 결과 |
|---|---|
| 입력 길이 | Client ID 17, secret 63 → 정상 |
| OAuth 자격 증명 | `200` → 유효 |
| 암호화 | `operator-oauth` SealedSecret 생성 |

- 서버의 `~/operator-oauth.yaml`, `~/cloudflare-api-token.yaml`은 암호화 파일이라 남아도 됨. 정리하려면 `rm ~/operator-oauth.yaml ~/cloudflare-api-token.yaml`

## 작성 파일 (OAuth)

| 파일 | 내용 |
|---|---|
| `infrastructure/tailscale-operator/operator-oauth.yaml` | SealedSecret `operator-oauth` (키 `client_id`, `client_secret`). 맨 위에 용도, 차트 기본 이름이라는 점, 다시 만드는 방법 주석 |

## A단계 결과

| 단계 | 결과 |
|---|---|
| A-1 | Cloudflare 토큰 `choimory-dev-cert-manager` (`choimory.dev`, DNS Edit, Zone Read) |
| A-2 | Tailscale `tagOwners`에 `tag:k8s-operator`, `tag:k8s` 추가 |
| A-3 | OAuth 클라이언트 발급 (Devices Core, Auth Keys, Services, 태그 `tag:k8s-operator`) |
| A-4 | `infra-gateway/cloudflare-api-token.yaml`, `tailscale-operator/operator-oauth.yaml` |

## 다음 작업 (B. gitops 1차)

1. cert-manager, Tailscale Operator 차트의 최신 안정 버전과 values 키 확인
2. 작성
   - `infrastructure/apps/`의 `cert-manager.yaml`, `tailscale-operator.yaml`, `infra-gateway.yaml`
   - 각 values 파일
   - `infra-gateway/`의 `gateway.yaml`, `issuer.yaml`, `certificate.yaml`, `kustomization.yaml`
   - `namespace.yaml` 라벨, README
3. `kubectl kustomize` 빌드 검증 후 변경 내용 설명, 사용자 push

---

# B단계 gitops 1차 작성

> 응 B단계 작업 시작해줘

## 답변

- B단계(gitops 1차) 작성 완료, 로컬 검증 통과. 커밋, push는 사용자 진행

## 버전

| 항목 | 버전 | 근거 |
|---|---|---|
| cert-manager | 차트 `v1.21.3` | 최신 안정 버전. 공식 문서상 Kubernetes 1.33~1.36 지원 (클러스터 `v1.36.6`) |
| Tailscale Operator | 차트 `1.104.7` | 최신 버전 (2026-11-12) |

## 변경 파일 (B단계)

| 파일 | 내용 |
|---|---|
| `infrastructure/apps/cert-manager.yaml` | wave 5, `ServerSideApply` (CRD 크기) |
| `infrastructure/cert-manager/values.yaml` | CRD 설치와 보존(`crds.keep`), `startupapicheck` 끔, 작은 requests, DNS-01 재귀 DNS 옵션은 주석으로 대기 |
| `infrastructure/apps/tailscale-operator.yaml` | wave 5. 소스 3개 (차트, values 참조, SealedSecret 디렉토리) |
| `infrastructure/tailscale-operator/values.yaml` | 기기 이름 `choimory-k8s-operator`, 작은 requests, 쓰지 않는 Tailscale Ingress 끔 |
| `infrastructure/tailscale-operator/kustomization.yaml` | SealedSecret만 묶음 (values.yaml이 매니페스트로 적용되지 않게) |
| `infrastructure/apps/infra-gateway.yaml` | wave 6 |
| `infrastructure/infra-gateway/gateway.yaml` | EnvoyProxy, GatewayClass, Gateway (아래 설명) |
| `infrastructure/infra-gateway/issuer.yaml` | ClusterIssuer `letsencrypt` (prod, Cloudflare DNS-01, 대상 `choimory.dev`). public 저장소라 이메일 없음 |
| `infrastructure/infra-gateway/certificate.yaml` | `*.infra.choimory.dev` → Secret `infra-choimory-dev-tls` |
| `infrastructure/infra-gateway/kustomization.yaml` | 위 파일 4개와 SealedSecret 묶음 |
| `infrastructure/apps/namespace.yaml` | `choimory-infra`에 `infra-gateway-access: "true"` 라벨 |
| `README.md` | 구성 표, 디렉토리, sync wave 표, "인프라 도구 접속 (infra-gateway)" 섹션 신설, 비밀값 표에 3개 추가 |

- `gateway.yaml` 구성
  - EnvoyProxy `envoy-proxy-infra`: Service 이름 `infra-gateway-proxy`, 타입 `LoadBalancer`, `loadBalancerClass: tailscale`, tailnet 기기 이름 annotation `tailscale.com/hostname: choimory-infra-gateway`. wave -1로 GatewayClass보다 먼저 생성
  - GatewayClass `envoy-infra`
  - Gateway `infra-gateway`: HTTPS 443, 호스트 `*.infra.choimory.dev`, 인증서 Secret 참조, `infra-gateway-access` 라벨 네임스페이스의 HTTPRoute만 허용

## 검증 (B단계)

- `kubectl kustomize`로 `infra-gateway`(리소스 6개), `tailscale-operator`(1개) 빌드 정상
- 새 Application, values 파일 YAML 문법 확인
- 필드 이름 확인
  - Envoy Gateway v1.10 소스: `envoyService`의 `loadBalancerClass`, `annotations`, `name`
  - Tailscale 차트: 기본 Secret 이름 `operator-oauth`
  - cert-manager 차트: `crds.enabled`, `webhook.resources`, `cainjector.resources`

## 참고 (B단계)

- 빌드 결과에서 ClusterIssuer, GatewayClass(클러스터 범위)에 `namespace`가 붙어 나옴. 기존 `gateway`의 GatewayClass도 같은 상태로 정상 동작 중이라 유지
- `git status`에서 새 파일들이 이미 스테이징(`A`)된 상태. Claude가 `git add`하지 않았으므로 IDE 자동 추가로 추정. 커밋 전 확인
- `.agents/histories`(캡처 이미지의 이메일 포함)는 아직 커밋되지 않은 상태

## push 후 확인 (C단계)

```bash
# Application 상태
kubectl -n choimory-infra get application cert-manager tailscale-operator infra-gateway
kubectl -n choimory-infra get pods | grep -E 'cert-manager|operator|infra-gateway|ts-'

# 인증서 발급 (보통 1~2분 안에 READY True)
kubectl -n choimory-infra get certificate infra-choimory-dev
kubectl -n choimory-infra get challenges

# infra-gateway-proxy Service의 tailnet IP(100.x)
kubectl -n choimory-infra get svc infra-gateway-proxy
kubectl -n choimory-infra get gateway infra-gateway
```

- Tailscale 관리 화면 Machines에 `choimory-k8s-operator`, `choimory-infra-gateway` 표시 확인
- 인증서가 `False`로 오래 머물면 재시도하지 말고 `kubectl -n choimory-infra describe challenges` 확인
- 커밋 메시지 예: `feat: infra-gateway(tailnet 전용)와 cert-manager, tailscale-operator 추가`

---

# C단계 첫 확인 (Application 없음)

> ```
> Error from server (NotFound): applications.argoproj.io "cert-manager" not found
> Error from server (NotFound): applications.argoproj.io "tailscale-operator" not found
> Error from server (NotFound): applications.argoproj.io "infra-gateway" not found
> sealed-secrets-controller-5b56d98858-wbm4r          1/1     Running   0          41h
> error: the server doesn't have a resource type "certificate"
> error: the server doesn't have a resource type "challenges"
> Error from server (NotFound): services "infra-gateway-proxy" not found
> Error from server (NotFound): gateways.gateway.networking.k8s.io "infra-gateway" not found
> ```

## 답변

- 커밋 `73c283b`는 `origin/main`에 반영됨 (`release/1.0.10` 머지). push는 정상
- Application이 하나도 생기지 않음 → root Application이 새 커밋을 아직 반영하지 않은 상태

## 원인 후보

1. 아직 감지 전: ArgoCD는 기본 3분 간격으로 Git 확인. 머지 직후면 대기
2. root 동기화가 막힘: root는 wave 순서로 적용하고, 이 저장소는 Application Health를 기다리도록 설정됨. 앞 wave Application이 Healthy가 아니면 다음 wave로 못 넘어가 새 Application도 안 생김

## 확인과 새로고침

```bash
kubectl -n choimory-infra get applications
kubectl -n choimory-infra get application root \
  -o jsonpath='{.status.sync.revision}{"\n"}{.status.sync.status}{"\n"}{.status.operationState.phase}{"\n"}{.status.operationState.message}{"\n"}'

# 감지 전이면 즉시 새로고침
kubectl -n choimory-infra annotate application root argocd.argoproj.io/refresh=hard --overwrite
```

| 볼 것 | 의미 |
|---|---|
| revision이 머지 커밋인지 | 아니면 새 커밋 감지 전 |
| phase `Running`, message "waiting for healthy state of ..." | 앞 wave의 어떤 Application이 Healthy가 아니어서 막힘 |
| `get applications`에서 Healthy가 아닌 것 | 막고 있는 Application |

- 결과: 잠시 후 다시 확인했을 때 정상 생성됨 (감지 지연이었음)

---

# 서버 kubectl alias

> 기다리는동안 서버에 kubectl alias 명령어 하나 주라 kube로 할거임

## 답변

```bash
cat >> ~/.bashrc <<'EOF'

# kubectl 줄임말
alias kube='kubectl'
# kube에도 kubectl 자동완성 적용
source <(kubectl completion bash)
complete -o default -F __start_kubectl kube
EOF
source ~/.bashrc
```

- 확인: `kube get pods -n choimory-infra`
- 자동완성은 `bash-completion` 패키지 필요. 탭이 안 되면 `sudo apt install -y bash-completion` 후 재로그인
- (선택) 기본 네임스페이스 변경: `kubectl config set-context --current --namespace=choimory-infra`. 프로젝트 네임스페이스 조회 시 `-n`을 깜빡하기 쉬움

---

# C단계 확인 결과

> ```
> NAME                 SYNC STATUS   HEALTH STATUS
> cert-manager         Synced        Healthy
> tailscale-operator   Synced        Healthy
> infra-gateway        Synced        Healthy
> cert-manager-6495489497-cjjm6                       1/1     Running   0          3m3s
> cert-manager-cainjector-56ff59b55d-2wnmt            1/1     Running   0          3m3s
> cert-manager-webhook-cb9b94777-h89tn                1/1     Running   0          3m3s
> infra-gateway-proxy-78cf444596-t24cr                2/2     Running   0          63s
> operator-6c7556c5ff-dmm5c                           1/1     Running   0          3m1s
> sealed-secrets-controller-5b56d98858-wbm4r          1/1     Running   0          41h
> ts-infra-gateway-proxy-9nwld-0                      1/1     Running   0          62s
> NAME                 READY   SECRET                   AGE
> infra-choimory-dev   True    infra-choimory-dev-tls   2m31s
> No resources found in choimory-infra namespace.
> NAME                  TYPE           CLUSTER-IP     EXTERNAL-IP                                               PORT(S)         AGE
> infra-gateway-proxy   LoadBalancer   10.106.28.23   100.66.206.127,choimory-infra-gateway.<TAILNET>.ts.net   443:32761/TCP   63s
> NAME            CLASS         ADDRESS          PROGRAMMED   AGE
> infra-gateway   envoy-infra   100.66.206.127   True         2m31s
> ```

## 답변

- C단계 확인 완료. 모두 정상

| 확인 | 결과 | 판단 |
|---|---|---|
| Application 3개 | Synced, Healthy | ✅ |
| cert-manager (Pod 3개), Operator | Running | ✅ |
| Envoy 프록시 `infra-gateway-proxy` | 2/2 Running | ✅ |
| Tailscale 프록시 `ts-infra-gateway-proxy-...` | Running | ✅ |
| 인증서 `infra-choimory-dev` | READY True. Let's Encrypt 첫 발급 성공 | ✅ |
| challenges | 없음. 발급이 끝나 정리된 것 | ✅ |
| Service EXTERNAL-IP | `100.66.206.127`, `choimory-infra-gateway.<TAILNET>.ts.net` | ✅ |
| Gateway | ADDRESS `100.66.206.127`, PROGRAMMED True | ✅ |

- Tailscale 관리 화면 Machines에 `choimory-k8s-operator`, `choimory-infra-gateway` 표시

## DNS 전 사전 확인 (선택)

- Mac에서 Tailscale 켠 상태로 실행. DNS 레코드 없이 IP를 직접 지정해 인증서와 경로 확인

```bash
curl -sv --resolve argocd.infra.choimory.dev:443:100.66.206.127 https://argocd.infra.choimory.dev 2>&1 | grep -E 'subject:|issuer:|SSL certificate verify|HTTP/'
```

- `subject: CN=*.infra.choimory.dev`, issuer Let's Encrypt, `SSL certificate verify ok`면 정상
- 응답 코드는 404가 정상 (ArgoCD HTTPRoute는 E단계에서 연결)

## 다음 작업 (D. DNS)

- Cloudflare 대시보드 → `choimory.dev` → DNS → Records → Add record

| 항목 | 값 |
|---|---|
| Type | `A` |
| Name | `*.infra` |
| IPv4 address | `100.66.206.127` |
| Proxy status | 꺼짐 (DNS only, 회색 구름). 반드시 |
| TTL | Auto |

- 주황 구름(Proxied)이면 Cloudflare가 중계를 시도하지만 tailnet IP라 닿지 않아 실패
- 저장 후 Mac에서 `dig +short test.infra.choimory.dev` → `100.66.206.127`이면 정상. 더 구체적인 레코드라 기존 `*.choimory.dev` Tunnel 레코드보다 우선
- DNS 확인 후 E단계(ArgoCD values 2차 작성)

---

# DNS 전 사전 확인 결과

> ```
> ╭─choimory@moryui-MacBookPro ~
> ╰─$ curl -sv --resolve argocd.infra.choimory.dev:443:100.66.206.127 https://argocd.infra.choimory.dev 2>&1 | grep -E 'subject:|issuer:|SSL certificate verify|HTTP/'
>
> *  subject: CN=*.infra.choimory.dev
> *  issuer: C=US; O=Let's Encrypt; CN=YR1
> *  SSL certificate verify ok.
> * using HTTP/2
> > GET / HTTP/2
> < HTTP/2 404
> ```

## 답변

- Mac에서 Tailscale로 `infra-gateway`까지 HTTPS가 끝까지 연결됨

| 항목 | 결과 | 판단 |
|---|---|---|
| 인증서 대상 | `CN=*.infra.choimory.dev` | ✅ |
| 발급자 | Let's Encrypt (`YR1`, Let's Encrypt의 현행 중간 인증서) | ✅ |
| 검증 | `SSL certificate verify ok` | ✅ |
| 프로토콜 | HTTP/2 | ✅ |
| 응답 | `404` | ✅ 예상대로. ArgoCD HTTPRoute가 아직 없음 |

---

# DNS 레코드 화면 확인 (추가2)

> 추가 2 확인 (Cloudflare DNS Records 화면 캡처. 캡처 파일은 정리됨)

## 답변

- D단계 레코드가 맞게 들어감

| 레코드 | Type | Content | Proxy status | 판단 |
|---|---|---|---|---|
| `*.infra.choimory.dev` | A | `100.66.206.127` | DNS only - reserved IP | ✅ |
| `*.choimory.dev` | Tunnel | `choimory-tunnel` | Proxied | 기존 그대로 |
| `choimory.dev` | Tunnel | `choimory-tunnel` | Proxied | 기존 그대로 |

- "reserved IP": `100.66.x.x`가 CGNAT 예약 대역(`100.64.0.0/10`)이라 Cloudflare가 프록시할 수 없는 주소로 인식한 표시. DNS only가 맞으므로 정상
- 화면의 Recommendations(MX, SPF/DKIM/DMARC)는 메일 관련 안내라 이번 작업과 무관

## 참고 (추가2)

- 메일 위조 방지 (선택, 나중에): `choimory.dev`로 메일을 보낼 일이 없으면 SPF(`v=spf1 -all`), DMARC(`p=reject`) TXT 레코드를 넣어 도메인 이름 위조를 어렵게 할 수 있음
- 이전 캡처 참조: `img_1.png`~`img_7.png`가 디렉토리에서 정리되어 "추가1", "추가1 분석"의 이미지 링크는 원본과 다름 (이메일 노출 정리 목적)

---

# D단계 dig 확인과 E단계 계획

> ```
> ╭─choimory@moryui-MacBookPro ~
> ╰─$ dig +short test.infra.choimory.dev
> dig +short argocd.infra.choimory.dev
>
> 100.66.206.127
> 100.66.206.127
> ```

## 답변

- D단계(DNS) 완료. 두 이름 모두 `100.66.206.127`로 조회되어 와일드카드 레코드 정상 동작

## E단계 작성 계획

- ArgoCD 차트 `10.17.0`의 values 키 사전 확인

| 변경 | 값 | 이유 |
|---|---|---|
| `global.domain` | `argocd.infra.choimory.dev` | 차트가 이 값으로 `configs.cm.url`을 만듦. 지금은 차트 기본값 `argocd.example.com` |
| `configs.params."server.insecure"` | `true` | TLS는 `infra-gateway`가 처리, argocd-server는 HTTP로 받음. 없으면 무한 리다이렉트 |
| `server.httproute` | `enabled: true`, `parentRefs: infra-gateway`(`sectionName: https`), `hostnames: argocd.infra.choimory.dev` | 차트가 HTTPRoute 생성. 규칙은 기본값(`/` 전체) |
| `README.md` | "관리 화면과 DB 접속" 표의 ArgoCD 줄 | port-forward 대신 `https://argocd.infra.choimory.dev`(Tailscale 필요) |

- `choimory-infra`에 이미 `infra-gateway-access` 라벨이 있어 HTTPRoute가 바로 붙음

## push 후 확인과 F단계 (계획)

1. 서버: `kube -n choimory-infra get httproute`, `kube -n choimory-infra rollout status deploy/argocd-server`
2. Mac(Tailscale 켬): `https://argocd.infra.choimory.dev` → 자물쇠, 로그인 화면
3. Tailscale 끄고 접속 안 되는지 확인
4. 초기 비밀번호 로그인 → 비밀번호 변경 → `argocd-initial-admin-secret` 삭제

- 주의: push 후 argocd-server 재시작으로 ArgoCD 화면이 잠깐 끊길 수 있음 (동기화, 배포 영향 없음)
- 주의: port-forward(`svc/argocd-server 8080:443`)는 `server.insecure` 적용 후 포트가 `80`으로 바뀜. 이후는 도메인으로 접속

---

# 블로그 IP 노출과 tailnet IP 고정 여부

> 질문두개
> 1. 사진에 보이는 *.infra의 ip 보여도 돼? 블로그 올릴건데
> 2. add record의 ip는 고정임? tailnet ip라 고정인가?

## 답변

- 1: 보여도 괜찮음
- 2: 사실상 고정이지만 기기가 새로 만들어지면 바뀜

## 1. 블로그에 IP가 보여도 되나

- 이미 공개된 정보: `*.infra.choimory.dev` A 레코드는 공개 DNS라 누구나 `dig`로 조회 가능. 캡처를 가려도 숨겨지지 않음
- 밖에서 접속 불가: `100.66.206.127`은 CGNAT 예약 대역의 tailnet 주소라 인터넷에서 라우팅되지 않음. tailnet에 로그인한 기기만 닿음
- 드러나는 것은 "Tailscale을 쓴다"는 정도. 집 공인 IP, 서버 위치는 알 수 없음

| 블로그에서 가리는 게 좋은 것 | 이유 |
|---|---|
| tailnet 이름 (`tail....ts.net`) | 계정 식별자 성격. 접속은 안 되지만 노출할 필요 없음 |
| 계정 이메일 | Cloudflare 캡처들에 있었음 |
| 토큰, OAuth, Tunnel ID | 원문은 물론 앞뒤 일부도 가림 |
| SealedSecret 암호화 값 | 노출돼도 안전하지만 의미 없는 긴 문자열이라 줄이는 편이 깔끔 |

## 2. tailnet IP 고정 여부

- Tailscale IP는 기기 단위로 한 번 배정되고 기기가 tailnet에 남아 있는 한 유지 (DHCP처럼 주기적으로 바뀌지 않음)
- `choimory-infra-gateway`는 Operator가 만든 프록시. 기기 정보(키)가 클러스터 Secret(`ts-infra-gateway-proxy-...`)에 저장되어 Pod 재시작, 서버 재부팅, Operator 업그레이드로는 바뀌지 않음
- 태그 기기라 키 만료도 기본으로 꺼짐

| 바뀌는 경우 | 예 |
|---|---|
| 프록시 Service가 지워졌다 다시 생김 | `infra-gateway` Application 삭제, EnvoyProxy나 Service 이름 변경, `loadBalancerClass` 변경 |
| 상태 Secret 삭제 | 수동 삭제, 네임스페이스 정리 |
| 클러스터 재설치 | kubeadm 재구축 |
| 관리 화면에서 기기 삭제 | Machines에서 Remove |

- 바뀌면 Cloudflare의 `*.infra` A 레코드 IP 하나만 수정. 인증서, Gateway, HTTPRoute는 그대로
- Tailscale 관리 화면 → Machines → `choimory-infra-gateway` → Edit IPv4 address로 주소 직접 지정 가능. 재설치 후 새 기기에도 같은 주소를 지정하면 DNS 수정 불필요 (지정하는 순간 IP가 바뀌므로 DNS도 함께 수정). 지금은 불필요
- MagicDNS 이름(`choimory-infra-gateway.<TAILNET>.ts.net`)을 CNAME으로 거는 방법은 불가. tailnet 안 DNS에서만 풀리고 공개 DNS에서는 조회되지 않음

---

# E단계 gitops 2차 작성

> 시작

## 답변

- E단계(gitops 2차) 작성 완료. 커밋, push는 사용자 진행

## 변경 파일 (E단계)

| 파일 | 변경 |
|---|---|
| `infrastructure/argocd/values.yaml` | `global.domain: argocd.infra.choimory.dev`, `server.httproute`(`enabled: true`, `parentRefs: infra-gateway`/`sectionName: https`, `hostnames: argocd.infra.choimory.dev`, 규칙은 기본값), `configs.params."server.insecure": true` |
| `README.md` | 인프라 도구 접속 섹션에 도구 표(ArgoCD 주소, HTTPRoute 생성 위치) 추가. 운영 → 관리 화면은 ArgoCD를 도메인 접속으로 변경, port-forward는 비상용 `8080:80`(HTTP)으로 수정 |

## 검증 (E단계)

- 차트 `10.17.0` 템플릿 확인
  - `argocd-cm`의 `url`은 `https://<global.domain>`으로 생성
  - HTTPRoute backend 포트는 `server.insecure`가 `true`면 HTTP 포트(80)
- values YAML 파싱 결과 세 값이 의도대로 들어감
- 로컬에 helm이 없어 차트 전체 렌더링은 못 함. push 후 서버에서 확인

## push 후 확인 (E단계)

```bash
kube -n choimory-infra get application argocd
kube -n choimory-infra get httproute
kube -n choimory-infra rollout status deploy/argocd-server
# 상태 상세: Accepted True, ResolvedRefs True
kube -n choimory-infra describe httproute argocd-server
```

- push 직후 argocd-server 재시작으로 ArgoCD 화면이 잠깐 끊길 수 있음
- 커밋 메시지 예: `feat: ArgoCD를 infra-gateway(argocd.infra.choimory.dev)로 연결`

## F단계 (확인과 계정 정리)

1. Mac에서 Tailscale 켜고 `https://argocd.infra.choimory.dev` 접속 → 자물쇠, 로그인 화면
2. Tailscale 끄고 다시 접속 → 연결 안 되면(타임아웃) 정상
3. 초기 비밀번호 확인 (서버)

```bash
kube -n choimory-infra get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d; echo
```

4. `admin`으로 로그인 → User Info → Update Password로 변경
5. 새 비밀번호로 다시 로그인되는지 확인 후 초기 Secret 삭제

```bash
kube -n choimory-infra delete secret argocd-initial-admin-secret
```

- 초기 Secret은 ArgoCD 설치 때 자동 생성된 것이라 Git에 없어 지워도 다시 생기지 않음

---

# ArgoCD 접속 확인

> argocd tailscale키고 접속 확인했어, 끄면 접속 안되는것도 확인함

## 답변

- E단계 push와 F단계 1, 2번 확인 완료. ArgoCD가 tailnet 안에서만 열림

| 확인 | 결과 |
|---|---|
| Tailscale 켜고 `https://argocd.infra.choimory.dev` | 접속됨 ✅ |
| Tailscale 끄고 접속 | 안 됨 ✅ |

## 남은 것 (F단계 3~5)

```bash
# 1. 초기 비밀번호 확인 (서버)
kube -n choimory-infra get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d; echo
```

2. 브라우저에서 `admin` 로그인 → User Info → Update Password로 변경
3. 로그아웃 후 새 비밀번호로 재로그인 확인
4. 초기 Secret 삭제 (서버)

```bash
kube -n choimory-infra delete secret argocd-initial-admin-secret
```

- 비밀번호는 채팅, 문서에 남기지 않고 비밀번호 관리자에 저장

## 그다음

- 계정 정리까지 끝나면 4단계 완료
- 계획대로 여기서 한 번 끊고, 5단계(Tailscale Operator로 DB, Redis 노출)는 따로 진행. Operator가 이미 설치되어 DB Service 쪽 설정만 하면 됨
