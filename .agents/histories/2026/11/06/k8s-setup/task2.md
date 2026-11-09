# 목차

- [개요](#개요)
- [2단계 Actions 워크플로 작성](#2단계-actions-워크플로-작성)
  - [답변](#답변)
  - [변경 파일](#변경-파일)
  - [워크플로 구성](#워크플로-구성)
  - [사용한 action 버전](#사용한-action-버전)
  - [검증 결과](#검증-결과)
  - [직접 해야 할 일](#직접-해야-할-일)
  - [순서 주의](#순서-주의)
- [front package-lock.json 커밋 이유](#front-package-lockjson-커밋-이유)
  - [답변](#답변-1)
  - [푼 이유](#푼-이유)
  - [커밋을 안 하면 안 되는 이유](#커밋을-안-하면-안-되는-이유)
  - [비용](#비용)

---

# 개요

- 구현 순서 2단계(앱 레포 Actions 워크플로)를 기록함
- 대상: `choimory-dev-member-api`, `choimory-dev-front`
- 인증 방식과 분석 내용은 [plan5.md](./plan5.md), 1단계 작업은 [task1.md](./task1.md) 참고

---

# 2단계 Actions 워크플로 작성

> 추천대로 다 진행하고 워크플로 작성해줘

## 답변

- plan5의 추천안(CI 테스트 실패 시 배포 중단, front `package-lock.json` 커밋 대상 포함, Spring Boot 정식 버전 변경)을 적용하고 두 워크플로를 작성함
- 커밋은 하지 않음

## 변경 파일

| 레포 | 파일 | 변경 |
| --- | --- | --- |
| member-api | `build.gradle` | Spring Boot `3.5.4-SNAPSHOT` → `3.5.4` |
| member-api | `.github/workflows/deploy.yml` | 새로 작성 |
| front | `.gitignore` | `package-lock.json` 제외 해제 |
| front | `.github/workflows/deploy.yml` | 새로 작성 |

## 워크플로 구성

- 트리거: `stg` push → stg, `main` push → prod
- 같은 브랜치에 연속으로 push되면 이전 실행을 취소함 (`concurrency`)
- `test` job이 성공해야 `deploy` job이 실행됨

| | member-api | front |
| --- | --- | --- |
| 테스트 | `./gradlew test --no-daemon` (JDK 21) | `npm ci`, `npm run lint`, `npm run test:run` (Node 22) |
| 이미지 | `ghcr.io/choimory/choimory-dev-member-api` | `ghcr.io/choimory/choimory-dev-front` |
| 태그 | `<profile>-<sha>` | `<profile>-<sha>` |
| 빌드 인자 | 없음 | `NEXT_PUBLIC_MEMBER_API_BASE_URL` (GitHub Environments `stg`, `prod`의 변수) |
| gitops 대상 | `projects/choimory-dev/member-api/overlays/<profile>` | `projects/choimory-dev/front/overlays/<profile>` |

### deploy job 흐름

1. 앱 레포 체크아웃, GHCR 로그인(`GITHUB_TOKEN`), Buildx 설정
2. 이미지 빌드, push (Actions 레이어 캐시 사용)
3. `choimory-gitops`의 `main` 체크아웃 (Secret `GITOPS_DEPLOY_KEY`)
4. `kustomize edit set image`로 overlay 태그 변경, 커밋, `main`에 push

### 초안 외에 추가한 것

- front 변수 검사: Environment 변수가 비어 있으면 API 주소 없는 이미지가 나가지 않도록 빌드 전에 중단
- push 재시도: 두 앱이 동시에 gitops에 push해 충돌하면 rebase 후 최대 5번 재시도

## 사용한 action 버전

- 작성 시점(2026-11-09)의 최신 릴리스 기준 메이저 태그 사용
- `actions/checkout@v7`, `actions/setup-java@v6`, `actions/setup-node@v7`, `gradle/actions/setup-gradle@v6`, `docker/login-action@v4`, `docker/setup-buildx-action@v4`, `docker/build-push-action@v7`
- kustomize는 GitHub 호스팅 러너(`ubuntu-latest`의 Ubuntu 24.04 기준)에 기본 포함되어 별도 설치하지 않음

## 검증 결과

| 항목 | 결과 |
| --- | --- |
| member-api `./gradlew test` (Spring Boot `3.5.4`) | 통과 (`BUILD SUCCESSFUL`) |
| 두 워크플로 YAML 문법 | 정상 |
| action 메이저 태그 존재 | 확인 |
| 워크플로 실제 실행 | 미검증. GitHub에 올려야 확인 가능 |
| front lint, 테스트 | 로컬에서 실행하지 않음 |

## 직접 해야 할 일

- member-api와 front의 변경을 각각 `dev`에 커밋
  - front의 `package-lock.json`은 작업 후 확인 시 이미 스테이징되어 있었음. Claude가 스테이징한 것은 아니므로 의도한 상태인지 확인 필요
- Deploy Key 2개 생성 ([plan5.md](./plan5.md#순서-member-api-예시) 절차)
  - 공개키: `choimory-gitops`의 Deploy keys에 등록 (`Allow write access` 체크)
  - 개인키: 각 앱 레포의 Actions Secret `GITOPS_DEPLOY_KEY`에 등록
- front의 GitHub Environments에 변수 `NEXT_PUBLIC_MEMBER_API_BASE_URL` 등록

| Environment | 값 |
| --- | --- |
| `stg` | `https://staging-api.choimory.dev/member` |
| `prod` | `https://api.choimory.dev/member` |

## 순서 주의

- 앱 레포의 `stg` 머지는 3단계 gitops가 `main`에 push된 뒤에 진행. 그 전에 머지하면 태그 수정 단계에서 실패함
- 첫 push 후 GHCR 패키지가 public인지 확인 필요

---

# front package-lock.json 커밋 이유

> 프론트의 package lock은 왜 gitignore 풀었어? 이것도 커밋해야해?

## 답변

- 커밋하는 것이 좋음. 앱 프로젝트에서 lock 파일은 빌드할 의존성 버전을 고정하는 파일이라 소스와 함께 버전 관리하는 것이 표준

## 푼 이유

- `package.json`에는 `"tailwindcss": "^4"`, `"typescript": "^5"`, `"vitest": "^5.0.4"`처럼 범위로 적힌 버전이 있음

| | lock 파일 없음 (이전) | lock 파일 커밋 (지금) |
| --- | --- | --- |
| 설치 명령 | `npm install` (Dockerfile 분기) | `npm ci` |
| 받는 버전 | 빌드할 때의 범위 내 최신 | lock에 적힌 정확한 버전 |
| 같은 커밋을 다시 빌드 | 결과가 달라질 수 있음 | 같은 의존성으로 빌드 |
| 위험 | 코드 변경이 없어도 어느 날 빌드나 테스트가 깨지고, 간접 의존성까지 몰래 바뀜 | lock에 없으면 `npm ci`가 오류를 내서 바로 드러남 |

- 자동 배포라서 `stg`에서 확인한 뒤 `main`을 빌드할 때 그 사이에 패키지가 업데이트되면 확인하지 않은 버전이 prod에 나갈 수 있음
- 환경별로 이미지를 따로 빌드하는 구조라 lock 파일이 그 차이를 막아 줌

## 커밋을 안 하면 안 되는 이유

- front 워크플로의 `test` job이 `npm ci`와 `cache: npm`을 씀
- 둘 다 lock 파일이 필요해서 커밋하지 않으면 워크플로가 바로 실패함

## 비용

- 패키지를 추가하거나 올릴 때 `package-lock.json` diff가 커지지만 리뷰에서 접어 두면 됨
- 이후 `npm install`로 lock이 바뀌면 그 변경도 같이 커밋해야 함
