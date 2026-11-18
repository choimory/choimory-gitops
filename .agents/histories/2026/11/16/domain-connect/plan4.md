# 목차

- [개요](#개요)
- [3단계 소개](#3단계-소개)
  - [답변](#답변)
  - [브라우저로 따로 확인하는 이유](#브라우저로-따로-확인하는-이유)
  - [요청 흐름 두 가지](#요청-흐름-두-가지)
  - [확인 항목 (stg)](#확인-항목-stg)
  - [prod](#prod)
  - [문제 시 고칠 곳](#문제-시-고칠-곳)
  - [진행 방식](#진행-방식)
- [추가1](#추가1)
- [추가1 분석](#추가1-분석)
  - [답변](#답변-1)
  - [유력한 원인: 쿠키가 API 도메인에만 저장됨](#유력한-원인-쿠키가-api-도메인에만-저장됨)
  - [로컬에서 됐던 이유](#로컬에서-됐던-이유)
  - [확인 방법](#확인-방법)
  - [원인별 수정 방향](#원인별-수정-방향)
  - [prod 테스트 계정](#prod-테스트-계정)
- [확인2](#확인2)
- [확인2 분석](#확인2-분석)
  - [답변](#답변-2)
  - [캡처의 refresh token 노출](#캡처의-refresh-token-노출)
  - [캡처에서 확인된 것](#캡처에서-확인된-것)
  - [원인 추정 (수정)](#원인-추정-수정)
  - [확인할 것 (Network 탭)](#확인할-것-network-탭)
  - [수정 방향 (front)](#수정-방향-front)
- [프론트 문제 여부](#프론트-문제-여부)
  - [답변](#답변-3)
  - [인프라가 아니라고 보는 근거](#인프라가-아니라고-보는-근거)
  - [확정 방법](#확정-방법)
- [인프라와 CORS](#인프라와-cors)
  - [답변](#답변-4)
  - [CORS 처리 위치](#cors-처리-위치)
  - [앱 쪽 CORS 확인](#앱-쪽-cors-확인)
  - [인프라 관점 정리](#인프라-관점-정리)
- [4단계 소개](#4단계-소개)
  - [답변](#답변-5)
  - [목표 (4단계)](#목표-4단계)
  - [순서가 중요한 이유](#순서가-중요한-이유)
  - [진행 단계 (4단계)](#진행-단계-4단계)
  - [gitops 변경 (4단계)](#gitops-변경-4단계)
  - [결정 필요 사항 (4단계)](#결정-필요-사항-4단계)
  - [참고 (4단계)](#참고-4단계)
- [외부 접근 제한 방법 비교](#외부-접근-제한-방법-비교)
  - [답변](#답변-6)
  - [선택지 비교](#선택지-비교)
  - [판단 기준](#판단-기준)
  - [Tailscale 전용으로 바꾸면](#tailscale-전용으로-바꾸면)
- [결정 필요 사항 정리](#결정-필요-사항-정리)
  - [답변](#답변-7)
  - [1. 인프라 도구 접근 방식](#1-인프라-도구-접근-방식)
  - [2. 접근 방식별 추가 결정](#2-접근-방식별-추가-결정)
  - [3. 별개로 남은 정리](#3-별개로-남은-정리)
- [확인3](#확인3)
- [확인3 분석](#확인3-분석)
  - [답변](#답변-8)
  - [확장성 기준 선택지](#확장성-기준-선택지)
  - [E2 구조](#e2-구조)
  - [방식 비교 (확장성)](#방식-비교-확장성)
  - [의견 (확인3)](#의견-확인3)
- [E2의 Tailscale 사용 여부](#e2의-tailscale-사용-여부)
  - [답변](#답변-9)
  - [E2 구성 요소별 역할](#e2-구성-요소별-역할)
  - [E2 접속 흐름](#e2-접속-흐름)
  - [E1, E2, A 차이](#e1-e2-a-차이)
- [E2 결정과 4단계 계획](#e2-결정과-4단계-계획)
  - [답변](#답변-10)
  - [완성 후 구조](#완성-후-구조)
  - [설치하는 것](#설치하는-것)
  - [내부 Gateway에 별도 GatewayClass가 필요한 이유](#내부-gateway에-별도-gatewayclass가-필요한-이유)
  - [진행 순서 (E2)](#진행-순서-e2)
  - [gitops 구성안 (E2)](#gitops-구성안-e2)
  - [결정 필요 사항 (E2)](#결정-필요-사항-e2)
  - [참고 (E2)](#참고-e2)
- [E2에서 인증서가 필요한 이유](#e2에서-인증서가-필요한-이유)
  - [답변](#답변-11)
  - [1. Universal SSL은 Cloudflare 경유 시에만 적용](#1-universal-ssl은-cloudflare-경유-시에만-적용)
  - [2. Tailscale 자동 인증서는 ts.net 전용](#2-tailscale-자동-인증서는-tsnet-전용)
  - [3. .dev는 HTTP로 열 수 없음](#3-dev는-http로-열-수-없음)
  - [Cloudflare API 토큰이 필요한 이유](#cloudflare-api-토큰이-필요한-이유)

---

# 개요

- 2단계(Cloudflare Tunnel, cloudflared, DNS)가 끝나 `choimory.dev`, `staging.choimory.dev`, `staging-member-api.choimory.dev`가 외부에서 응답함
- 3단계는 브라우저에서 실제 사용자처럼 사용하며 앱 동작을 확인하는 단계
- k8s-setup 때 도메인이 없어 미뤄 둔 브라우저 흐름(CORS, 쿠키) 확인
- 참고 문서: [plan3.md](./plan3.md)(전체 단계), [task3.md](./task3.md)(2단계 결과)

---

# 3단계 소개

> 3단계 소개해줘

## 답변

- 2단계는 curl로 도메인이 앱까지 닿는 것을 확인
- 3단계는 브라우저에서만 드러나는 문제(CORS, 쿠키, HTTPS 혼합 콘텐츠 등)를 확인

## 브라우저로 따로 확인하는 이유

| 구분 | curl (2단계) | 브라우저 (3단계) |
|---|---|---|
| CORS | 검사하지 않음 | 다른 도메인 API 호출 시 Origin을 검사해 차단 가능 |
| 쿠키 | 직접 넣지 않으면 없음 | `SameSite`, `Secure`, `Domain` 규칙에 따라 저장, 전송 여부가 갈림 |
| JS 실행 | 없음 | front가 빌드 시 고정된 API 주소(`NEXT_PUBLIC_MEMBER_API_BASE_URL`)로 호출 |
| HTTPS | 인증서만 확인 | `http://` 리소스가 섞이면 차단 |

## 요청 흐름 두 가지

```
1) 브라우저 → https://staging-member-api.choimory.dev (빌드 시 고정된 주소, CORS 적용)
2) 브라우저 → https://staging.choimory.dev (Next 서버) → http://choimory-dev-member-api:8080 (클러스터 내부, CORS 무관)
```

- 화면 기능마다 어느 경로를 쓰는지에 따라 문제 위치가 다름

## 확인 항목 (stg)

| 순서 | 항목 | 확인 방법 | 정상 |
|---|---|---|---|
| 1 | 화면 로드 | `https://staging.choimory.dev` 접속, 개발자도구 Console | 자물쇠 표시, Console 오류 없음 |
| 2 | API 주소 | 개발자도구 Network에서 API 요청 도메인 | `staging-member-api.choimory.dev` (localhost, 옛 `staging-api...` 아님) |
| 3 | CORS | Network에서 API 요청과 preflight(`OPTIONS`) | `Access-Control-Allow-Origin: https://staging.choimory.dev`, CORS 오류 없음 |
| 4 | 회원가입 | 화면에서 가입 | 성공 응답, DB에 행 생성 |
| 5 | 로그인 | 화면에서 로그인 | 성공, 토큰이나 쿠키 저장 (Application 탭) |
| 6 | 쿠키 속성 | Application → Cookies | `Secure`, `SameSite=Lax`, Domain이 의도한 값 |
| 7 | 인증 API | 로그인 후 내 정보 조회 등 | 200 |
| 8 | 토큰 재발급 | 액세스 토큰 만료 후 또는 재발급 동작 | 새 토큰 발급, Redis 저장 |
| 9 | 로그아웃 | 화면에서 로그아웃 | 쿠키, 토큰 삭제, Redis에서 제거 |
| 10 | 외부망 | 휴대폰 LTE(와이파이 끔)로 접속 | 집 네트워크와 같은 결과 |

## prod

- 화면 로드, API 주소, CORS(1~3번)만 확인 추천
- 5차와 같이 prod에는 테스트 계정을 만들지 않음. 로그인 실패(401) 응답이 정상으로 오는지만 확인

## 문제 시 고칠 곳

| 증상 | 원인 위치 | 고칠 곳 |
|---|---|---|
| CORS 오류 | member-api 허용 Origin 설정 | member-api 앱 저장소 (재빌드, 배포) |
| API 주소가 틀림 | front 빌드 변수 | front 저장소 GitHub Environments 변수 → 재빌드 |
| 쿠키가 저장되지 않음 | member-api 쿠키 속성 (`Secure`, `SameSite`, `Domain`) | member-api 앱 저장소 |
| 502/504 | Gateway, Pod 상태 | gitops, 서버 확인 |
| 혼합 콘텐츠 경고 | front 코드의 `http://` 주소 | front 앱 저장소 |

- 3단계는 gitops 변경이 없을 가능성이 큼. 문제가 생기면 대부분 앱 저장소 쪽

## 진행 방식

1. 브라우저에서 확인 항목을 차례로 진행. 이상한 부분은 Console, Network 오류를 캡처하거나 붙여서 공유
2. 회원가입, 로그인 결과는 서버에서 DB, Redis로 함께 확인 (필요 시 명령 제공)
3. 결과는 task4에 기록

---

# 추가1

- 개발서버, 운영서버 모두 회원가입 정상
- 로그인시 api 응답 ok 확인
- 마이페이지 진입 확인
- 근데 메인페이지랑 상단 로그인 버튼이 비회원에게 보이는 내용으로 보임 (메인이 랜딩페이지고, 로그아웃버튼이 아닌 여전히 로그인 버튼)

---

# 추가1 분석

> - 개발서버, 운영서버 모두 회원가입 정상
> - 로그인시 api 응답 ok 확인
> - 마이페이지 진입 확인
> - 근데 메인페이지랑 상단 로그인 버튼이 비회원에게 보이는 내용으로 보임 (메인이 랜딩페이지고, 로그아웃버튼이 아닌 여전히 로그인 버튼)

## 답변

- 회원가입, 로그인, 마이페이지는 정상이고 메인, 헤더만 비로그인 상태로 보이는 문제
- 마이페이지와 메인, 헤더가 로그인 상태를 판단하는 쪽이 서로 다를 가능성이 큼

## 유력한 원인: 쿠키가 API 도메인에만 저장됨

| 화면 | 로그인 판단 방식 (추정) | 쿠키 전송 |
|---|---|---|
| 마이페이지 | 브라우저가 member-api(`staging-member-api.choimory.dev`)를 직접 호출 | 전송됨. 쿠키가 그 도메인에 저장 |
| 메인, 헤더 | Next 서버(`staging.choimory.dev`)가 요청 쿠키를 보고 판단 (서버 렌더링, middleware 등) | 전송 안 됨. 쿠키가 API 도메인에만 있음 |

- member-api가 로그인 응답 `Set-Cookie`에 `Domain`을 지정하지 않으면 쿠키는 `staging-member-api.choimory.dev` 전용으로 저장되어 `staging.choimory.dev` 요청에 붙지 않음

## 로컬에서 됐던 이유

- `localhost:3000`(front)과 `localhost:8080`(api)은 포트만 다른 같은 호스트
- 쿠키는 포트를 구분하지 않아 로컬에서는 드러나지 않음. 서브도메인으로 나뉜 지금 처음 나타나는 전형적인 증상

## 확인 방법

- stg에서 로그인한 상태로 브라우저 개발자도구 사용

| 순서 | 확인 | 보는 것 |
|---|---|---|
| 1 | 메인에서 새로고침 | 새로고침해도 비회원 화면이면 쿠키 원인 가능성 높음. 새로고침하면 바뀌면 클라이언트 상태 갱신 문제 |
| 2 | Network → 로그인 요청 → Response Headers | `Set-Cookie`의 `Domain`, `SameSite`, `Secure`, `Path` |
| 3 | Application → Cookies | 쿠키가 `https://staging-member-api.choimory.dev`에만 있는지, `https://staging.choimory.dev`에도 있는지 |
| 4 | Application → Local Storage | 토큰을 localStorage에 저장하는 구조인지 |
| 5 | Network → 메인 페이지 문서 요청 → Request Headers | `Cookie`에 인증 쿠키가 실려 가는지 |

- 2, 3번 캡처로 원인 확정 가능. 쿠키 값(토큰)은 마스킹

## 원인별 수정 방향

| 원인 | 수정 | 주의 |
|---|---|---|
| 쿠키가 API 도메인 전용 | A. member-api가 `Domain=choimory.dev`로 쿠키 발급 | stg와 prod가 쿠키를 공유하게 됨. `choimory.dev`가 prod이고 `staging.choimory.dev`가 그 하위라 stg 로그인 쿠키가 prod에도 전송됨. 환경별 쿠키 이름 분리 필요 (예: `stg_access_token`) |
| | B. 로그인 요청을 front의 Next 서버(Route Handler)를 거치게 하고 front 도메인에 쿠키 발급 (BFF 방식) | 쿠키가 환경별 도메인에 분리되어 섞이지 않음. Next 서버가 이미 member-api를 내부 호출하는 구조(`MEMBER_API_BASE_URL`)라 맞추기 쉬울 수 있음 |
| 클라이언트 상태 갱신 누락 (새로고침하면 정상) | 로그인 성공 후 `router.refresh()` 또는 전역 상태 갱신 | 쿠키 문제와 별개 |
| 메인이 빌드 시 정적 생성 | 메인을 동적 렌더링으로 | 응답 헤더가 `no-store`라 가능성 낮음 |

- 모두 앱 저장소 쪽 수정. front, member-api 저장소가 이 디렉토리에 없어 코드는 미확인. 저장소 경로를 받으면 로그인 판단 코드와 쿠키 발급 코드 확인

## prod 테스트 계정

- prod에서도 회원가입을 해 테스트 계정이 생김. 3단계가 끝나면 삭제하거나 테스트용임을 기록 추천

---

# 확인2

- 새로고침해도 그대로임
- local, session 스토리지에는 저장값없음
- 쿠키에는 리프레쉬 토큰 저장되어 있음

---

# 확인2 분석

> ![img.png](img.png)
>
> - 새로고침해도 그대로임
> - local, session 스토리지에는 저장값없음
> - 쿠키에는 리프레쉬 토큰 저장되어 있음

## 답변

- 추가1 분석의 "쿠키가 API 도메인에만 저장됨" 추정은 틀림. 쿠키는 이미 front 도메인에 저장되어 있음 (front가 BFF 방식)
- 원인은 로그인 상태 복구(silent refresh)가 공통 레이아웃에 없는 것으로 추정

## 캡처의 refresh token 노출

- `img.png`의 Value 칸에 prod refresh_token(JWT) 앞부분이 보임. 커밋하면 public 저장소에 올라감
- 커밋 전 이미지의 토큰 값 마스킹 또는 이미지 제외
- 다른 곳에 공유됐을 수 있으면 prod에서 로그아웃해 해당 refresh token 무효화 (Redis에서 삭제)

## 캡처에서 확인된 것

| 항목 | 값 | 의미 |
|---|---|---|
| 페이지 | `choimory.dev` (prod) | |
| 쿠키 | `refresh_token` 하나 | access token은 어디에도 저장 안 됨 (Local, Session 저장소 비어 있음) |
| Domain | `choimory.dev` (앞에 점 없음) | front 도메인 전용. front Next 서버(`/api/auth`)가 발급 → 이미 BFF 방식 |
| Path | `/api/auth` | `/api/auth/...` 요청에만 쿠키 전송. 메인(`/`) 등 페이지 요청에는 안 붙음 |
| HttpOnly, Secure, SameSite | ✓, ✓, Lax | JS로 읽을 수 없음. 보안 설정 적절 |

## 원인 추정 (수정)

- "access token은 메모리, refresh token은 `/api/auth` 전용 쿠키" 구조

| 시점 | 상태 |
|---|---|
| 로그인 직후 | access token이 JS 메모리에 있음 |
| 새로고침, 다른 페이지 이동 | 메모리가 비어 access token 없음 |
| 메인, 헤더 | access token이 없으면 비회원 표시. refresh를 시도하지 않으면 계속 비회원 |
| 마이페이지 | 보호된 페이지라 진입 시 `/api/auth/refresh`로 토큰을 다시 받는 로직이 있는 것으로 보임 → 정상 |

- 앱이 처음 뜰 때 로그인 상태를 복구하는 로직(silent refresh)이 마이페이지에만 있고 공통 레이아웃(헤더)에 없는 것으로 보임
- 쿠키 Path가 `/api/auth`라 Next 서버가 메인을 렌더링할 때도 로그인 여부를 알 수 없음 → 새로고침해도 비회원

## 확인할 것 (Network 탭)

| 화면 | 보는 것 |
|---|---|
| 메인 새로고침 | `/api/auth/refresh` 같은 요청이 나가는지 |
| 마이페이지 진입 | 어떤 요청이 나가는지 (refresh 요청, 응답 200 여부) |
| 로컬 개발 환경 | 로컬에서 메인이 로그인 상태로 보였는지 |

## 수정 방향 (front)

| 방법 | 내용 | 의견 |
|---|---|---|
| A. 공통 레이아웃에서 silent refresh | 앱이 처음 뜰 때(최상위 Provider) `/api/auth/refresh` 한 번 호출, 성공하면 access token과 사용자 정보를 전역 상태에 넣고 헤더, 메인이 구독 | 추천. 메모리 access token + HttpOnly refresh 구조를 유지하는 표준 방식 |
| B. 로그인 표시용 쿠키 추가 | Path `/`인 `logged_in=1` 같은 비민감 쿠키를 같이 발급해 서버 렌더링 시 헤더 표시에 사용 | A와 함께 쓰면 첫 화면 깜빡임 감소. 실제 인증은 A |
| C. refresh_token Path를 `/`로 | 서버가 모든 요청에서 쿠키를 봄 | 비추천. refresh token 노출 범위 증가 |

- gitops, 인프라 문제가 아닌 front 앱 수정 필요. front 저장소 경로를 받으면 마이페이지 refresh 로직과 헤더 로그인 판단 코드 확인

---

# 프론트 문제 여부

> 프론트문제임?

## 답변

- front 앱 문제일 가능성이 높음. 인프라(도메인, Tunnel, Gateway)는 정상

## 인프라가 아니라고 보는 근거

| 확인된 것 | 의미 |
|---|---|
| 회원가입, 로그인 API 성공 | Cloudflare → Tunnel → Gateway → 앱 정상 왕복 |
| 마이페이지 진입 성공 | 로그인 상태 복구 자체는 가능 (refresh 쿠키, member-api, Redis 정상) |
| refresh 쿠키가 `Secure`로 front 도메인에 저장 | HTTPS, 쿠키 발급 정상 |
| `cf-cache-status: DYNAMIC` | Cloudflare가 페이지를 캐시해 옛 화면을 보여주는 것 아님 |

- 같은 쿠키, 같은 서버로 마이페이지는 되고 메인, 헤더만 안 됨 → 화면마다 로그인 판단 코드가 다름 (front 코드 영역)

## 확정 방법

- 메인에서 Network 탭을 연 상태로 새로고침

| 결과 | 판단 |
|---|---|
| `/api/auth/refresh` 등 요청이 아예 안 나감 | front 문제 확정. 헤더, 메인에 로그인 복구 로직 없음 |
| 요청이 나가고 200인데도 비회원 표시 | front 문제 확정. 결과를 헤더 상태에 반영하지 않음 |
| 요청이 나가는데 401, 500 등 실패 | member-api나 쿠키 경로 재확인 |

- 로컬에서도 같다면 원래 없던 기능, 로컬에서 됐다면 환경별 설정 차이. 어느 쪽이든 앱 쪽 수정

---

# 인프라와 CORS

> 그럼 인프라에서는 CORS도 오케이도 다 된거지?

## 답변

- 인프라 작업은 완료. CORS는 인프라가 아닌 member-api 앱이 처리하는 부분

## CORS 처리 위치

| 구간 | CORS 관여 | 상태 |
|---|---|---|
| Cloudflare, Tunnel, cloudflared | 관여 안 함. 헤더 그대로 전달 | 정상 |
| Envoy Gateway (`gateway-proxy`, HTTPRoute) | CORS 설정이 없어 그대로 전달 | 정상 |
| member-api (Spring) | CORS 판단 (허용 Origin, 메서드, 헤더) | 아래 참고 |

- 2단계 curl 응답의 `vary: Origin,Access-Control-Request-Method,...`는 member-api CORS 필터가 동작한다는 표시

## 앱 쪽 CORS 확인

- front는 로그인을 자기 도메인의 `/api/auth`(Next 서버)로 처리 → Next 서버가 member-api를 클러스터 안에서 호출해 브라우저 CORS를 거치지 않음
- 실제 CORS 검사 여부는 브라우저가 member-api 도메인을 직접 호출하는지에 달림

| Network 탭에서 | 판단 |
|---|---|
| `member-api.choimory.dev`(또는 `staging-member-api...`) 요청이 있고 성공 | CORS 정상 (응답에 `Access-Control-Allow-Origin: https://choimory.dev`) |
| 그 도메인 요청이 아예 없음 | 모든 호출이 front를 거치는 구조라 CORS 미사용. 문제없음 |
| Console에 `CORS policy` 오류 | member-api 허용 Origin 수정 (앱 저장소) |

## 인프라 관점 정리

| 항목 | 상태 |
|---|---|
| 도메인, DNS, Cloudflare | 완료 |
| HTTPS (Universal SSL) | 완료 |
| Tunnel, cloudflared, Gateway 라우팅 | 완료 |
| 앱 도메인 접속 (stg, prod) | 완료 |
| 남은 정리 | 서버 인증 파일 원본 삭제, Always Use HTTPS 켜기, AI Crawl Control Block |
| 다음 인프라 작업 | 4단계 ArgoCD 도메인과 Access, 5단계 Tailscale DB 접속 |

- 헤더 로그인 표시는 front에서 따로 수정. 인프라 작업은 4단계로 진행 가능

---

# 4단계 소개

> 4단계 소개해줘

## 답변

- ArgoCD를 도메인으로 열되 Cloudflare Access 로그인을 통과한 본인만 접근하게 하는 단계. 마지막에 ArgoCD 초기 비밀번호 정리
- 이후 Grafana, Prometheus 등 인프라 도구에 같은 구성 재사용

## 목표 (4단계)

```
브라우저 → argocd.choimory.dev
  → Cloudflare Access (로그인 화면: 본인 이메일만 통과)
  → Tunnel → cloudflared → gateway-proxy → HTTPRoute → argocd-server
  → ArgoCD 로그인
```

- 앞단 Cloudflare Access, 뒷단 ArgoCD 자체 로그인의 이중 잠금
- Tunnel이라 Cloudflare를 우회해 ArgoCD로 직접 들어오는 경로 없음

## 순서가 중요한 이유

- `*.choimory.dev`가 와일드카드로 Tunnel에 연결되어 있어 ArgoCD HTTPRoute를 push하는 순간 인터넷에 공개됨
- Cloudflare Access를 먼저 걸고 HTTPRoute를 push
- 지금 `argocd.choimory.dev`는 HTTPRoute가 없어 Envoy가 404 응답 (안전)

## 진행 단계 (4단계)

| 순서 | 위치 | 할 일 | 담당 |
|---|---|---|---|
| 1 | Cloudflare Zero Trust | 처음이면 팀 이름 설정, 로그인 수단 설정 (이메일 OTP 또는 GitHub) | 사용자 |
| 2 | Cloudflare Zero Trust | Access Application(Self-hosted) `argocd.choimory.dev` 생성, 본인 이메일만 허용 (인프라 도구 공통 정책으로 재사용) | 사용자 |
| 3 | 브라우저 | `argocd.choimory.dev` 접속 시 Access 로그인 화면이 먼저 뜨는지 확인 (통과 후 아직 404가 정상) | 사용자 |
| 4 | gitops | 아래 변경 작성 | Claude 작성, 사용자 push |
| 5 | 브라우저 | Access 통과 → ArgoCD 로그인 화면 확인 | 사용자 |
| 6 | 서버, 브라우저 | 초기 비밀번호 로그인 → 비밀번호 변경 → `argocd-initial-admin-secret` 삭제 | 사용자 (명령 제공) |

## gitops 변경 (4단계)

| 파일 | 변경 | 이유 |
|---|---|---|
| `infrastructure/apps/namespace.yaml` | `choimory-infra`에 `gateway-access: "true"` 라벨 | Gateway가 이 라벨이 있는 네임스페이스의 HTTPRoute만 받음 |
| `infrastructure/argocd/values.yaml` | `configs.params."server.insecure": true` | TLS는 Cloudflare, Gateway는 HTTP로 전달. 없으면 argocd-server가 HTTPS로 리다이렉트해 무한 리다이렉트 |
| `infrastructure/argocd/values.yaml` | `configs.cm.url: https://argocd.choimory.dev` | ArgoCD가 외부 주소를 알아야 링크가 맞게 생성 |
| ArgoCD HTTPRoute (위치 결정 필요) | `argocd.choimory.dev` → `argocd-server:80` | 도메인 연결 |
| `README.md` | 인프라 도구 접속 방식 | |

| HTTPRoute 위치 | 내용 | 의견 |
|---|---|---|
| A. ArgoCD 차트의 HTTPRoute 기능 | values에 `server.httproute` (차트 `10.17.0` 지원 여부 작업 시 확인) | 추천. ArgoCD 설정이 values 한 곳에 모임 |
| B. 별도 디렉토리 | 예: `infrastructure/routes/argocd.yaml` + Application | 이후 Grafana 등 라우트도 모을 수 있음 |

## 결정 필요 사항 (4단계)

1. ArgoCD 도메인: `argocd.choimory.dev` 여부
2. Access 로그인 수단: 이메일 OTP(간단) / GitHub 로그인(OAuth 앱 등록 필요)
3. HTTPRoute 위치: A(차트 기능) / B(별도 디렉토리)
4. Access 세션 유지 시간: 기본 24시간 / 더 길게(예: 1주)

## 참고 (4단계)

- 외부에서 ArgoCD CLI를 쓰려면 Access 때문에 `cloudflared access login` 필요. 서버에서 `kubectl`로 관리하면 무관
- `server.insecure` 변경 시 argocd-server 재시작. ArgoCD가 자기 자신을 Git으로 관리하므로 push만 하면 됨. UI가 잠깐 끊길 수 있으나 배포 동작은 영향 없음
- Zero Trust 무료 플랜은 사용자 50명까지

---

# 외부 접근 제한 방법 비교

> 궁금한게 결국 외부의 접근을 제한하는거잖아. 그럼 Cloudflare Access말고 Gateway나 ACL 같은 다른걸로 할수는 없는거임?

## 답변

- 가능. 외부 접근 제한을 어디서 거느냐에 따라 선택지가 여러 개 있음

## 선택지 비교

| 방법 | 차단 위치 | 통과 대상 | 장점 | 단점 |
|---|---|---|---|---|
| A. Cloudflare Access (현재 계획) | Cloudflare 엣지 | 로그인한 사람 (이메일 OTP, GitHub 등) | 클러스터에 닿기 전에 차단. 간단, 무료. 도구 추가 시 도메인만 등록 | 설정이 Cloudflare 대시보드에 있어 gitops 밖 |
| B. Gateway IP 허용 목록 (Envoy SecurityPolicy) | Envoy Gateway | 특정 IP | gitops 관리 | Tunnel 뒤라 실제 IP는 `CF-Connecting-IP` 헤더로 봐야 하는 추가 설정 필요. 집 IP가 바뀌고 스터디카페, LTE IP가 매번 달라 사실상 사용 어려움 |
| C. Cloudflare WAF IP 규칙 | Cloudflare 엣지 | 특정 IP | 간단 | B와 같은 IP 문제 |
| D. Gateway 인증 (Envoy SecurityPolicy OIDC, Basic Auth) | Envoy Gateway | 로그인한 사람 | gitops 관리. Cloudflare에 묶이지 않음 | OIDC는 Google 등에 앱 등록, client secret을 SealedSecret으로 관리. Basic Auth는 간단하지만 보안이 약함. 요청이 클러스터까지 들어온 뒤 차단 |
| E. 외부 공개 없이 Tailscale 전용 (Tailscale ACL) | 네트워크 자체 | tailnet에 연결된 내 기기 | 인터넷에 노출되지 않아 가장 안전. 5단계 DB용 Tailscale Operator 재사용. ACL도 Tailscale에서 관리 | Tailscale을 켠 기기에서만 접속. 주소가 `argocd.<tailnet>.ts.net` 형태 (`choimory.dev` 아님) |

## 판단 기준

- Tailscale이 없는 기기(회사 PC, 남의 컴퓨터)에서 ArgoCD를 볼 일이 있는지

| 상황 | 추천 |
|---|---|
| 맥북, 아이패드, 아이폰에 Tailscale이 항상 있음 | E (Tailscale 전용). 가장 안전하고 5단계 DB 접속과 방식이 같아 구성 단순 |
| Tailscale 없는 기기에서도 접속 필요 | A (Cloudflare Access) |
| Cloudflare에 묶이지 않고 전부 gitops로 | D (Gateway OIDC). 설정 부담이 가장 큼 |
| B, C (IP 기반) | 접속 위치가 자주 바뀌는 환경이라 비추천 |

## Tailscale 전용으로 바꾸면

- 4단계와 5단계가 "Tailscale Operator 설치 → ArgoCD, DB를 tailnet에 노출" 하나로 합쳐짐
- 불필요해지는 것: Cloudflare Access, ArgoCD HTTPRoute, `choimory-infra`의 `gateway-access` 라벨
- `server.insecure` 필요 여부는 노출 방식에 따라 작업 때 결정. Tailscale이 HTTPS 인증서 자동 발급
- 이후 Grafana도 같은 방식으로 tailnet에 노출
- k8s-setup 때 "인프라 도구는 도메인 + Access"로 정했지만 그때는 DB만 Tailscale로 하기로 한 시점. Tailscale Operator를 어차피 설치하게 되어 다시 판단할 시점

---

# 결정 필요 사항 정리

> 정해야하는게 뭐지?

## 답변

- 지금 정할 것은 사실상 하나(인프라 도구 접근 방식). 그 답에 따라 나머지 결정 항목이 정해짐

## 1. 인프라 도구 접근 방식

- Tailscale이 없는 기기(회사 PC, 남의 컴퓨터 등)에서 ArgoCD를 볼 일이 있는지

| 답 | 방식 | 결과 |
|---|---|---|
| 없음 (내 기기에 Tailscale이 항상 있음) | E. Tailscale 전용 (추천) | 4, 5단계가 합쳐짐. 인터넷 노출 없음 |
| 있음 | A. Cloudflare Access | 4단계, 5단계 따로 진행 |

## 2. 접근 방식별 추가 결정

### E (Tailscale 전용)

| 항목 | 선택지 | 추천 |
|---|---|---|
| ArgoCD 노출 방식 | Tailscale Ingress(HTTPS 자동) / Service annotation(TCP) | Ingress (브라우저 HTTPS 경고 없음) |
| tailnet 기기 이름 | 예: `argocd`, `choimory-dev-stg-postgres` | 작업 때 표로 제안 |
| PostgreSQL 노출 방식 | 별도 Service를 만들어 노출 / CNPG Cluster 설정으로 추가 Service | 작업 때 확인 후 제안 |
| Tailscale ACL 접근 제한 | 기본(내 tailnet 전체) / 태그로 제한 | 기본으로 시작 |

### A (Cloudflare Access)

| 항목 | 선택지 | 추천 |
|---|---|---|
| ArgoCD 도메인 | `argocd.choimory.dev` 등 | `argocd.choimory.dev` |
| Access 로그인 수단 | 이메일 OTP / GitHub | 이메일 OTP (간단) |
| HTTPRoute 위치 | 차트 기능 / 별도 디렉토리 | 차트 기능 (지원 여부 확인 후) |
| Access 세션 유지 시간 | 24시간 / 1주 등 | 취향 |
| 5단계(DB) | Tailscale Operator로 별도 진행 | |

## 3. 별개로 남은 정리

| 항목 | 위치 |
|---|---|
| 서버의 Tunnel 인증 파일 원본 삭제 | 서버 |
| Always Use HTTPS 켜기, AI Crawl Control Block | Cloudflare 대시보드 |
| `img.png`의 refresh token 마스킹 (커밋 전) | 로컬 |
| prod 테스트 계정 정리 | prod DB |
| 메인, 헤더 로그인 표시 문제 | front 저장소 |

---

# 확인3

- 주로 tailscale 있는 기기에서만 하지만, 아니더라도 (회사맥북) taiscale 등록해놓으면 되니까 큰 상관없음
- 중요한거는 확장성인데, 다른 프로젝트의 grafana, prometheus가 추가되어서 grafana가 두개로 되면? 그럼 도메인으로 구분하는게 낫지 않을까? 그럼 cloudflare를 써야하나? 와 같은 고민은 있음

---

# 확인3 분석

> - 주로 tailscale 있는 기기에서만 하지만, 아니더라도 (회사맥북) taiscale 등록해놓으면 되니까 큰 상관없음
> - 중요한거는 확장성인데, 다른 프로젝트의 grafana, prometheus가 추가되어서 grafana가 두개로 되면? 그럼 도메인으로 구분하는게 낫지 않을까? 그럼 cloudflare를 써야하나? 와 같은 고민은 있음

## 답변

- Tailscale 없는 기기도 등록하면 되므로 접근성은 문제없음. 남은 고민은 확장성
- 도메인으로 구분하는 데 Cloudflare Access, Tunnel이 꼭 필요하지는 않음. Tailscale 전용으로 두면서 도메인으로 구분하는 방법도 있음

## 확장성 기준 선택지

- 예시: choimory 공용 Grafana와 project-b Grafana가 둘 다 있는 경우

| 방식 | 주소 예시 | 구분 방법 | 도구 추가 시 할 일 | 비고 |
|---|---|---|---|---|
| E1. Tailscale 기기별 이름 | `https://choimory-grafana.<tailnet>.ts.net`, `https://project-b-grafana.<tailnet>.ts.net` | 기기 이름 (서브도메인처럼 동작) | Tailscale Ingress 하나 추가 | 가장 단순. 도구마다 Tailscale 프록시 Pod, tailnet 기기 하나 (무료 플랜 기기 수 한도 내) |
| E2. 내부 전용 Gateway + 내 도메인 | `https://grafana.internal.choimory.dev`, `https://grafana-b.internal.choimory.dev` | 도메인 (HTTPRoute) | HTTPRoute 하나 추가 (앱과 같은 방식) | tailnet에만 열린 내부 Gateway 하나. 도메인은 Cloudflare DNS로 tailnet IP를 가리킴. 인터넷 접속 불가 |
| A. Cloudflare Access + 공개 도메인 | `https://grafana.choimory.dev`, `https://grafana-b.choimory.dev` | 도메인 (HTTPRoute) | HTTPRoute 추가 + Access에 도메인 등록 | 인터넷에 공개, 앞을 로그인으로 막음 |

## E2 구조

```
내 기기 (Tailscale 켬)
  → grafana.internal.choimory.dev  (Cloudflare DNS, 회색 구름: tailnet IP 100.x)
  → 내부 Gateway (tailnet에만 노출, Tailscale 기기 1개)
  → HTTPRoute 호스트 매칭 → grafana / argocd / ...
```

- 공개 앱의 `gateway` + HTTPRoute 패턴을 내부용으로 하나 더 두는 형태. 도구 추가 시 HTTPRoute만 추가
- DNS 레코드는 공개되지만 `100.x` 주소는 tailnet 밖에서 닿지 않음
- Cloudflare 프록시를 거치지 않아 Universal SSL을 못 씀 → HTTPS 인증서 직접 발급
  - cert-manager로 Let's Encrypt 와일드카드 인증서(`*.internal.choimory.dev`)
  - 발급 방식은 Cloudflare API 토큰을 쓰는 DNS-01. 토큰은 SealedSecret
  - 자체 인증서라 두 단계 서브도메인 가능
- 추가 설치: cert-manager, 내부 Gateway, Tailscale Operator(어차피 설치 예정)

## 방식 비교 (확장성)

| | E1 | E2 | A |
|---|---|---|---|
| 인터넷 노출 | 없음 | 없음 | 있음 (Access로 보호) |
| 도메인 구분 | `*.ts.net` 기기 이름 | 내 도메인 | 내 도메인 |
| 도구 추가 비용 | Ingress 1개 + 기기 1개 | HTTPRoute 1개 | HTTPRoute + Access 등록 |
| 처음 구축 비용 | 낮음 | 높음 (cert-manager, 내부 Gateway, DNS) | 중간 |
| gitops 관리 범위 | 전부 | 전부 (DNS 레코드 1개만 수동) | Access는 대시보드 |

## 의견 (확인3)

- 도구가 ArgoCD 하나인 지금은 E1로 시작 추천
- 도구가 늘어나는 시점(Grafana, Prometheus 설치 작업)에 E2로 옮기는 것이 부담이 적음. E1 → E2는 노출 방식만 바꾸면 됨
- 처음부터 E2도 가능하나 이번 작업 범위가 cert-manager, 내부 Gateway까지 커짐
- DB(PostgreSQL, Redis)는 TCP라 어느 방식이든 Tailscale Operator로 Service마다 tailnet 기기를 만드는 방식(5단계 계획) 유지
- 결정 필요: E1으로 시작 / 처음부터 E2 / A(Cloudflare Access)

---

# E2의 Tailscale 사용 여부

> e2도 tailscale 쓰는거임?

## 답변

- 사용함. 접속 통로는 Tailscale이고 도메인은 이름표 역할만 함

## E2 구성 요소별 역할

| 구성 요소 | 역할 |
|---|---|
| Tailscale | 실제 접속 통로. 내부 Gateway를 tailnet 기기(`100.x` IP)로 노출. Tailscale을 켠 내 기기만 이 IP에 닿음 |
| Cloudflare DNS | `*.internal.choimory.dev` → `100.x` 주소 안내만 담당 (회색 구름, 프록시, Tunnel 안 거침) |
| cert-manager + Cloudflare API | HTTPS 인증서 발급. DNS 레코드로 도메인 소유를 증명하고 Let's Encrypt 인증서 발급 |
| 내부 Gateway + HTTPRoute | 도메인별로 grafana, argocd 등에 분배 (공개 앱의 `gateway`와 같은 방식) |

## E2 접속 흐름

```
내 기기 (Tailscale 켬)
  1. grafana.internal.choimory.dev 조회 → Cloudflare DNS가 "100.x.x.x" 응답
  2. 100.x.x.x로 접속 → Tailscale 네트워크로 내부 Gateway 도달
  3. 내부 Gateway가 호스트를 보고 grafana로 전달

Tailscale 끈 기기 / 외부인
  1. DNS 조회는 됨 (100.x.x.x를 알게 됨)
  2. 100.x.x.x는 tailnet 전용 주소라 접속 불가
```

## E1, E2, A 차이

| | E1 | E2 | A |
|---|---|---|---|
| 접속 통로 | Tailscale | Tailscale | Cloudflare Tunnel (인터넷) |
| 이름 | Tailscale이 주는 `*.ts.net` | 내 도메인 (Cloudflare DNS) | 내 도메인 |
| 인증서 | Tailscale 자동 발급 | cert-manager로 직접 발급 | Cloudflare Universal SSL |
| 접근 제한 | tailnet 기기만 | tailnet 기기만 | Cloudflare Access 로그인 |

- E1, E2 모두 Tailscale 전용. 차이는 주소가 `*.ts.net`인지 내 도메인인지뿐
- E2에서 Cloudflare는 DNS와 인증서 발급에만 쓰이고 트래픽은 Cloudflare를 거치지 않음

---

# E2 결정과 4단계 계획

> e2로 할래 지금 해두면 나중에 추가만 하면 되잖아

## 답변

- E2로 결정. 한 번 구축하면 이후 도구 추가는 HTTPRoute 하나로 끝남

## 완성 후 구조

```
[공개]  브라우저 → Cloudflare → Tunnel → cloudflared → gateway-proxy → HTTPRoute → 앱
[내부]  내 기기(Tailscale) → *.internal.choimory.dev (DNS: 100.x) → internal-gateway-proxy(tailnet 기기) → HTTPRoute → argocd, (나중에) grafana ...
```

## 설치하는 것

| 구성 | 역할 | 비고 |
|---|---|---|
| cert-manager | Let's Encrypt 와일드카드 인증서 `*.internal.choimory.dev` 발급, 자동 갱신 | Cloudflare API 토큰으로 DNS-01 인증 |
| Tailscale Operator | 내부 Gateway Service를 tailnet 기기로 노출 (5단계 DB에도 재사용) | OAuth 클라이언트 필요 |
| 내부 Gateway 묶음 | GatewayClass, EnvoyProxy, Gateway (HTTPS 443) | 별도 GatewayClass 필요 |
| ArgoCD 설정 | `server.insecure`, `url`, HTTPRoute | |

## 내부 Gateway에 별도 GatewayClass가 필요한 이유

- EnvoyProxy에서 프록시 이름을 `gateway-proxy`로 고정했고, 이 설정은 GatewayClass 단위
- 같은 클래스로 Gateway를 하나 더 만들면 이름이 겹침 (주석으로 남긴 경우)
- 내부용 GatewayClass, EnvoyProxy를 따로 두고, 내부 EnvoyProxy Service는 `LoadBalancer` + `loadBalancerClass: tailscale`로 Tailscale Operator가 tailnet IP를 붙이게 함

## 진행 순서 (E2)

| 순서 | 위치 | 할 일 | 담당 |
|---|---|---|---|
| 1 | Cloudflare | API 토큰 발급 (`choimory.dev` Zone DNS Edit, Zone Read) | 사용자 |
| 2 | Tailscale 관리 화면 | ACL 태그 소유자 추가(`tag:k8s-operator`, `tag:k8s`), OAuth 클라이언트 발급 | 사용자 (화면 안내) |
| 3 | 서버 | 토큰, OAuth 값을 SealedSecret 2개로 암호화 | 사용자 (명령 제공) |
| 4 | gitops | cert-manager, Tailscale Operator, 내부 Gateway, ClusterIssuer, Certificate 작성 | Claude 작성, 사용자 push |
| 5 | 서버 | 인증서 발급(`Ready True`), 내부 Gateway tailnet IP 확인 | 사용자 |
| 6 | Cloudflare DNS | `*.internal.choimory.dev` A 레코드 → `100.x` (회색 구름, DNS only) | 사용자 |
| 7 | gitops | ArgoCD values(`server.insecure`, `url`), HTTPRoute | Claude 작성, 사용자 push |
| 8 | 내 기기 | `https://argocd.internal.choimory.dev` 접속 → 초기 비밀번호 로그인 → 변경 → 초기 Secret 삭제 | 사용자 |

## gitops 구성안 (E2)

```
infrastructure/
├── apps/
│   ├── cert-manager.yaml           # wave 5 (CRD 제공)
│   ├── tailscale-operator.yaml     # wave 5
│   └── internal-gateway.yaml       # wave 6
├── cert-manager/values.yaml
├── tailscale-operator/
│   ├── values.yaml
│   └── operator-oauth.yaml          # SealedSecret
└── internal-gateway/
    ├── gateway.yaml                 # EnvoyProxy, GatewayClass, Gateway(HTTPS)
    ├── issuer.yaml                  # ClusterIssuer (Let's Encrypt, Cloudflare DNS-01)
    ├── certificate.yaml             # *.internal.choimory.dev
    └── cloudflare-api-token.yaml    # SealedSecret
```

- ArgoCD HTTPRoute 위치는 작업 때 결정 (차트 기능 또는 `internal-gateway/routes/`)
- 버전은 작업 시점 최신 안정 버전으로 고정

## 결정 필요 사항 (E2)

| 항목 | 제안 | 비고 |
|---|---|---|
| 내부 도메인 | `internal.choimory.dev` | 짧게 `in.`, `admin.` 등 |
| 내부 Gateway 이름 | `internal-gateway` / 프록시 `internal-gateway-proxy` / 클래스 `envoy-internal` | 기존 `gateway`, `gateway-proxy`와 같은 규칙 |
| 내부 Gateway 허용 범위 | `internal-gateway-access: "true"` 라벨 네임스페이스 | 공개 Gateway와 같은 방식. 다른 프로젝트 Grafana도 라벨만 붙이면 됨 |
| DB 노출(5단계) | 이번에 같이 / 따로 | Operator를 이번에 설치해 이어서 하기 좋음 |

## 참고 (E2)

- tailnet IP 고정: Operator가 상태를 Secret에 저장해 Service를 지우지 않는 한 `100.x` IP 유지. 바뀌면 DNS 레코드 하나만 수정
- 공개 DNS에 `100.x`가 보여도 tailnet 밖에서는 닿지 않아 문제없음
- Let's Encrypt 발급 한도: 와일드카드 하나를 90일마다 갱신해 신경 쓸 수준 아님

---

# E2에서 인증서가 필요한 이유

> e2는 인증서 만들어야 하는 이유가 뭐임? cloudflare를 쓰는게 아니라 tailscale+내가구매한도메인 조합이라서 universal ssl을 안쓰는거라 그런가?

## 답변

- 맞음. 이유는 세 가지

## 1. Universal SSL은 Cloudflare 경유 시에만 적용

| | 공개 앱 (지금) | E2 |
|---|---|---|
| 경로 | 브라우저 → Cloudflare 엣지 → Tunnel → 서버 | 브라우저 → Tailscale → 서버(내부 Gateway) |
| 브라우저가 처음 HTTPS로 대화하는 상대 | Cloudflare | 내부 Gateway |
| 인증서 제시 | Cloudflare (Universal SSL) | 내부 Gateway가 직접 |

- Universal SSL은 Cloudflare 엣지의 인증서라 주황 구름(Proxied)으로 Cloudflare를 거치는 요청에만 쓰임
- E2는 DNS만 Cloudflare(회색 구름)이고 트래픽은 Cloudflare를 거치지 않아 서버가 `*.internal.choimory.dev` 인증서를 직접 가져야 함

## 2. Tailscale 자동 인증서는 ts.net 전용

- E1에서 인증서가 필요 없던 이유는 Tailscale이 `xxx.<tailnet>.ts.net` 인증서를 자동 발급하기 때문
- 내 도메인(`choimory.dev`) 인증서는 Tailscale이 발급할 수 없음

## 3. .dev는 HTTP로 열 수 없음

- Tailscale이 이미 암호화하므로 HTTP로 열면 되지 않나 싶지만, `.dev`는 HSTS preload 도메인이라 브라우저가 `*.choimory.dev`를 무조건 HTTPS로만 접속 → HTTP만 열면 접속 불가
- 자체 서명 인증서는 매번 브라우저 경고, 기기마다 신뢰 등록이 필요해 번거로움
- 브라우저가 신뢰하는 공인 인증서(Let's Encrypt) 필요

## Cloudflare API 토큰이 필요한 이유

| 확인 방식 | 방법 | E2에서 |
|---|---|---|
| HTTP-01 | Let's Encrypt가 인터넷에서 내 서버로 접속해 확인 | 불가. 내부 Gateway는 tailnet 전용 |
| DNS-01 | 내 DNS에 확인용 TXT 레코드를 올려 증명 | 가능. cert-manager가 Cloudflare API로 TXT 레코드 자동 추가, 삭제. 와일드카드 인증서는 이 방식만 가능 |

- cert-manager가 Cloudflare DNS를 조작하도록 DNS 편집 권한만 있는 API 토큰 필요. 90일마다 자동 갱신
