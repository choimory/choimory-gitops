# 목차

- [개요](#개요)
- [작업 마무리](#작업-마무리)
  - [답변](#답변)
  - [단계별 결과](#단계별-결과)
  - [최종 구성](#최종-구성)
  - [주요 결정](#주요-결정)
  - [진행 중 문제와 해결](#진행-중-문제와-해결)
  - [README 반영](#readme-반영)
  - [남은 일 (이번 범위 밖)](#남은-일-이번-범위-밖)
  - [문서](#문서)

---

# 개요

- k8s-setup으로 만든 클러스터에 도메인을 연결하고, 인프라 도구와 DB를 Tailscale 전용으로 여는 작업을 마무리함
- 공개 앱은 Cloudflare Tunnel, 인프라 도구는 Tailscale 전용 Gateway, DB와 Redis는 Tailscale Operator로 노출
- 마지막으로 PostgreSQL, Redis 계정을 도메인 기준으로 다시 만듦
- 기간: 2026-11-16 ~ 2026-11-18

---

# 작업 마무리

> 작업 마무리

## 답변

- 0~5단계와 계정 재생성까지 모두 완료하고 stg, prod에서 확인함
- calendar를 `DONE:도메인 연동 작업(domain-connect)`으로 변경하고 README에 PostgreSQL 재생성 절차를 반영함

## 단계별 결과

| 단계 | 작업 | 결과 | 문서 |
|---|---|---|---|
| 0 | 공용 Gateway 이름 `shared-gateway` → `gateway` | 완료 | plan1, task1 |
| 1 | `choimory.dev` 구매(가비아), 네임서버를 Cloudflare로 | Active | plan2, task2 |
| 2 | Cloudflare Tunnel `choimory-tunnel`, cloudflared(gitops 관리), Envoy 프록시 이름 `gateway-proxy` 고정 | 앱 도메인 4개 외부 접속 | plan3, task3 |
| 3 | 브라우저 확인 (CORS, 쿠키) | 인프라 문제 없음. 메인·헤더 로그인 표시는 front 앱 문제 | plan4 |
| 4 | 인프라 도구 Tailscale 전용 (E2): `infra-gateway`, `*.infra.choimory.dev`, cert-manager + Let's Encrypt(DNS-01), Tailscale Operator, ArgoCD 연결 | ArgoCD가 tailnet에서만 열림 | plan4, plan5, task4 |
| 5 | PostgreSQL, Redis를 tailnet 기기 4개로 노출 (CNPG `managed.services.additional`, Redis 전용 Service), Redis 매니페스트 리소스별 분리 | Mac에서 stg, prod 4곳 접속 | plan6, task5, plan7 |
| 추가 | 계정 재생성: PostgreSQL `choimory_dev_member`, Redis ACL 사용자 `choimory-dev` | 앱 회원가입·로그인, DataGrip 확인 | plan7, plan8, task6 |

## 최종 구성

```
[공개 앱]   브라우저 → Cloudflare(Universal SSL) → Tunnel → cloudflared → gateway-proxy → HTTPRoute → front, member-api
[인프라]    내 기기(Tailscale) → *.infra.choimory.dev(DNS: 100.x) → infra-gateway-proxy(Let's Encrypt) → HTTPRoute → ArgoCD
[DB]       내 기기(Tailscale) → choimory-dev-{postgres,redis}-{stg,prod} (tailnet 기기) → PostgreSQL, Redis
```

| 대상 | 계정 | 비밀값 |
|---|---|---|
| PostgreSQL | 도메인별 `choimory_dev_{도메인}`. member는 DB `member`, 스키마 `public` | SealedSecret `choimory-dev-postgres-member` (basic-auth). CNPG `initdb`와 member-api가 같이 사용 |
| Redis | 공통 `choimory-dev`, `default` 사용자 끔. database 0은 api-gateway 예약, member는 1 | SealedSecret `choimory-dev-redis-secret` (`username`, `password`) |

- member-api는 앱 코드 수정 없이 gitops env만 변경 (`redis-username`은 앱에 이미 있었고, database는 `SPRING_DATA_REDIS_DATABASE`)

## 주요 결정

| 항목 | 결정 | 이유 |
|---|---|---|
| 외부 접속 | Cloudflare Tunnel | 원룸 공유기 아래라 포트포워딩 불가. 인바운드 포트 없이 공개 |
| 공개 인증서 | Cloudflare Universal SSL | 트래픽이 Cloudflare를 거침 |
| Tunnel 관리 | gitops (인증 파일 SealedSecret) | 저장소 전체를 gitops로 일관되게 |
| Tunnel 프로토콜 | HTTP/2 (TCP 443) 고정 | 원룸 네트워크에서 UDP 7844(QUIC) 차단 |
| 인프라 도구 접근 | Tailscale 전용 Gateway + 내 도메인 (E2) | 인터넷 노출 없음. 도구 추가는 HTTPRoute 하나 |
| 인프라 인증서 | Let's Encrypt 와일드카드 (cert-manager, DNS-01) | Cloudflare를 거치지 않아 직접 발급 필요 |
| tailnet 접근 제한 | 전체 허용 유지 | tailnet 멤버 = 함께 개발, 운영하는 사람 |
| DB 계정 | PostgreSQL 도메인별, Redis 공통 + database 번호 | 도메인 단위로 데이터 소유 구분 |
| 스키마 | 업무 데이터는 `public`, 앱 전용 데이터(예: batch 메타데이터)만 별도 스키마 | 스키마는 데이터 주인 기준으로 나눔 |

## 진행 중 문제와 해결

| 문제 | 원인 | 해결 |
|---|---|---|
| cloudflared Pod 재시작 반복 | UDP 7844(QUIC) 차단 | `protocol: http2` 고정 |
| Cluster 재생성 후 prod `unrecoverable` | selfHeal이 예전 인스턴스 정리 전에 다시 만듦 | PVC 정리 확인 후 Cluster 한 번 더 삭제 |
| stg 예전 PostgreSQL Pod, PVC가 Terminating에서 멈춤 | CNPG Pod의 긴 종료 대기 | Pod 강제 삭제 후 Cluster 한 번 더 삭제 |
| DataGrip Redis 접속 실패 | 계정명 입력 오류 (ACL이라 username 필수) | 계정명 수정 |

## README 반영

- 이번 작업 중 각 단계에서 구성, 디렉토리 구조, 인프라 도구 접속, 비밀값, DB 접속 표를 반영함
- 마무리로 "운영 > PostgreSQL 다시 만들기" 추가: `initdb` 변경 시 재생성 절차, 상태가 꼬였을 때 대처, 도메인 DB 추가 방법(`managed.roles`, `Database`), 같은 도메인 앱의 스키마 분리

## 남은 일 (이번 범위 밖)

| 항목 | 내용 |
|---|---|
| sealed-secrets 키 백업 | 서버 `kubernetes-keys/`에만 있음. 서버 밖(비밀번호 관리자 등)에 보관 |
| 서버 홈 정리 | SealedSecret 결과 파일 `choimory-dev-*-{stg,prod}.yaml` 4개 삭제 |
| front | 메인, 헤더의 로그인 상태 표시 (공통 레이아웃에서 silent refresh) |
| member-batch | 만들 때 같은 `member` DB, 메타데이터는 `batch` 스키마, Flyway 이력 분리 |
| 히스토리 커밋 | `.agents/histories/2026/11/16/` 커밋 (마스킹 완료) |

## 문서

- plan1~8, task1~6, 노션 "6차 작업 QNA"
