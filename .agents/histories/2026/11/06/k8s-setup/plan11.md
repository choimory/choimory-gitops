# 목차

- [개요](#개요)
- [앱 배포 진행 순서](#앱-배포-진행-순서)
  - [답변](#답변)
  - [순서](#순서)
  - [놓치기 쉬운 것](#놓치기-쉬운-것)

---

# 개요

- 네임스페이스 변경(`infrastructure` → `choimory-infra`)이 끝나 7단계(앱 배포)로 복귀함
- 대상: `choimory-dev-front`, `choimory-dev-member-api`
- 배포는 Actions의 Run workflow 수동 실행. 브랜치 `stg`는 stg 환경, `main`은 prod 환경
- 인프라 도구 접속은 앱 배포와 Cloudflare Tunnel 이후로 미룸 ([plan10.md](./plan10.md#인프라-도구-접속은-이후로-미룸))
- 참고 문서: [plan9.md](./plan9.md#다음-작업-api-도메인-변경과-7단계-진행)(7단계 진행 순서), [task7.md](./task7.md)(deploy.yml 수정, Deploy Key·변수 등록), [task8.md](./task8.md)(네임스페이스 변경)

---

# 앱 배포 진행 순서

> 그럼 이제 프론트랑 api를 stg, prod push한다음에 github actions에서 run workflow만 해주면 되지?

## 답변

- 거의 맞음. 다만 사이에 할 것이 몇 가지 있고, stg를 먼저 확인한 뒤 prod를 실행하는 순서가 좋음

## 순서

| 순서 | 작업 | 확인할 것 |
|---|---|---|
| 1 | 앱 레포 2개 `deploy.yml` 커밋 → `dev` push → `stg`, `main` 머지, push | push마다 test job 성공, deploy job skipped |
| 2 | 두 레포 Actions → deploy → Run workflow → 브랜치 `stg` | test, deploy 성공. gitops `main`에 `github-actions[bot]`의 `deploy: ...` 커밋 생성 |
| 3 | GHCR 패키지 2개를 public으로 전환 (첫 빌드 후 1회만) | 안 하면 `ImagePullBackOff` 계속 |
| 4 | stg 확인 (명령어는 그때 안내) | Pod `Running`, 헬스체크, 회원가입·로그인 |
| 5 | 두 레포 Run workflow → 브랜치 `main` | prod 배포 |
| 6 | prod 확인 | stg와 같은 방식 |

## 놓치기 쉬운 것

- 3번 GHCR public 전환
  - GitHub 프로필 → Packages → `choimory-dev-front`, `choimory-dev-member-api` → Package settings → Change visibility → Public
  - 패키지는 첫 빌드 때 만들어지므로 2번 이후에만 가능
- gitops 로컬 레포 pull
  - 2번부터 Actions가 gitops `main`에 이미지 태그를 커밋함
  - 다음에 gitops를 수정할 때는 먼저 `main`을 pull 받아 `dev`에 반영해야 충돌이 나지 않음
- stg와 prod를 동시에 실행하지 않기
  - 동시에 돌려도 동작은 하지만, stg에서 문제가 있으면 prod까지 같이 깨짐
  - stg 확인 후 prod 실행 추천
- member-api 첫 실행
  - stg 프로필이라 처음 뜰 때 Flyway가 `member` DB에 테이블을 만듦
  - 빈 DB라 문제없을 것으로 보지만, 실패하면 Pod 로그로 확인
