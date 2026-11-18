# 목차

- [개요](#개요)
- [1단계 도메인과 Cloudflare DNS 진행](#1단계-도메인과-cloudflare-dns-진행)
  - [답변](#답변)
  - [1-1. Cloudflare에 사이트 추가](#1-1-cloudflare에-사이트-추가)
  - [1-2. 가비아에서 네임서버 변경](#1-2-가비아에서-네임서버-변경)
  - [1-3. Cloudflare Active 확인](#1-3-cloudflare-active-확인)
  - [1-4. SSL/TLS 확인](#1-4-ssltls-확인)
  - [완료 후 확인](#완료-후-확인)
- [추가1](#추가1)
- [Connect your domain 화면](#connect-your-domain-화면)
  - [답변](#답변-1)
  - [봇 설정 변경 가능 여부](#봇-설정-변경-가능-여부)
  - [전부 Block 후 나중에 열기](#전부-block-후-나중에-열기)
- [Review your DNS records 화면](#review-your-dns-records-화면)
  - [답변](#답변-2)
  - [화면에서 할 일](#화면에서-할-일)
- [네임서버 변경 화면](#네임서버-변경-화면)
  - [답변](#답변-3)
  - [DNSSEC 확인](#dnssec-확인)
- [네임서버 전파 대기와 Active](#네임서버-전파-대기와-active)
  - [답변](#답변-4)
  - [Active 확인](#active-확인)
  - [남은 1단계 설정](#남은-1단계-설정)
- [1단계 결과와 다음 단계](#1단계-결과와-다음-단계)
  - [답변](#답변-5)
  - [연결 상태](#연결-상태)
  - [2단계 흐름](#2단계-흐름)

---

# 개요

- 가비아에서 `choimory.dev` 구매 완료
- plan2 작업 순서의 1단계(도메인과 Cloudflare DNS)를 진행함
- 1단계는 Cloudflare와 가비아 웹 화면에서 직접 진행하는 작업이며 gitops 변경은 없음
- 참고 문서: [plan2.md](./plan2.md)(1. 도메인과 Cloudflare DNS)

---

# 1단계 도메인과 Cloudflare DNS 진행

> 도메인 구매 완료했어 1단계 진행하자

## 답변

- Cloudflare 사이트 추가 → 가비아 네임서버 변경 → Active 확인 → SSL/TLS 확인 순서로 진행
- 완료 후 `dig`로 네임서버 변경 확인

## 1-1. Cloudflare에 사이트 추가

1. https://dash.cloudflare.com 로그인 (계정이 없으면 가입)
2. Account Home → Onboard a domain(또는 Add a domain)
3. `choimory.dev` 입력. DNS 레코드 가져오기는 Quick scan 기본값, AI crawler 등 옵션도 기본값
4. 플랜은 Free 선택
5. DNS 레코드 확인 화면에서 가비아 파킹, 기본 A 레코드가 가져와졌으면 모두 삭제
   - 지금은 레코드가 없어도 됨. Tunnel 레코드는 2단계에서 생성
6. 마지막 화면의 네임서버 2개(`xxx.ns.cloudflare.com`, `yyy.ns.cloudflare.com` 형태) 확인

## 1-2. 가비아에서 네임서버 변경

1. My가비아 → 서비스 관리 → 도메인 → `choimory.dev` 관리
2. DNSSEC가 켜져 있으면 먼저 끄기 (보통 꺼져 있음)
3. 네임서버 설정에서 가비아 기본 네임서버를 지우고 1차, 2차에 Cloudflare 네임서버 입력 후 적용
   - 3, 4차는 비움
   - 소유자 인증(휴대폰, 이메일)을 요구할 수 있음

## 1-3. Cloudflare Active 확인

1. Cloudflare 화면에서 Check nameservers now
2. 보통 수 분~1시간, 길면 24시간 안에 Active로 변경. Active 시 메일 수신

## 1-4. SSL/TLS 확인

| 위치 | 설정 | 비고 |
|---|---|---|
| SSL/TLS → Edge Certificates | Universal SSL이 Active인지 확인 | Active 후 보통 15분 안에 발급 |
| SSL/TLS → Edge Certificates | Always Use HTTPS 켜기 | `.dev`는 HTTPS만 쓰지만 켜 둠 |
| SSL/TLS → Overview | 암호화 모드는 기본값 유지 | Tunnel 연결에는 적용되지 않음 |

## 완료 후 확인

- `dig NS choimory.dev`로 Cloudflare 네임서버 확인
- 확인 후 2단계(Tunnel 생성, cloudflared 배포) 진행

---


# Connect your domain 화면

> task2 추가1 확인해봐

## 답변

- Cloudflare 사이트 추가 첫 화면(`img.png`). 기본값으로 진행해도 구성에 문제 없음

| 항목 | 의미 | 추천 |
|---|---|---|
| Domain name | 추가할 도메인 | `choimory.dev` |
| I monetize pages that serve ads | 광고 수익 사이트 표시 | 체크 안 함 |
| Search | 검색엔진 봇 허용 여부 | Allow면 검색 노출 |
| Training | AI 학습 크롤러 허용 여부 | 취향 |
| Agents, Personal Agents | 사람 대신 접속하는 AI 에이전트 | 서비스 이용에 영향 없음 |
| Enable Bot Preference Sync | 위 설정을 `robots.txt` 앞에 자동으로 붙임 | 켜 둬도 됨 |
| Import DNS records | 기존 DNS 레코드 가져오기 | Automatic 유지 |

- Cloudflare를 거치는 HTTP 요청에만 적용. 브라우저 사용자, Tunnel 동작과 무관
- ArgoCD 등 인프라 도구는 Cloudflare Access로 보호하므로 봇 설정과 무관

## 봇 설정 변경 가능 여부

> 나중에 설정을 바꿀수도 있어?

- 전부 변경 가능
- 봇 설정은 AI Crawl Control(또는 Security → Bots), DNS는 DNS → Records, 플랜은 Overview/Billing
- 메뉴 위치가 자주 바뀌므로 대시보드 검색창에 "AI Crawl", "robots"로 검색
- 변경 즉시 적용되며 사이트, Tunnel 동작에 영향 없음

## 전부 Block 후 나중에 열기

> 그럼 난 다 block하고 나중에 자리잡으면 열어도 되나

- 가능. 준비 단계에서는 자연스러운 선택

| 항목 | 영향 |
|---|---|
| Search | 검색에 안 나옴. 완성 전 화면, staging이 노출되지 않는 장점 |
| Training | AI 학습 크롤러 차단 |
| Agents | AI가 사용자 대신 사이트를 열 때 차단 |
| 일반 사용자, 앱 내부 호출 | 영향 없음 |

- 나중에 Allow로 바꾸면 즉시 적용. 불이익은 없지만 재수집까지 며칠~몇 주. 빨리 노출하려면 Google Search Console 등록
- 설정은 도메인 전체(모든 서브도메인)에 적용. prod만 열고 staging은 막으려면 WAF 규칙이나 staging의 `robots.txt`/`noindex` 필요
- 카카오톡, 슬랙 링크 미리보기가 안 나오면 이 설정부터 확인
- 결정: 전부 Block으로 두고 서비스가 자리 잡으면 Search부터 열기

---

# Review your DNS records 화면

> 추가1 다시 확인

## 답변

- DNS 레코드 확인 화면(`img_1.png`). 레코드 0개가 정상

| 항목 | 의미 | 대응 |
|---|---|---|
| Records we found: 0 | 가져온 레코드 없음 | 정상. 새 도메인이라 지울 파킹 레코드도 없음 |
| MX 레코드 안내 | `@choimory.dev` 메일 수신에 필요 | 지금 불필요. 필요하면 Cloudflare Email Routing(무료) |
| `www` 레코드 안내 | `www.choimory.dev` 사용에 필요 | 지금 불필요. HTTPRoute에 없어 접속 시 404. 원하면 나중에 Redirect Rules로 `choimory.dev`로 리다이렉트 |
| 루트 도메인 레코드 안내 | `choimory.dev` 접속에 필요 | 2단계에서 Tunnel이 생성 |
| DNS Setup: Full | 네임서버를 통째로 Cloudflare로 옮기는 방식 | 계획과 일치. Tunnel에 필요 |
| Proxied / DNS only | 주황 구름은 Cloudflare 경유, 회색은 DNS만 | Tunnel 레코드는 자동으로 Proxied |

- 2단계에서 `choimory.dev`, `*.choimory.dev` → Tunnel CNAME이 생김 (와일드카드는 직접 추가할 수 있음)
- 지금 임의로 A 레코드를 만들면 나중에 Tunnel 레코드와 충돌하므로 비워 둠

## 화면에서 할 일

> 그럼 저 화면에서 나는 뭐하면됨?

1. 레코드 목록이 비어 있는지 확인
2. 파란 안내 3개(MX, www, 루트 도메인)는 무시하거나 닫기
3. Add record, Import는 누르지 않음
4. Continue to activation

---

# 네임서버 변경 화면

> 추가1 다시 확인

## 답변

- Update your nameservers to activate Cloudflare 화면(`img_2.png`)

| 항목 | 내용 |
|---|---|
| 넣을 네임서버 | `jule.ns.cloudflare.com`, `micah.ns.cloudflare.com` |
| 지울 네임서버 | `ns.gabia.co.kr`, `ns.gabia.net`, `ns1.gabia.co.kr` |
| DNSSEC 끄기 | 켜져 있으면 끄기 |
| Only allow Cloudflare IP addresses at your origin | 해당 없음. Tunnel은 서버로 들어오는 연결이 없어 열어 둔 포트가 없음 |

- 변경 전 `.dev` 등록 기관 조회 결과: 가비아 네임서버 3개가 등록되어 있었음
- 할 일: 가비아에서 기존 3개 삭제 → 1차 `jule`, 2차 `micah` 입력 → 적용 → Cloudflare에서 I updated my nameservers

## DNSSEC 확인

> dnssec은 어케 끔

- 이미 꺼져 있어 할 일 없음
- `.dev` 등록 기관에 DS 레코드가 없음을 확인 (DNSSEC가 켜져 있으면 DS 레코드가 있어야 함)

```bash
dig DS choimory.dev @ns-tld1.charlestonroadregistry.com +norec +noall +answer +authority
# 응답에 DS 없이 dev. SOA만 옴
```

- 가비아 도메인 관리 화면의 DNSSEC 항목이 "미사용"이면 그대로 둠
- Cloudflare Active 후 Cloudflare에서 DNSSEC를 켤 수 있음 (DNS → Settings → DNSSEC). 필수는 아님

---

# 네임서버 전파 대기와 Active

> 추가1 확인

## 답변

- Waiting for your registrar to propagate your new nameservers 화면(`img_3.png`)
- 가비아 네임서버 변경은 이미 완료. Cloudflare가 주기적으로 확인하는 단계라 할 일 없음

```bash
dig NS choimory.dev @ns-tld1.charlestonroadregistry.com +norec +noall +authority
# choimory.dev.  10800  IN  NS  jule.ns.cloudflare.com.
# choimory.dev.  10800  IN  NS  micah.ns.cloudflare.com.
```

## Active 확인

> 추가1 확인

- Your domain is now protected by Cloudflare 화면(`img_4.png`). Active 완료
- 공개 DNS 확인

```bash
dig +short NS choimory.dev @1.1.1.1
# jule.ns.cloudflare.com.
# micah.ns.cloudflare.com.
```

- `img_4.png` 왼쪽 위에 계정 이메일 일부가 보임. 저장소가 public이므로 커밋 전 마스킹 또는 제외 필요

## 남은 1단계 설정

| 순서 | 메뉴 | 할 일 |
|---|---|---|
| 1 | SSL/TLS → Edge Certificates | Universal SSL 인증서 Active 확인 (Pending이면 보통 15분 안에 Active) |
| 2 | SSL/TLS → Edge Certificates | Always Use HTTPS 켜기 |
| 3 | SSL/TLS → Overview | 암호화 모드 기본값 유지 (Tunnel에는 적용되지 않음) |
| 4 | AI Crawl Control | Search, Training, Agents를 Block으로 (이미 Block이면 확인만) |

- DNS 레코드가 없어 지금 `https://choimory.dev` 접속은 오류가 정상

---

# 1단계 결과와 다음 단계

> 그럼 도메인 연동은 끝난거?

## 답변

- 끝난 것은 도메인과 Cloudflare의 연결까지. 클러스터 앱 접속은 2단계부터

## 연결 상태

```
브라우저 → choimory.dev → Cloudflare (완료) ──(끊김)── 클러스터 Gateway → 앱
```

| 구간 | 상태 |
|---|---|
| 도메인 구매 (가비아) | 완료 |
| 네임서버를 Cloudflare로 변경, Active | 완료 |
| HTTPS 인증서 (Universal SSL) | 자동 발급 (상태 확인 남음) |
| Cloudflare → 클러스터 연결 (Tunnel) | 2단계 |
| DNS 레코드 (`choimory.dev`, `*.choimory.dev` → Tunnel) | 2단계 |
| 앱 도메인 접속 확인 | 3단계 |

## 2단계 흐름

> 그러니까 도메인사고 cloudflare에 입히는건 끝났고 이제 서버에서 cloudflared 작업해야하는거지

- 맞음. 다만 cloudflared는 서버에 직접 설치하지 않고 클러스터 안에 Pod로 배포. 서버에서 직접 할 일은 토큰 암호화(kubeseal)뿐

| 순서 | 위치 | 할 일 |
|---|---|---|
| 1 | Cloudflare 대시보드 (Zero Trust → Networks → Tunnels) | Tunnel `choimory-tunnel` 생성, 토큰 발급 |
| 2 | 서버 | 토큰을 `kubeseal`로 암호화해 SealedSecret 생성 (원문 저장 안 함) |
| 3 | gitops | `choimory-cloudflared` Application, Deployment, SealedSecret 추가. 확인 후 Envoy Service 고정 이름 지정 |
| 4 | push | ArgoCD가 cloudflared Pod 배포 → 대시보드 Tunnel 상태 Healthy |
| 5 | Cloudflare 대시보드 | Public Hostname에 `choimory.dev`, `*.choimory.dev`를 Envoy 프록시 Service로 연결. 와일드카드 DNS 레코드 확인 |
| 6 | 브라우저 | 앱 도메인 접속 확인 (3단계) |

- 시작 전 결정: Tunnel 관리 방식 (대시보드 관리 추천)
