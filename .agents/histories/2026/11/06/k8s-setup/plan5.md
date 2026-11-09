# 목차

- [개요](#개요)
- [gitops 커밋 인증 방식 비교](#gitops-커밋-인증-방식-비교)
  - [답변](#답변)
  - [이전 설명 정정](#이전-설명-정정)
  - [인증 방식 비교](#인증-방식-비교)
  - [공통 사항](#공통-사항)
  - [추천: 앱 레포별 Deploy Key](#추천-앱-레포별-deploy-key)
  - [결정 필요 사항](#결정-필요-사항)
- [Deploy Key 생성과 등록 위치](#deploy-key-생성과-등록-위치)
  - [답변](#답변-1)
  - [키가 놓이는 곳](#키가-놓이는-곳)
  - [순서 (member-api 예시)](#순서-member-api-예시)
  - [홈서버가 gitops 레포를 읽을 때](#홈서버가-gitops-레포를-읽을-때)
- [Deploy Key 보관 위치](#deploy-key-보관-위치)
  - [답변](#답변-2)
  - [최종 위치](#최종-위치)
  - [재발급](#재발급)
- [Deploy Key 방식 확정](#deploy-key-방식-확정)
  - [답변](#답변-3)
- [2단계 Actions 워크플로 분석](#2단계-actions-워크플로-분석)
  - [답변](#답변-4)
  - [작성할 워크플로](#작성할-워크플로)
  - [해결이 필요한 것](#해결이-필요한-것)
  - [순서 주의](#순서-주의)
  - [3단계로 넘길 내용](#3단계로-넘길-내용)
  - [결정 필요 사항](#결정-필요-사항-1)

---

# 개요

- 구현 순서 2단계(앱 레포 Actions 워크플로)에서 Actions가 gitops 레포의 overlay 이미지 태그를 수정해 커밋해야 함
- 기본 제공되는 `GITHUB_TOKEN`은 워크플로가 도는 레포에만 쓸 수 있어 별도 인증이 필요함
- 이 문서는 gitops 레포 커밋 인증 방식을 비교함
- 참고 문서: [plan4.md](./plan4.md)(구현 순서), [task1.md](./task1.md)(1단계 작업)

---

# gitops 커밋 인증 방식 비교

> 인증방식 비교 설명

## 답변

- Deploy Key, Fine-grained PAT, GitHub App을 비교함
- 기존 추천(Fine-grained PAT)을 앱 레포별 Deploy Key로 변경함

## 이전 설명 정정

- 앞에서 Deploy Key는 SSH 설정이 번거롭다고 설명했으나 과장된 설명이었음
- `actions/checkout`의 `ssh-key` 옵션 한 줄이면 됨

## 인증 방식 비교

| | Deploy Key | Fine-grained PAT | GitHub App |
| --- | --- | --- | --- |
| 정체 | gitops 레포에 등록하는 SSH 키 | 내 계정으로 발급하는 토큰 | 직접 만든 앱이 발급받는 임시 토큰 |
| 접근 범위 | 등록한 레포 하나 | 선택한 레포, 권한별 지정 | 설치한 레포, 권한별 지정 |
| 유출 시 피해 | gitops 레포 쓰기 | gitops 레포 쓰기 | 앱 개인키가 유출되면 gitops 레포 쓰기 |
| 만료 | 없음 | 만료일을 두고 갱신하는 방식 | 토큰은 약 1시간, 앱 개인키는 만료 없음 |
| 관리할 것 | 앱 레포마다 키 Secret | 토큰 Secret, 주기적 갱신 | App ID와 개인키 Secret, 앱 생성·설치 |
| 커밋 작성자 | 워크플로에서 지정 | 워크플로에서 지정 (push 주체는 내 계정) | 앱 봇으로 표시 가능 |
| 초기 설정 | 키 생성 후 등록 | 웹에서 발급 | 가장 많음 |
| 주 단점 | 키 교체는 수동 | 만료되면 배포가 멈춤 | 혼자 쓰기엔 절차가 많음 |

## 공통 사항

- 셋 다 gitops 레포에 쓸 수 있으면 사실상 배포 권한을 가짐. ArgoCD가 그 레포 내용을 그대로 클러스터에 반영하기 때문
- 레포가 public이어도 Secret은 포크 PR 워크플로에 전달되지 않으므로, `push` 트리거만 쓰면 노출 경로가 늘지 않음

## 추천: 앱 레포별 Deploy Key

- 범위: 키가 gitops 레포 하나에만 묶여 있어 유출돼도 다른 레포나 계정에는 영향이 없음
- 만료 없음: PAT처럼 만료되어 배포가 멈출 일이 없음
- 개별 폐기: member-api와 front에 키를 따로 주면 한쪽만 폐기 가능
- 비용: 프로젝트가 늘 때마다 키를 하나 더 만들어 등록해야 함 (몇 분 정도의 작업)
- 앱 레포가 많아지거나 실무 방식을 익혀 보고 싶어지면 GitHub App으로 전환 가능. 워크플로에서는 인증 부분만 바뀜

## 결정 필요 사항

- [ ] gitops 레포 커밋 인증 방식: 앱 레포별 Deploy Key로 할지

---

# Deploy Key 생성과 등록 위치

> deploy key는 서버에서 생성한 ssh를 서버에 등록하고 리포에도 저장하는건가?

## 답변

- 아니요. 서버는 관여하지 않음
- 이 키는 GitHub Actions가 gitops 레포에 push할 때만 쓰고, 홈서버에는 저장하지 않음
- 키 생성은 맥북 같은 아무 PC에서 한 번만 하면 됨

## 키가 놓이는 곳

| 키 | 저장 위치 | 역할 |
| --- | --- | --- |
| 공개키 | gitops 레포 → Settings → Deploy keys (`Allow write access` 체크) | 자물쇠 |
| 개인키 | 앱 레포 → Settings → Secrets → Actions | 열쇠. 워크플로가 꺼내 씀 |

- 레포의 파일로 커밋하는 것이 아니라 둘 다 GitHub 설정 화면에 등록함
- 개인키가 파일로 커밋되면 public 레포라 바로 유출됨

## 순서 (member-api 예시)

```bash
# 1. 맥북에서 키 생성 (패스프레이즈 없이)
ssh-keygen -t ed25519 -C "member-api-actions" -f ./member-api-deploy -N ""
```

2. `member-api-deploy.pub` 내용을 `choimory-gitops`의 Deploy keys에 등록
3. `member-api-deploy` 내용을 `choimory-dev-member-api`의 Actions Secret(예: `GITOPS_DEPLOY_KEY`)에 등록
4. 맥북에 남은 키 파일 두 개를 삭제. 다시 필요하면 새로 만들면 됨

- front도 같은 방식으로 키를 하나 더 만듦

## 홈서버가 gitops 레포를 읽을 때

- 별개의 문제이며 gitops 레포가 public이면 ArgoCD는 인증 없이 읽을 수 있음
- private이면 ArgoCD용 읽기 전용 Deploy Key를 따로 만들고, 그때는 개인키를 클러스터에 등록해야 함
- [ ] `choimory-gitops` 레포의 public 여부 확인

---

# Deploy Key 보관 위치

> ssh 키 만든다음에 github 리포에만 가지고 있는건가?

## 답변

- 네. GitHub에만 있으면 되며, 등록을 마치면 맥북의 키 파일은 지워도 됨

## 최종 위치

| 키 | 최종 위치 |
| --- | --- |
| 공개키 | `choimory-gitops` 레포의 Deploy keys |
| 개인키 | 앱 레포의 Actions Secret |
| 맥북, 홈서버 | 없음 |

- 워크플로가 돌 때 GitHub가 Secret의 개인키를 러너에 넘겨 주고, 러너는 작업이 끝나면 사라짐

## 재발급

- 등록한 Secret은 GitHub 화면에서 다시 볼 수 없지만 백업할 필요는 없음
- 키를 잃어버리거나 유출이 의심되면 새 키를 만들어 Deploy key와 Secret을 교체하고, 예전 Deploy key를 삭제

---

# Deploy Key 방식 확정

> deploy key 방식으로 가자

## 답변

- [x] gitops 레포 커밋 인증 방식: 앱 레포별 Deploy Key

---

# 2단계 Actions 워크플로 분석

> public임, 2단계 분석 시작하자

## 답변

- `choimory-gitops`가 public이므로 ArgoCD용 Deploy Key는 필요 없음
- 2단계 분석을 완료했으며 파일은 아직 수정하지 않음
- 두 앱 레포 모두 `.github/workflows`가 없어 새로 작성하면 됨
- member-api의 1단계 변경은 `47f9811`로 커밋됐고 아직 push 전임

## 작성할 워크플로

- 파일은 레포마다 `.github/workflows/deploy.yml` 하나
- 아래는 front 기준 초안이며, action 버전은 작성할 때 확인해서 정함

```yaml
name: deploy
on:
  push:
    branches: [stg, main]

concurrency:
  group: deploy-${{ github.ref_name }}
  cancel-in-progress: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ github.ref_name == 'main' && 'prod' || 'stg' }}
    permissions:
      contents: read
      packages: write
    env:
      PROFILE: ${{ github.ref_name == 'main' && 'prod' || 'stg' }}
      IMAGE: ghcr.io/choimory/choimory-dev-front
    steps:
      # 1. 코드 체크아웃, 테스트
      # 2. GHCR 로그인 (GITHUB_TOKEN)
      # 3. 이미지 빌드, push: $IMAGE:$PROFILE-$GITHUB_SHA
      #    build-args: NEXT_PUBLIC_MEMBER_API_BASE_URL=${{ vars.NEXT_PUBLIC_MEMBER_API_BASE_URL }}
      # 4. choimory-gitops 체크아웃 (ssh-key: ${{ secrets.GITOPS_DEPLOY_KEY }})
      # 5. projects/choimory-dev/front/overlays/$PROFILE 에서 kustomize edit set image
      # 6. 커밋, main에 push (충돌 시 pull --rebase 후 재시도)
```

| | member-api | front |
| --- | --- | --- |
| 이미지 | `ghcr.io/choimory/choimory-dev-member-api` | `ghcr.io/choimory/choimory-dev-front` |
| 테스트 | `./gradlew test` (Testcontainers) | `npm run lint`, `npm run test:run` |
| 빌드 인자 | 없음 | stg `https://staging-api.choimory.dev/member`, prod `https://api.choimory.dev/member` |

## 해결이 필요한 것

- front 미커밋: `Dockerfile`, `.dockerignore`, `next.config.ts`의 `output: "standalone"`이 아직 커밋되지 않음. 이 셋이 있어야 빌드됨
- front `package-lock.json`이 git 제외 대상: CI에서 매번 `npm install`로 `^` 범위의 최신 버전을 받게 되어 같은 커밋도 빌드 결과가 달라질 수 있음. `.gitignore`에서 빼고 커밋하는 것을 추천
- member-api의 Spring Boot `3.5.4-SNAPSHOT`: 저장소에 아직 있어 지금 빌드는 되지만, SNAPSHOT은 내용이 바뀌거나 삭제될 수 있음. 정식 `3.5.4`가 Gradle Plugin Portal에 있으므로 변경 추천
- gitops 레포 브랜치: 지금 작업 브랜치는 `dev`인데 워크플로와 ArgoCD는 계획대로 `main`을 봄. 3단계 결과물이 `main`에 올라가야 함

## 순서 주의

- 워크플로는 `stg`, `main` 브랜치에 파일이 있어야 동작함. `dev`에 커밋해도 바로 돌지 않음
- `stg`로 머지하는 것은 3단계 gitops가 `main`에 push된 뒤여야 함. overlay가 없으면 태그 수정 단계에서 실패함

## 3단계로 넘길 내용

- member-api에 actuator가 없어 probe는 TCP로 하거나 actuator를 추가해야 함
- `${redis-username}`은 기본값이 없어 값이 비어 있어도 환경변수 자체는 넣어야 함
- `spring-boot-starter-data-elasticsearch`가 포함되어 있는데 클러스터에는 Elasticsearch가 없음. 기동에 영향이 있는지 7단계에서 확인 필요

## 결정 필요 사항

- [x] `choimory-gitops` 레포: public
- [ ] 테스트: 배포 전 CI에서 테스트를 돌려 실패하면 배포를 막을지 (추천)
- [ ] front `package-lock.json` 커밋 여부
- [ ] member-api Spring Boot `3.5.4` 정식 버전 변경 여부 (변경 시 `./gradlew test`로 검증)
- [ ] 두 워크플로 파일 작성 허락
