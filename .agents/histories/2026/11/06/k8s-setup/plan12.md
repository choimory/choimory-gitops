# 목차

- [개요](#개요)
- [남은 작업](#남은-작업)
  - [답변](#답변)
  - [1. 바로 이어서 할 작업](#1-바로-이어서-할-작업)
  - [2. Tunnel 이후 작업](#2-tunnel-이후-작업)
  - [3. 직접 정리할 것](#3-직접-정리할-것)
  - [4. 범위를 정해야 하는 것](#4-범위를-정해야-하는-것)
  - [5. 개선 후보](#5-개선-후보)
  - [6. 문서 마무리](#6-문서-마무리)
  - [추천 순서](#추천-순서)
- [Cloudflare Tunnel 작업으로 묶기](#cloudflare-tunnel-작업으로-묶기)
  - [답변](#답변-1)
  - [Cloudflare Tunnel 작업으로 묶을 것](#cloudflare-tunnel-작업으로-묶을-것)
  - [그 밖의 남은 것](#그-밖의-남은-것)
  - [작업 단위 제안](#작업-단위-제안)
- [모니터링은 설치 작업으로 분리](#모니터링은-설치-작업으로-분리)
  - [답변](#답변-2)
  - [정리](#정리)
  - [Tunnel 작업 때 챙길 것](#tunnel-작업-때-챙길-것)
  - [작업 단위](#작업-단위)
- [k8s-setup 마무리 범위](#k8s-setup-마무리-범위)
  - [답변](#답변-3)
  - [범위 정리](#범위-정리)
  - [k8s-setup 마무리 시 할 일](#k8s-setup-마무리-시-할-일)
- [DB는 Tailscale 전용 접속](#db는-tailscale-전용-접속)
  - [답변](#답변-4)
  - [방법 비교](#방법-비교)
  - [다음 작업 범위](#다음-작업-범위)

---

# 개요

- task9에서 7단계(앱 배포와 1차 확인)를 완료함. stg, prod 모두 정상
- 이 문서는 7단계 이후 남은 작업과 작업 단위를 정리함
- 참고 문서: [plan9.md](./plan9.md)(범위를 정할 것), [plan10.md](./plan10.md)(인프라 도구 접속 연기), [plan11.md](./plan11.md)(앱 배포 순서), [task8.md](./task8.md)(네임스페이스 변경), [task9.md](./task9.md)(앱 배포)

---

# 남은 작업

> 남은 작업이 cloudflare tunnel, db 접속 정도인가?

## 답변

- 아님. 두 가지 외에도 남은 항목이 있어 문서(plan9~11, task8~9)에 남겨둔 것을 모아 정리함

## 1. 바로 이어서 할 작업

| 작업 | 내용 |
|---|---|
| Cloudflare Tunnel (2차) | cloudflared를 클러스터에 배포하고 앱 도메인 4개 연결 (`choimory.dev`, `staging.choimory.dev`, `member-api.choimory.dev`, `staging-member-api.choimory.dev`) |
| 브라우저 흐름 확인 | 도메인이 연결돼야 확인 가능: front → member-api 호출, CORS, 쿠키, 로그인 화면 동작. 지금은 port-forward와 curl로 API만 확인함 |
| DB 접속 | 방법은 정리 완료. DataGrip에 연결만 등록하면 됨. 매번 port-forward가 번거로우면 인프라 접속 작업 때 함께 개선 |

## 2. Tunnel 이후 작업

- plan10에서 미룬 것

| 작업 | 내용 |
|---|---|
| 인프라 도구 도메인 연결 | ArgoCD 등을 구매한 도메인과 Cloudflare Access(로그인 관문)로 연결 |
| ArgoCD 초기 비밀번호 | 위 작업의 첫 단계: 로그인 → 비밀번호 변경 → `argocd-initial-admin-secret` 삭제 |
| 모니터링 | Grafana, Prometheus는 아직 설치하지 않음. 접속뿐 아니라 설치부터 필요 |

## 3. 직접 정리할 것

- sealed-secrets 키 백업(`~/sealed-secrets-key-backup.yaml`)을 서버 밖에 보관하고 서버의 파일 삭제
- gitops 로컬 레포에서 `main`을 pull 받아 `dev`에 반영 (Actions의 태그 커밋 3개)

## 4. 범위를 정해야 하는 것

- plan9에서 보류한 것
- etcd와 `/etc/kubernetes/pki` 백업: 클러스터 복구 시 필요
- PostgreSQL 백업 (CloudNativePG 백업 설정): prod에 실제 데이터가 쌓이기 전 추천
- Uptime Kuma, Homepage 연동
- 배포 실패 알림
- HPA 테스트

## 5. 개선 후보

- 급하지 않음
- member-api Dockerfile이 빌드마다 의존성을 새로 받는 구조. Maven `429`가 반복되면 개선
- Spring Boot 버전업. 관리하는 Testcontainers가 `1.21.4` 이상이 되면 `ext` 설정 삭제 가능
- Gradle `8.14.2` 오래된 버전 경고
- `ubuntu-latest`가 Ubuntu 26으로 바뀌는 데 따른 러너 변화 확인

## 6. 문서 마무리

- task9를 닫고, k8s-setup 작업이 끝날 때 done 문서 작성
- README에 현재 구조 반영 (네임스페이스 `choimory-infra`, 디렉토리 구성, 서브도메인, 수동 배포 방식)
- calendar를 `DONE`으로 변경

## 추천 순서

1. Cloudflare Tunnel + 브라우저 흐름 확인
2. PostgreSQL 백업 (prod 데이터가 쌓이기 전)
3. 인프라 도구 도메인 연결 + ArgoCD 비밀번호 + 모니터링 설치
4. 나머지 범위 결정 항목

- k8s-setup 작업을 어디까지로 볼지 정할 필요 있음. 예: Tunnel까지를 이번 작업으로 닫고, 백업과 모니터링은 새 작업 디렉토리로 분리

---

# Cloudflare Tunnel 작업으로 묶기

> cloudflare tunnel, argocd 계정설정, 브라우저 접속은 다 cloudflare tunnel로 도메인 연결해서 외부접속한 후에 진행할수 있는 내용들이라고 볼수 있을듯

## 답변

- 맞음. 세 가지 모두 도메인으로 외부 접속이 되어야 진행하거나 확인할 수 있는 작업이라 하나로 묶음

## Cloudflare Tunnel 작업으로 묶을 것

| 순서 | 작업 | Tunnel이 필요한 이유 |
|---|---|---|
| 1 | Cloudflare Tunnel로 앱 도메인 4개 연결 | 작업 자체 |
| 2 | 브라우저 접속 확인 (front → member-api, CORS, 쿠키, 로그인 화면) | 실제 도메인과 HTTPS가 있어야 CORS, 쿠키를 제대로 확인 가능 |
| 3 | ArgoCD 도메인 연결 + Cloudflare Access | 인프라 도구를 외부로 여는 작업 |
| 4 | ArgoCD 계정 설정 (로그인 → 비밀번호 변경 → 초기 Secret 삭제) | 3번 직후 진행. Access를 먼저 걸고 계정을 바꿈 |

- ArgoCD 계정 설정은 엄밀히는 지금도 port-forward로 가능하지만, 도메인 연결 때 한 번에 처리하기로 앞서 정해 이 묶음에 포함
- 모니터링(Grafana, Prometheus)은 설치부터 필요해 이 묶음과 별개 작업으로 두는 것을 추천

## 그 밖의 남은 것

| 구분 | 항목 |
|---|---|
| 직접 정리 | sealed-secrets 키 백업 서버 밖 보관, gitops `main` pull, DataGrip에 DB 연결 등록 |
| 범위 결정 | PostgreSQL 백업, etcd·pki 백업, 모니터링, 알림, HPA, Uptime Kuma·Homepage 연동 |
| 개선 후보 | Dockerfile 의존성 다운로드, Spring Boot 버전업, Gradle 버전, 러너 이미지 변경 |

## 작업 단위 제안

1. k8s-setup은 여기서 마무리
   - 클러스터 설치부터 7단계(앱 배포와 1차 확인)까지가 이번 작업 범위
   - task9를 닫고 done 문서, README, calendar `DONE`으로 정리
2. Cloudflare Tunnel은 새 작업 디렉토리로 시작
   - 예: `cloudflare-tunnel`
   - 위 묶음(1~4)을 plan1부터 새로 진행
3. 백업, 모니터링 등은 그 뒤에 각각 새 작업으로

- 이렇게 나누면 k8s-setup 문서가 더 길어지지 않고 Tunnel 작업 기록도 따로 찾아보기 쉬움
- 결정 필요: 이 방식으로 진행할지, Cloudflare Tunnel도 k8s-setup 안에서 이어서 할지

---

# 모니터링은 설치 작업으로 분리

> grafana, prometheus는 일단 argocd로 인프라도구 도메인부여 및 접속 되면 나머지 인프라 도구들은 설치하고 도메인만 부여하면 되는것이기 때문에 도메인작업이라기보단 나중에 할 설치작업이고

## 답변

- 맞음. ArgoCD로 인프라 도구에 도메인을 붙이는 방식을 한 번 만들어 두면, Grafana와 Prometheus는 설치 작업이 되고 도메인은 같은 방식으로 붙이기만 하면 됨

## 정리

| 작업 | 성격 | 시점 |
|---|---|---|
| Cloudflare Tunnel + 앱 도메인 + 브라우저 확인 | 도메인 작업 | 다음 작업 |
| ArgoCD 도메인 + Cloudflare Access + 계정 설정 | 도메인 작업. 인프라 도구 공통 방식을 만드는 단계 | 위와 함께 |
| Grafana, Prometheus 등 | 설치 작업. 도메인은 위 방식을 그대로 재사용 | 나중에 별도 작업 |

## Tunnel 작업 때 챙길 것

- ArgoCD 연결 시 이후 도구도 같은 방식으로 붙일 수 있게 구성
  - Tunnel: `*.choimory.dev` 와일드카드로 연결. 도구를 추가해도 Tunnel 설정은 그대로
  - 라우팅: 도구마다 HTTPRoute(또는 그에 맞는 설정) 하나만 추가하면 연결되게 함
  - 접근 제한: Cloudflare Access에 인프라 도구 공통 정책을 두고, 도메인만 추가하면 같은 로그인 관문이 걸리게 함
- 이렇게 해 두면 Grafana 설치 시 할 일은 설치, 도메인 하나 추가, Access 등록뿐

## 작업 단위

1. k8s-setup: 7단계까지로 마무리
2. cloudflare-tunnel (새 작업): 앱 도메인, 브라우저 확인, ArgoCD 도메인과 Access, ArgoCD 계정 설정
3. 모니터링 설치 (나중에 새 작업): Grafana, Prometheus 설치. 도메인은 2번 방식 재사용
4. 백업 등 나머지는 범위를 정한 뒤 각각 새 작업으로

---

# k8s-setup 마무리 범위

> 그럼 k8s-setup은 k8s 클러스터 설치 작업이므로 마무리됐다고 볼수있고, 나중에 도메인작업할때 cloudflare tunnel이랑 프론트, 인프라도구, db등에 도메인 연동하면 될듯

## 답변

- 맞음. k8s-setup의 목표였던 클러스터 설치부터 앱 배포까지는 끝났으므로 마무리하고, 도메인 연동은 다음 작업으로 넘김

## 범위 정리

| 작업 | 범위 | 상태 |
|---|---|---|
| k8s-setup | kubeadm 클러스터, Cilium, ArgoCD, 공용 도구, gitops 구조, SealedSecret, CI/CD, stg·prod 배포와 1차 확인 | 완료 |
| 도메인 작업 (다음, 새 작업) | Cloudflare Tunnel, 앱 도메인과 브라우저 확인, 인프라 도구 도메인과 Access, ArgoCD 계정 설정, DB 접속 방식 | 예정 |
| 모니터링 설치 (나중에) | Grafana, Prometheus 설치. 도메인은 도메인 작업의 방식 재사용 | 예정 |
| 백업 등 | 범위를 정한 뒤 각각 새 작업 | 미정 |

## k8s-setup 마무리 시 할 일

1. `done1.md` 작성: 목표, 최종 구성(스펙, 디렉토리, 네임스페이스, 배포 흐름), 단계별 결과, 겪은 문제와 해결, 남긴 작업
2. calendar 변경: `TASK:k8s 구축 작업(k8s-setup)` → `DONE:k8s 구축 작업(k8s-setup)`
3. README 반영: 작업 히스토리가 아닌 기술 내용만 갱신 (프로젝트 구조, 네임스페이스 `choimory-infra`, 디렉토리 구성, 서브도메인 방식, 수동 배포 흐름)

- done 문서와 README는 작성 후 변경 내용을 먼저 설명하고, 커밋은 하지 않음

---

# DB는 Tailscale 전용 접속

> 정확히는 db는 도메인이 아니라 터널링 없이 그냥 tailscale키면 접속되도록 설정하는 작업을 할거임

## 답변

- DB는 도메인이나 터널링(SSH 터널, port-forward) 없이 Tailscale만 켜면 바로 접속되는 방식으로 다음 작업 계획에 반영

## 방법 비교

| 방법 | 접속 주소 (예) | 장점 | 단점 |
|---|---|---|---|
| A. Tailscale Kubernetes Operator (추천) | `choimory-dev-stg-postgres:5432` | DB Service에 annotation만 붙이면 tailnet에 DB마다 기기가 생김. tailnet에서만 접속. gitops로 관리 | operator 설치와 OAuth 클라이언트 발급 필요 |
| B. 서버의 `tailscale serve --tcp` | `choimory-server:15432` | 설치할 것 없음 | 서버에서 직접 설정해 gitops 밖에 있음. 대상 Service IP가 바뀌면 재설정 필요 |
| C. NodePort | `choimory-server:30432` | 가장 간단. gitops로 관리 | 공유기 내부망에도 열림. k8s 포트는 방화벽으로 막기 어려워 Tailscale 전용 제한이 힘듦 |

- A안이면 DB 4개(stg·prod의 postgres, redis)가 각각 tailnet 기기 이름을 가지며, DataGrip에 SSH 터널 없이 호스트와 포트만 넣으면 됨
- A안의 operator는 앞서 ArgoCD 접속 선택지로 검토한 것과 같음. 인프라 도구는 Cloudflare 도메인으로 열기로 했으므로 operator는 DB 접속 전용. 무료 플랜 기기 수 제한 안에서 충분

## 다음 작업 범위

| 대상 | 접속 방식 |
|---|---|
| front, member-api | Cloudflare Tunnel + 도메인 (공개) |
| ArgoCD 등 인프라 도구 | Cloudflare Tunnel + 도메인 + Cloudflare Access (로그인 관문) |
| PostgreSQL, Redis | Tailscale 전용 (도메인, 터널링 없음) |

- 세부 방식(A·B·C)은 다음 작업의 plan에서 결정
- prod DB는 Tailscale로 접속하더라도 DB 도구에서 읽기 전용 사용 추천
