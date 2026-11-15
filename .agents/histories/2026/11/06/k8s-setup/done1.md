# 목차

- [개요](#개요)
- [작업 마무리](#작업-마무리)
  - [답변](#답변)
  - [목표와 결과](#목표와-결과)
  - [최종 구성](#최종-구성)
  - [단계별 결과](#단계별-결과)
  - [진행 중 바뀐 결정](#진행-중-바뀐-결정)
  - [겪은 문제와 해결](#겪은-문제와-해결)
  - [최종 확인 결과](#최종-확인-결과)
  - [다음 작업으로 넘긴 것](#다음-작업으로-넘긴-것)
  - [직접 정리할 것](#직접-정리할-것)
  - [문서 목록](#문서-목록)

---

# 개요

- k8s 구축 작업(k8s-setup) 전체 요약
- 기간: 2026-11-06 ~ 2026-11-16
- 범위: 홈서버에 kubeadm 클러스터를 설치하고, GitOps(ArgoCD)와 CI(GitHub Actions)로 choimory-dev 앱(front, member-api)을 stg, prod에 배포해 1차 확인까지
- 도메인 연결(Cloudflare Tunnel)과 외부 접속, DB의 Tailscale 접속은 다음 작업으로 넘김

---

# 작업 마무리

> 작업 마무리하자

## 답변

- k8s-setup의 목표였던 클러스터 설치부터 앱 배포와 1차 확인까지 완료
- calendar를 `DONE`으로 변경하고, README에 현재 구성을 반영함

## 목표와 결과

| 목표 | 결과 |
|---|---|
| 실무형 k8s(kubeadm) 클러스터를 홈서버에 Docker Compose 앱과 함께 운영 | 완료. Docker 앱 8개와 공존, 재부팅 복구 확인 |
| GitOps로 클러스터 구성 전체 관리 | 완료. ArgoCD가 `choimory-gitops` `main`을 보고 공용 도구와 프로젝트를 관리 |
| CI/CD로 앱 배포 | 완료. Actions → GHCR → gitops 태그 커밋 → ArgoCD 자동 배포 |
| stg, prod 환경 분리 | 완료. 네임스페이스, overlay, NetworkPolicy로 분리 |
| 1차 확인 (port-forward, curl) | 완료. stg API 전체 흐름, prod 읽기 확인 |

## 최종 구성

| 항목 | 내용 |
|---|---|
| 서버 | Beelink EQR6 (Ryzen 7 7735HS), RAM 26Gi, Ubuntu 26.04.1, 노드 1대 |
| Kubernetes | kubeadm `v1.36.6`, containerd `v2.4.2` (Docker와 공유) |
| CNI | Cilium `1.20.4` (Helm 직접 설치) |
| CD | ArgoCD 차트 `10.17.0`, 자기 자신도 관리 |
| 공용 도구 | 네임스페이스 `choimory-infra`: Sealed Secrets, local-path-provisioner, metrics-server, CloudNativePG 오퍼레이터, Envoy Gateway, 공용 Gateway |
| 프로젝트 | `choimory-dev-stg`, `choimory-dev-prod`: front, member-api, PostgreSQL(CloudNativePG), Redis |
| gitops 디렉토리 | `bootstrap/`, `infrastructure/`, `projects/choimory-dev/{argocd,namespaces,postgres,redis,choimory-dev-front,choimory-dev-member-api}` |
| 도메인 (HTTPRoute) | `choimory.dev`, `staging.choimory.dev`, `member-api.choimory.dev`, `staging-member-api.choimory.dev` |
| 배포 | push는 테스트만 자동. 빌드·배포는 Actions Run workflow 수동 실행 (`stg`, `main`만) |
| 비밀값 | Sealed Secrets (Redis 비밀번호, JWT 키) |

- 상세 구성은 [README.md](../../../../../../README.md) 참고

## 단계별 결과

| 단계 | 내용 | 결과 | 기록 |
|---|---|---|---|
| 분석 | 스펙, 개발 방향, 네임스페이스·디렉토리·자원, 구현 순서 | 확정 | plan1~plan4 |
| 1 | member-api stg 프로필, actuator | 완료 | task1, task3 |
| 2 | 앱 레포 Actions 워크플로 | 완료 | plan5, task2 |
| 3 | gitops 작성 | 완료 | plan6, task3 |
| 4 | 서버 k8s 설치 (노드 준비, kubeadm, Cilium) | 완료 | plan7, task4 |
| 5 | ArgoCD 설치, root 등록, 공용 도구 동기화 | 완료 | task5 |
| 6 | SealedSecret | 완료 | task5, task6 |
| 중간 | 디렉토리 구성 변경, API 도메인 변경, Actions 수동 배포 | 완료 | plan9, task7 |
| 중간 | 공용 네임스페이스 `infrastructure` → `choimory-infra` | 완료 | plan10, task8 |
| 7 | 앱 배포와 1차 확인 | 완료 | plan11, task9 |
| 마무리 | 남은 작업 정리, 범위 분리 | 완료 | plan12 |

## 진행 중 바뀐 결정

| 항목 | 처음 | 최종 | 이유 |
|---|---|---|---|
| 클러스터 | k3s | kubeadm | 실무 k8s 경험 목적 |
| 서버 RAM 기준 | 32GB | 26Gi | `free -h` 실제 값 |
| 공용 네임스페이스 | `infrastructure` | `choimory-infra` | 이름 규칙 통일 |
| gitops 디렉토리 | `front`, `member-api`, `data` | `choimory-dev-front`, `choimory-dev-member-api`, `postgres`, `redis` | 프로젝트 접두어, DB·Redis 분리 |
| API 도메인 | 경로 방식 `api.choimory.dev/member` | 서비스별 서브도메인 | 앱 경로(`/v1/member`)와 서비스 이름 중복 |
| 배포 | push 시 자동 | 테스트만 자동, 배포는 수동 실행 | 배포 시점을 직접 통제 |
| Testcontainers | `1.21.3` (Spring Boot 관리) | `1.21.4` | 러너의 최신 Docker 대응 |

## 겪은 문제와 해결

| 문제 | 원인 | 해결 |
|---|---|---|
| ArgoCD `gateway` Degraded | GatewayClass가 EnvoyProxy보다 먼저 처리됨 | EnvoyProxy에 `sync-wave: "-1"` |
| wave 7 미생성 | root 동기화가 재시도를 모두 소진 | `operation` patch로 수동 동기화 |
| SealedSecret 일부만 반영 | 앱마다 Git을 다시 읽는 시점이 다름 | 해당 앱 hard refresh |
| Actions gitops 체크아웃 실패 (`error in libcrypto`) | Deploy Key 개인키의 BEGIN/END 줄 누락 | 줄을 포함해 재등록 |
| member-api 테스트 전체 실패 | 러너의 최신 Docker에 Testcontainers 1.21.3이 접속 못함 | 루트 `build.gradle` `subprojects`에 `ext['testcontainers.version'] = '1.21.4'` |
| member-api 이미지 빌드 실패 | Maven Central `429 Too Many Requests` | 재실행 |
| front port-forward 확인 오판 | 서버 3000번을 Docker 앱이 사용 중 | 13000 등 겹치지 않는 포트 사용 |
| 네임스페이스 변경 위험 | ArgoCD prune, sealed-secrets 키, finalizer, metrics APIService, Helm CRD 소유권 | 키 재백업 → 컨트롤러 정지 → finalizer 제거 → 순서대로 삭제·재설치 |

## 최종 확인 결과

| 항목 | stg | prod |
|---|---|---|
| ArgoCD 앱 | `Synced`, `Healthy` | `Synced`, `Healthy` |
| member-api actuator | `UP` | `UP` |
| front | `200`, 제목 `choimory.dev` | `200`, 제목 `choimory.dev` |
| API | 회원가입, 로그인, 재발급, 로그아웃, Redis 토큰 저장 | 없는 계정 로그인 `401` |

- 메모리: k8s Pod 합계 약 3.4Gi (예상 3.5~4Gi 안), 서버 여유 20Gi
- member-api stg는 상한 512Mi 중 약 72% 사용. 외부 공개 후 지켜볼 것

## 다음 작업으로 넘긴 것

| 작업 | 내용 |
|---|---|
| 도메인 작업 (다음) | Cloudflare Tunnel과 앱 도메인, 브라우저 흐름(CORS, 쿠키) 확인, 인프라 도구 도메인과 Cloudflare Access, ArgoCD 계정 설정(초기 비밀번호 변경 후 `argocd-initial-admin-secret` 삭제), DB를 Tailscale만 켜면 접속되도록 설정 |
| 모니터링 설치 | Grafana, Prometheus. 도메인은 도메인 작업의 방식 재사용 |
| 백업 (DAS 증설 후) | Docker Compose 앱, PostgreSQL(CloudNativePG), etcd·pki |
| 범위 결정 | 배포 실패 알림, HPA 테스트, Uptime Kuma·Homepage 연동 |
| 개선 후보 | member-api Dockerfile 의존성 다운로드(Maven 429 반복 시), Spring Boot 버전업 후 Testcontainers `ext` 삭제, Gradle 버전, 러너 이미지 변경 대응 |

## 직접 정리할 것

- sealed-secrets 키 백업(`~/sealed-secrets-key-backup.yaml`, 개인키 원본)을 서버 밖에 보관하고 서버의 파일 삭제. 블로그 자료로 쓰지 않음
- 서버 `~/sealed`는 블로그 참고용으로 유지 (암호화된 YAML만 있음, 블로그에는 `encryptedData` 생략)
- gitops 로컬 레포에서 `main`을 받아 `dev`에 반영 (Actions의 태그 커밋)
- 백업 전까지 prod DB에 실제 사용자 데이터를 두지 않음

## 문서 목록

| 문서 | 내용 |
|---|---|
| [plan1.md](./plan1.md) | 스펙 결정 (k8s, CI, CD, CR) |
| [plan2.md](./plan2.md) | 개발 방향 (환경, 도메인, Gateway, DB·Redis) |
| [plan3.md](./plan3.md) | 네임스페이스, 디렉토리, 데이터 경로, 자원 |
| [plan4.md](./plan4.md) | 구현 순서 |
| [plan5.md](./plan5.md) | gitops 커밋 인증(Deploy Key), Actions 분석 |
| [plan6.md](./plan6.md) | gitops 구성안, actuator |
| [plan7.md](./plan7.md) | 서버 설치 명령어, Cilium |
| [plan8.md](./plan8.md) | 5단계 이후 남은 작업 |
| [plan9.md](./plan9.md) | 7단계 전 결정 (디렉토리, Actions 수동, API 도메인, GitHub secret·variable) |
| [plan10.md](./plan10.md) | 네임스페이스 변경 계획, 인프라 도구 접속 연기 |
| [plan11.md](./plan11.md) | 앱 배포 순서 |
| [plan12.md](./plan12.md) | 남은 작업과 작업 단위 분리 |
| [task1.md](./task1.md) ~ [task3.md](./task3.md) | 1~3단계 작성 |
| [task4.md](./task4.md) | 서버 k8s 설치 실행 기록 |
| [task5.md](./task5.md) | ArgoCD 설치, SealedSecret 생성 |
| [task6.md](./task6.md) | 6단계 반영 확인 |
| [task7.md](./task7.md) | 디렉토리·API 도메인 변경, Actions 수정, Deploy Key 등록 |
| [task8.md](./task8.md) | 네임스페이스 변경 실행 기록 |
| [task9.md](./task9.md) | 앱 배포와 1차 확인 |
