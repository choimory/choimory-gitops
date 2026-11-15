# 목차

- [개요](#개요)
- [run workflow 진행](#run-workflow-진행)
  - [프론트](#프론트)
  - [api](#api)
- [run workflow 실패 원인 분석](#run-workflow-실패-원인-분석)
  - [답변](#답변)
  - [1. front: 이미지는 올라갔고 gitops 커밋 단계에서 실패](#1-front-이미지는-올라갔고-gitops-커밋-단계에서-실패)
  - [2. member-api: 러너에서 Testcontainers가 Docker를 못 찾음](#2-member-api-러너에서-testcontainers가-docker를-못-찾음)
  - [정리](#정리)
- [front 배포 완료](#front-배포-완료)
  - [답변](#답변-1)
  - [확인한 것](#확인한-것)
  - [같이 처리할 것](#같이-처리할-것)
  - [front stg 확인 (서버)](#front-stg-확인-서버)
- [member-api 테스트 실패 수정](#member-api-테스트-실패-수정)
  - [답변](#답변-2)
  - [실패 원인](#실패-원인)
  - [수정 내용](#수정-내용)
  - [검증](#검증)
  - [다음 (직접)](#다음-직접)
- [member-domain 테스트 실패 수정](#member-domain-테스트-실패-수정)
  - [답변](#답변-3)
  - [원인](#원인)
  - [수정 내용](#수정-내용-1)
  - [검증](#검증-1)
  - [다음 (직접)](#다음-직접-1)
- [루트 설정만으로 충분한지](#루트-설정만으로-충분한지)
  - [답변](#답변-4)
  - [동작 방식](#동작-방식)
  - [확인한 근거](#확인한-근거)
  - [루트에 둔 이유](#루트에-둔-이유)
- [ext의 의미](#ext의-의미)
  - [답변](#답변-5)
  - [기본 개념](#기본-개념)
  - [이 값이 버전을 바꾸는 이유](#이-값이-버전을-바꾸는-이유)
  - [이 방식의 장점](#이-방식의-장점)
  - [참고](#참고)
- [member-api stg 빌드 실패 (Maven Central 429)](#member-api-stg-빌드-실패-maven-central-429)
  - [답변](#답변-6)
  - [stg 실패 원인](#stg-실패-원인)
  - [해결 (직접)](#해결-직접)
  - [반복되면 (나중에 개선)](#반복되면-나중에-개선)
  - [참고: prod도 이미 배포가 진행됨](#참고-prod도-이미-배포가-진행됨)
  - [다음](#다음)
- [stg, prod 배포 확인과 stg API 확인](#stg-prod-배포-확인과-stg-api-확인)
  - [답변](#답변-7)
  - [현재 상태](#현재-상태)
  - [stg 1차 확인 (서버)](#stg-1차-확인-서버)
  - [기대 결과](#기대-결과)
- [stg API 확인 결과와 prod 확인](#stg-api-확인-결과와-prod-확인)
  - [답변](#답변-8)
  - [결과](#결과)
  - [front 확인이 무효인 이유](#front-확인이-무효인-이유)
  - [front 다시 확인 (서버)](#front-다시-확인-서버)
  - [prod 확인](#prod-확인)
- [front stg 재확인](#front-stg-재확인)
  - [답변](#답변-9)
  - [결과](#결과-1)
  - [stg 1차 확인 완료](#stg-1차-확인-완료)
  - [다음](#다음-1)
- [prod 확인과 7단계 완료](#prod-확인과-7단계-완료)
  - [답변](#답변-10)
  - [prod 확인 결과](#prod-확인-결과)
  - [7단계 완료 정리](#7단계-완료-정리)
  - [남은 정리 (직접)](#남은-정리-직접)
  - [다음: 2차 Cloudflare Tunnel](#다음-2차-cloudflare-tunnel)

---

# 개요

- 구현 순서 7단계(앱 배포와 1차 확인)를 기록함
- 대상: `choimory-dev-front`, `choimory-dev-member-api`
- 진행 순서와 주의사항은 [plan11.md](./plan11.md) 참고
- 이전 작업은 [task8.md](./task8.md)(네임스페이스 변경) 참고

---

# run workflow 진행

## 프론트

```text
4 workflow runs
deploy
currently running:
deploy
#4:
Manually run
by
choimory
stg
1 minute ago
In progress
Merge branch 'release/1.0.3'
completed successfully:
deploy
#3:
Commit
bba1d79
pushed
by
choimory
main
3 minutes ago
1m 40s
system: github actions 수동으로 변경 (빌드 수동, 테스트 자동)
completed successfully:
deploy
#2:
Commit
99f66e7
pushed
by
choimory
stg
6 minutes ago
1m 44s
system: github actions 수동으로 변경 (빌드 수동, 테스트 자동)
completed successfully:
deploy
#1:
Commit
99f66e7
pushed
by
choimory
dev
6 minutes ago
2m 11s
Footer
```

- workflows/deploy에 이미 세개의 워크플로우가 실행되어 성공한 흔적
- 내가 수동 실행한 deploy는 꽤 오랜시간동안 진행중
- 수동실행한 deploy의 test job은 성공한것 확인되었고, deploy job은 진행중

```text
test summary
Vitest Test Report
Summary
Test Files: ✅ 97 passes · 97 total
Test Results: ✅ 574 passes · 574 total
Job summary generated at run-time
deploy summary
Docker Build summary
For a detailed look at the build, download the following build record archive and import it into Docker Desktop's Builds view.
Build records include details such as timing, dependencies, results, logs, traces, and other information about a build. Learn more

⬇️ choimory~choimory-dev-front~73ZCOY.dockerbuild (55.32 KB - includes 1 build record)

Find this useful? Let us know

ID	Name	Status	Cached	Duration
73ZCOY	choimory-dev-front	✅ completed	0%	2m26s
Build inputs
Job summary generated at run-time
Annotations
4 errors and 2 notices
deploy
The process '/usr/bin/git' failed with exit code 128
deploy
Could not read from remote repository.
deploy
Could not read from remote repository.
deploy
Could not read from remote repository.
test
"The ubuntu-latest label will migrate to Ubuntu 26 beginning October 19, 2026. For more information, see https://github.com/actions/runner-images/issues/14748"
deploy
"The ubuntu-latest label will migrate to Ubuntu 26 beginning October 19, 2026. For more information, see https://github.com/actions/runner-images/issues/14748"
```

```text
35s
Run actions/checkout@v7
Syncing repository: choimory/choimory-gitops
Getting Git version info
Temporarily overriding HOME='/home/runner/work/_temp/<TEMP_ID>' before making global git config changes
Adding repository directory to the temporary git global config as a safe directory
/usr/bin/git config --global --add safe.directory /home/runner/work/choimory-dev-front/choimory-dev-front/gitops
Determining repository object format
Initializing the repository
Disabling automatic garbage collection
Setting up auth
Fetching the repository
  /usr/bin/git -c protocol.version=2 fetch --no-tags --prune --no-recurse-submodules --depth=1 origin +refs/heads/main*:refs/remotes/origin/main* +refs/tags/main*:refs/tags/main*
  Load key "/home/runner/work/_temp/<TEMP_ID>": error in libcrypto
  git@github.com: Permission denied (publickey).
  Error: fatal: Could not read from remote repository.
  
  Please make sure you have the correct access rights
  and the repository exists.
  The process '/usr/bin/git' failed with exit code 128
  Waiting 19 seconds before trying again
  /usr/bin/git -c protocol.version=2 fetch --no-tags --prune --no-recurse-submodules --depth=1 origin +refs/heads/main*:refs/remotes/origin/main* +refs/tags/main*:refs/tags/main*
  Load key "/home/runner/work/_temp/<TEMP_ID>": error in libcrypto
  git@github.com: Permission denied (publickey).
  Error: fatal: Could not read from remote repository.
  
  Please make sure you have the correct access rights
  and the repository exists.
  The process '/usr/bin/git' failed with exit code 128
  Waiting 15 seconds before trying again
  /usr/bin/git -c protocol.version=2 fetch --no-tags --prune --no-recurse-submodules --depth=1 origin +refs/heads/main*:refs/remotes/origin/main* +refs/tags/main*:refs/tags/main*
  Load key "/home/runner/work/_temp/<TEMP_ID>": error in libcrypto
  git@github.com: Permission denied (publickey).
  Error: fatal: Could not read from remote repository.
  
  Please make sure you have the correct access rights
  and the repository exists.
  Error: The process '/usr/bin/git' failed with exit code 128
```

- 프론트의 deploy job도 실패

## api

```text
4 workflow runs
deploy
failed:
deploy
#4:
Manually run
by
choimory
stg
3 minutes ago
43s
Merge branch 'release/2.0.1'
failed:
deploy
#3:
Commit
5545816
pushed
by
choimory
main
4 minutes ago
1m 19s
system: github actions 수동으로 변경 (빌드 수동, 테스트 자동)
failed:
deploy
#2:
Commit
1ed043c
pushed
by
choimory
stg
5 minutes ago
41s
system: github actions 수동으로 변경 (빌드 수동, 테스트 자동)
failed:
deploy
#1:
Commit
1ed043c
pushed
by
choimory
dev
5 minutes ago
1m 21s
```

- workflows/deploy에 이미 세개의 워크플로우가 실행되어 실패한 흔적
- 수동실행한 deploy 워크플로우도 실패함
- deploy 워크플로우의 test job이 실패되었고, deploy는 이로인해 실행되지 않음

```text
test summary
Gradle Builds captured by gradle/actions/setup-gradle
Gradle Root Project	Requested Tasks	Gradle Version	Build Outcome	Build Scan®
choimory-dev-member-api	test	8.14.2 ℹ️	❌	Build Scan not published
ℹ️ Gradle version is out of date — consider upgrading. See Gradle release lifecycle

Gradle State Caching - ⚡ Enhanced (read-only)
Enhanced Caching uses the proprietary gradle-actions-caching provider. See DISTRIBUTION.md for terms of use and opt-out instructions.

Entries: 6 restored (285Mb), 0 saved (0Mb) - Expand for more details
Job summary generated at run-time
Annotations
1 error and 2 notices
test
Process completed with exit code 1.
Out-of-date Gradle version
Gradle 8.14.2 is out of date: consider updating to the latest Gradle version. See https://docs.gradle.org/current/userguide/feature_lifecycle.html#eol_support
test
"The ubuntu-latest label will migrate to Ubuntu 26 beginning October 19, 2026. For more information, see https://github.com/actions/runner-images/issues/14748"
```

- test summary 내용

```text
Run ./gradlew test --no-daemon
To honour the JVM settings for this build a single-use Daemon process will be forked. For more on this, please refer to https://docs.gradle.org/8.14.2/userguide/gradle_daemon.html#sec:disabling_the_daemon in the Gradle documentation.
Daemon will be stopped at the end of the build 
> Task :member-domain:compileJava

Note: /home/runner/work/choimory-dev-member-api/choimory-dev-member-api/member-api/src/main/java/dev/choimory/member/api/config/SecurityConfig.java uses or overrides a deprecated API.
> Task :member-api:compileJava
Note: Recompile with -Xlint:deprecation for details.

> Task :member-api:processResources
> Task :member-api:classes
> Task :member-api:compileTestJava
> Task :member-api:processTestResources NO-SOURCE
> Task :member-api:testClasses
> Task :member-domain:processResources NO-SOURCE
> Task :member-domain:classes
> Task :member-domain:jar

> Task :member-api:test

ChoimoryDevMemberApiApplicationTests > contextLoads() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:180
        Caused by: org.springframework.beans.factory.BeanCreationException at AbstractBeanFactory.java:328
            Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException at ConstructorResolver.java:804
                Caused by: org.springframework.beans.factory.BeanCreationException at AbstractAutowireCapableBeanFactory.java:1826
                    Caused by: org.springframework.beans.factory.BeanCreationException at AbstractAutowireCapableBeanFactory.java:614
                        Caused by: java.lang.IllegalStateException at DockerClientProviderStrategy.java:274

FlywayMigrationTest > initSchemaIsRecordedAsSuccess() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:180
        Caused by: org.springframework.beans.factory.BeanCreationException at AbstractBeanFactory.java:328
            Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException at ConstructorResolver.java:804
                Caused by: org.springframework.beans.factory.BeanCreationException at AbstractAutowireCapableBeanFactory.java:1826
                    Caused by: org.springframework.beans.factory.BeanCreationException at AbstractAutowireCapableBeanFactory.java:614
                        Caused by: java.lang.IllegalStateException at DockerClientProviderStrategy.java:229

FlywayMigrationTest > migrationCreatesAllTables() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

FlywayMigrationTest > migrationInsertsDefaultAuthorities() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

FlywayProfileTest > ProdProfile > flywayIsEnabledAndSqlLogIsOff() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:180
        Caused by: org.springframework.beans.factory.BeanCreationException at AbstractBeanFactory.java:328
            Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException at ConstructorResolver.java:804
                Caused by: org.springframework.beans.factory.BeanCreationException at AbstractAutowireCapableBeanFactory.java:1826
                    Caused by: org.springframework.beans.factory.BeanCreationException at AbstractAutowireCapableBeanFactory.java:614
                        Caused by: java.lang.IllegalStateException at DockerClientProviderStrategy.java:229

FlywayProfileTest > DevProfile > flywayIsEnabled() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:180
        Caused by: org.springframework.beans.factory.BeanCreationException at AbstractBeanFactory.java:328
            Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException at ConstructorResolver.java:804
                Caused by: org.springframework.beans.factory.BeanCreationException at AbstractAutowireCapableBeanFactory.java:1826
                    Caused by: org.springframework.beans.factory.BeanCreationException at AbstractAutowireCapableBeanFactory.java:614
                        Caused by: java.lang.IllegalStateException at DockerClientProviderStrategy.java:229

FlywayProfileTest > LocalProfile > flywayIsEnabled() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

FlywayTimestamptzMigrationTest > initializationError FAILED
    java.lang.IllegalStateException at DockerClientProviderStrategy.java:229

RedisConnectionTest > initializationError FAILED
    java.lang.IllegalStateException at DockerClientProviderStrategy.java:229

JwtRepositoryTest > blacklistSavesWithTimeToLive() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

JwtRepositoryTest > saveSetsTimeToLive() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

JwtRepositoryTest > saveExistsAndDelete() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

MemberJoinTest > nicknameAllowsOnlyKoreanEnglishDigitsWithLength3To15() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:180
        Caused by: org.springframework.beans.factory.BeanCreationException at AbstractBeanFactory.java:328
            Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException at ConstructorResolver.java:804
                Caused by: org.springframework.beans.factory.BeanCreationException at AbstractAutowireCapableBeanFactory.java:1826
                    Caused by: org.springframework.beans.factory.BeanCreationException at AbstractAutowireCapableBeanFactory.java:614
                        Caused by: java.lang.IllegalStateException at DockerClientProviderStrategy.java:229

MemberJoinTest > invalidRequestIsBadRequest() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

MemberJoinTest > duplicateEmailIsConflict() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

MemberJoinTest > duplicateNicknameIsConflict() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

MemberJoinTest > softDeletedEmailAndNicknameCannotBeReused() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

MemberJoinTest > uniqueViolationIsConvertedToConflict() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

MemberJoinTest > joinSavesMemberWithMemberAuthority() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

MemberJoinTest > emailWithoutDomainDotIsBadRequest() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

MemberReissueTest > reissueIsOpenWithoutAuthentication() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:180
        Caused by: org.springframework.beans.factory.BeanCreationException at AbstractBeanFactory.java:328
            Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException at ConstructorResolver.java:804
                Caused by: org.springframework.beans.factory.BeanCreationException at AbstractAutowireCapableBeanFactory.java:1826
                    Caused by: org.springframework.beans.factory.BeanCreationException at AbstractAutowireCapableBeanFactory.java:614
                        Caused by: java.lang.IllegalStateException at DockerClientProviderStrategy.java:229

MemberReissueTest > blankOrMalformedRequestIsBadRequest() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

MemberReissueTest > disabledOrDeletedMemberCannotReissue() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

MemberReissueTest > reissuedAccessTokenReflectsCurrentAuthorities() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

MemberReissueTest > reissueRotatesRefreshToken() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

MemberReissueTest > invalidRefreshTokensAreRejected() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

MemberReissueTest > usedRefreshTokenCannotBeReused() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

CorsTest > unauthorizedResponseHasCorsHeaders() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

CorsTest > preflightFromAllowedOriginIsOk() FAILED
LoginTest > accessTokenFromLoginAuthenticatesRequests() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

LoginTest > wrongPasswordAndUnknownEmailGetSameUnauthorized() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

LogoutTest > invalidRefreshTokensAreUnauthorized() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

LogoutTest > logoutIsIdempotent() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145


92 tests completed, 58 failed
LogoutTest > logoutRevokesRefreshTokenAndBlacklistsAccessToken() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

LogoutTest > logoutWorksWithoutValidAccessToken() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

LogoutTest > otherMembersAccessTokenIsNotBlacklisted() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

LogoutTest > malformedOrBlankRequestIsBadRequest() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

LogoutTest > logoutKeepsOtherDevicesRefreshToken() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

SecurityConfigTest > joinPathOtherMethodsAreNotOpen() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

SecurityConfigTest > unsafeMethodsWithoutAuthenticationAreUnauthorizedNotForbidden() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

SecurityConfigTest > joinPathIsOpenWithoutAuthentication() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

SecurityConfigTest > protectedPathWithoutAuthenticationIsUnauthorized() FAILED
    java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145

> Task :member-api:test FAILED
gradle/actions: Writing build results to /home/runner/work/_temp/.gradle-actions/build-results/__run-1794757041108.json

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':member-api:test'.
> There were failing tests. See the report at: file:///home/runner/work/choimory-dev-member-api/choimory-dev-member-api/member-api/build/reports/tests/test/index.html

* Try:
> Run with --scan to get full insights.

BUILD FAILED in 24s

[Incubating] Problems report is available at: file:///home/runner/work/choimory-dev-member-api/choimory-dev-member-api/build/reports/problems/problems-report.html
6 actionable tasks: 6 executed
Error: Process completed with exit code 1.
```

- test job 상세

---

# run workflow 실패 원인 분석

> task9 다시 확인

## 답변

- 원인은 두 가지. front는 Deploy Key 개인키 등록 문제, member-api는 러너에서 Testcontainers가 Docker를 찾지 못하는 문제

## 1. front: 이미지는 올라갔고 gitops 커밋 단계에서 실패

| 단계 | 결과 |
|---|---|
| test | 성공 (574개 통과) |
| 이미지 빌드, GHCR push | 성공 (`choimory-dev-front` 2m26s) |
| Checkout GitOps | 실패: `Load key ...: error in libcrypto`, `Permission denied (publickey)` |

### 원인

- `error in libcrypto`는 Secret `GITOPS_DEPLOY_KEY`에 저장된 개인키 형식이 깨졌을 때 나는 오류. 권한 문제가 아니라 러너가 키 파일 자체를 읽지 못함
- 흔한 원인
  - 붙여넣을 때 마지막 줄바꿈 누락
  - `-----BEGIN ...-----`, `-----END ...-----` 줄 누락
  - 줄바꿈이 공백으로 바뀜
  - 공개키(`.pub`)를 넣음

### 해결 (직접)

- Mac의 키 파일은 이미 지웠으므로 키를 새로 만들어 다시 등록. member-api도 같은 방식으로 등록했으므로 함께 재등록 추천

```bash
ssh-keygen -t ed25519 -C "choimory-dev-front-actions" -f ./choimory-dev-front-deploy -N ""
ssh-keygen -t ed25519 -C "choimory-dev-member-api-actions" -f ./choimory-dev-member-api-deploy -N ""

# 개인키를 클립보드로 정확히 복사 (붙여넣기 전 한 개씩)
pbcopy < ./choimory-dev-front-deploy
```

1. `choimory-gitops` → Deploy keys에서 기존 키 2개 삭제, 새 `.pub` 2개 등록 (Allow write access 체크)
2. 각 앱 레포 → Repository secret `GITOPS_DEPLOY_KEY` → Update → `pbcopy`로 복사한 개인키 붙여넣기. 첫 줄 `-----BEGIN OPENSSH PRIVATE KEY-----`, 마지막 줄 `-----END OPENSSH PRIVATE KEY-----` 확인
3. 키 파일 4개 삭제

### 지금 바로 할 수 있는 것

- front 이미지가 올라가 GHCR `choimory-dev-front` 패키지가 생김. 지금 public 전환 가능
- 키 재등록 후 front를 `stg`로 다시 Run workflow하면 gitops 커밋까지 진행됨

## 2. member-api: 러너에서 Testcontainers가 Docker를 못 찾음

- 92개 중 58개 실패. 모두 DB나 Redis 컨테이너가 필요한 테스트
- 원인은 모두 `IllegalStateException at DockerClientProviderStrategy`. Testcontainers가 사용할 수 있는 Docker 환경을 찾지 못했다고 판단할 때 나는 오류
- 러너에는 Docker가 있고 로컬(Mac)에서는 같은 테스트가 통과함. 러너의 Docker 버전과 Testcontainers 버전이 맞지 않는 문제로 판단
  - Spring Boot 3.5.4가 관리하는 Testcontainers는 `1.21.3`
  - 최근 Docker 엔진은 오래된 클라이언트의 API 버전 요청을 거부하도록 바뀜
  - Maven Central에는 1.21 계열 `1.21.4`와 `2.0.x`가 있음

### 수정안 (Claude, 허락 필요)

| 수정 | 내용 |
|---|---|
| member-api `build.gradle` | `ext['testcontainers.version'] = '1.21.4'` 추가. Testcontainers만 패치 버전으로 올리고 Spring Boot 버전은 유지 |
| 검증 | 로컬 `./gradlew test` 실행 (허락 필요) 후 push해서 Actions 테스트 확인 |

- `1.21.4`로 해결되는지는 Actions에서 실제로 돌려봐야 확정됨. 안 되면 다음 방법 검토
  - Testcontainers `2.0.x`로 올리기 (모듈 이름이 바뀌어 수정량 증가)
  - 러너에서 Docker API 버전 고정 설정 추가
- 로그의 `92 tests completed` 위쪽에 `client version ... is too old` 같은 Testcontainers 오류 줄이 있으면 원인 확정 가능

## 정리

| 문제 | 해결 | 담당 |
|---|---|---|
| front, member-api Deploy Key 형식 | 키 재생성, 재등록 | 직접 |
| GHCR front 패키지 | 지금 public 전환 가능 | 직접 |
| member-api 테스트 | Testcontainers `1.21.4`로 올림 | Claude (허락 필요) |

---

# front 배포 완료

> 일단 deploy private key github에 추가할때 begin line, end line 빼고 넣었어서, 그거 포함해서 다시 넣었더니 프론트는 배포 완료됐어

## 답변

- Deploy Key 개인키에서 BEGIN/END 줄이 빠진 것이 원인이었고, 다시 등록해 front stg 배포가 gitops까지 반영됨

## 확인한 것

- gitops `main`에 `github-actions[bot]`의 커밋 `117c9a2` (deploy: front stg-99f66e7...) 생성
- `choimory-dev-front/overlays/stg`의 이미지 태그가 `*-bootstrap`에서 실제 값(`stg-99f66e7...`)으로 바뀜
- Deploy Key 인증 → gitops 커밋 → push 전체 흐름이 동작함

## 같이 처리할 것

- member-api Secret도 같은 방식으로 등록했다면 BEGIN/END 줄을 넣어 다시 저장. 지금은 테스트 실패로 deploy job까지 가지 않아 드러나지 않았을 뿐임
- GHCR `choimory-dev-front` 패키지 public 전환. 안 하면 Pod가 `ImagePullBackOff`로 남음
  - GitHub 프로필 → Packages → `choimory-dev-front` → Package settings → Change visibility → Public

## front stg 확인 (서버)

- public 전환 후 1~3분 뒤 실행

```bash
# ArgoCD 반영과 Pod 상태
kubectl -n choimory-infra get applications | grep front
kubectl -n choimory-dev-stg get pods | grep front

# 헬스체크: 서버에서 front에 직접 요청
kubectl -n choimory-dev-stg port-forward svc/choimory-dev-front 3000:3000 &
sleep 3
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:3000/
kill %1
```

| 항목 | 기대 결과 |
|---|---|
| `choimory-dev-stg-front` 앱 | `Synced`, `Healthy` |
| front Pod | 새 Pod `1/1 Running`, 옛 `ImagePullBackOff` Pod는 사라짐 |
| curl | `200` |

- 계속 `ImagePullBackOff`면 아래 결과로 확인

```bash
kubectl -n choimory-dev-stg describe pod -l app=choimory-dev-front | tail -20
```

---

# member-api 테스트 실패 수정

> 일단 api도 배포 성공하도록 진행하자 test 실패 이유가 뭐라고 ?

## 답변

- 실패 원인을 설명하고 member-api 테스트 실패를 수정함. 로컬에서 테스트 92개 모두 통과
- 커밋은 하지 않음

## 실패 원인

- GitHub 러너에는 최신 Docker 엔진이 있고, 이 엔진은 오래된 클라이언트의 접속을 거부함
- 테스트에서 PostgreSQL·Redis 컨테이너를 띄우는 Testcontainers `1.21.3`(Spring Boot 3.5.4가 관리하는 버전)이 그 오래된 클라이언트에 해당
- 그래서 러너에서 "쓸 수 있는 Docker가 없다"(`DockerClientProviderStrategy`)고 판단해 컨테이너가 필요한 테스트 58개가 모두 실패함
- Mac의 Docker(`28.2.2`, 최소 API `1.24`)는 오래된 클라이언트도 받아줘서 로컬에서는 통과했음
- 코드 문제가 아니라 테스트 도구 버전과 러너 환경이 맞지 않는 문제

## 수정 내용

| 파일 | 변경 |
|---|---|
| member-api `member-api/build.gradle` | `ext['testcontainers.version'] = '1.21.4'` 추가, 이유와 삭제 조건을 주석으로 남김 |

- Testcontainers 공식 릴리스 노트에서 `1.21.4`가 최신 Docker 엔진 변경에 대응한 버전임을 확인
- Spring Boot 버전은 그대로 두고 Testcontainers만 같은 1.21 계열 패치 버전으로 올림

## 검증

- `dependencyInsight`로 `1.21.4`가 실제로 선택되는 것을 확인
- 로컬 `./gradlew test` 실행 결과 92개 모두 통과 (실패 0, 오류 0)
- 러너의 최신 Docker에서 통과하는지는 push 후 Actions에서 확인

## 다음 (직접)

1. member-api Secret `GITOPS_DEPLOY_KEY`를 BEGIN/END 줄 포함해 다시 등록. 안 하면 테스트가 통과해도 front 때와 같은 단계에서 실패함
2. 커밋 → `dev` push → release로 `stg`, `main` 머지
3. push된 Actions에서 test job 성공 확인
4. 성공하면 Actions에서 `stg`로 Run workflow 실행
5. 첫 빌드 후 GHCR `choimory-dev-member-api` 패키지 public 전환

---

# member-domain 테스트 실패 수정

> 이번에도 push 자동테스트랑 run workflow 수동 테스트 모두 실패했음
>
> ```
> Run ./gradlew test --no-daemon
> To honour the JVM settings for this build a single-use Daemon process will be forked. For more on this, please refer to https://docs.gradle.org/8.14.2/userguide/gradle_daemon.html#sec:disabling_the_daemon in the Gradle documentation.
> Daemon will be stopped at the end of the build 
> > Task :member-domain:compileJava
>
> > Task :member-api:compileJava
>
> > Task :member-api:processResources
> > Task :member-api:classes
> Note: /home/runner/work/choimory-dev-member-api/choimory-dev-member-api/member-api/src/main/java/dev/choimory/member/api/config/SecurityConfig.java uses or overrides a deprecated API.
> Note: Recompile with -Xlint:deprecation for details.
> > Task :member-api:compileTestJava
> > Task :member-api:processTestResources NO-SOURCE
> > Task :member-api:testClasses
> > Task :member-domain:processResources NO-SOURCE
> > Task :member-domain:classes
> > Task :member-domain:jar
> OpenJDK 64-Bit Server VM warning: Sharing is only supported for boot loader classes because bootstrap classpath has been appended
> > Task :member-api:test
>
> 2026-11-15T15:57:40.698Z  INFO 2521 --- [choimory-dev-member-api] [ionShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
> 2026-11-15T15:57:40.700Z  INFO 2521 --- [choimory-dev-member-api] [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
> 2026-11-15T15:57:40.704Z  INFO 2521 --- [choimory-dev-member-api] [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
> 2026-11-15T15:57:41.753Z  INFO 2521 --- [choimory-dev-member-api] [ionShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
> 2026-11-15T15:57:41.753Z  INFO 2521 --- [choimory-dev-member-api] [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-2 - Shutdown initiated...
> 2026-11-15T15:57:41.757Z  INFO 2521 --- [choimory-dev-member-api] [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-2 - Shutdown completed.
> 2026-11-15T15:57:42.827Z  INFO 2521 --- [choimory-dev-member-api] [ionShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
> 2026-11-15T15:57:42.827Z  INFO 2521 --- [choimory-dev-member-api] [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-3 - Shutdown initiated...
> 2026-11-15T15:57:42.833Z  INFO 2521 --- [choimory-dev-member-api] [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-3 - Shutdown completed.
> 2026-11-15T15:57:43.321Z  INFO 2521 --- [choimory-dev-member-api] [ionShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
> 2026-11-15T15:57:43.322Z  INFO 2521 --- [choimory-dev-member-api] [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-4 - Shutdown initiated...
> 2026-11-15T15:57:43.325Z  INFO 2521 --- [choimory-dev-member-api] [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-4 - Shutdown completed.
> 2026-11-15T15:57:43.917Z  INFO 2521 --- [choimory-dev-member-api] [ionShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
> 2026-11-15T15:57:43.918Z  INFO 2521 --- [choimory-dev-member-api] [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-5 - Shutdown initiated...
> 2026-11-15T15:57:43.921Z  INFO 2521 --- [choimory-dev-member-api] [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-5 - Shutdown completed.
> 2026-11-15T15:57:44.403Z  INFO 2521 --- [choimory-dev-member-api] [ionShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
> 2026-11-15T15:57:44.404Z  INFO 2521 --- [choimory-dev-member-api] [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-6 - Shutdown initiated...
> 2026-11-15T15:57:44.407Z  INFO 2521 --- [choimory-dev-member-api] [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-6 - Shutdown completed.
> Note: /home/runner/work/choimory-dev-member-api/choimory-dev-member-api/member-domain/src/test/java/dev/choimory/member/domain/EntityMappingTest.java uses or overrides a deprecated API.
>
> Note: Recompile with -Xlint:deprecation for details.
> > Task :member-domain:compileTestJava
>
> > Task :member-domain:processTestResources NO-SOURCE
> > Task :member-domain:testClasses
>
> > Task :member-domain:test
>
> EntityMappingTest > entityNullabilityMatchesDdl() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:180
>         Caused by: org.springframework.beans.factory.BeanCreationException at AbstractBeanFactory.java:328
>             Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException at ConstructorResolver.java:804
>                 Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException at ConstructorResolver.java:804
>                     Caused by: org.springframework.beans.factory.BeanCreationException at AbstractAutowireCapableBeanFactory.java:1826
>                         Caused by: org.springframework.beans.factory.BeanCreationException at AbstractAutowireCapableBeanFactory.java:614
>                             Caused by: java.lang.IllegalStateException at DockerClientProviderStrategy.java:274
>
> EntityMappingTest > entityTablesMatchDdlTables() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> EntityMappingTest > entityColumnsMatchDdlColumns() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> AgreementDtosTest > agreementDtoContainsFieldsAndAnswers() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> AgreementDtosTest > softDeletedAnswerIsExcludedFromDto() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> AgreementDtosTest > answerDtoDoesNotContainParent() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> AgreementDtosTest > memberAgreementDtoContainsAnswer() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> AgreementEntitiesTest > memberAgreementIsCascadedFromMemberAndDoesNotRemoveAnswer() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> AgreementEntitiesTest > answersAreRegisteredAndCascaded() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> AgreementEntitiesTest > softDeletedAnswerIsExcludedFromCollection() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> AgreementEntitiesTest > createSetsFields() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> AgreementEntitiesTest > duplicateMemberAgreementIsRejected() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> AuthorityDtosTest > authorityDtoContainsPermissionsAndActions() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> AuthorityDtosTest > menuDtoContainsChildrenButNotParent() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> AuthorityDtosTest > memberAuthorityDtoContainsAuthorityWithoutChildren() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> AuthorityDtosTest > menuFromWithoutChildrenLeavesChildrenNull() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> AuthorityDtosTest > childDtosDoNotContainParent() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> AuthorityEntitiesTest > childMenuIsRegisteredAndCascaded() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> AuthorityEntitiesTest > memberAuthorityIsCascadedFromMemberAndDoesNotRemoveAuthority() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> AuthorityEntitiesTest > menuWithChildrenCannotBeRemoved() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> AuthorityEntitiesTest > permissionAndActionsAreCascadedFromAuthority() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> AuthorityEntitiesTest > authorityIsSavedWithCodeAsString() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> BaseEntityTest > loadedEntityIsNotNew() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:180
>         Caused by: org.springframework.beans.factory.BeanCreationException at AbstractBeanFactory.java:328
>             Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException at ConstructorResolver.java:804
>                 Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException at ConstructorResolver.java:804
>                     Caused by: org.springframework.beans.factory.BeanCreationException at AbstractAutowireCapableBeanFactory.java:1826
>                         Caused by: org.springframework.beans.factory.BeanCreationException at AbstractAutowireCapableBeanFactory.java:614
>                             Caused by: java.lang.IllegalStateException at DockerClientProviderStrategy.java:229
>
> BaseEntityTest > softDeleteSetsDeletedAt() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> BaseEntityTest > softDeletedRowIsExcludedFromJpql() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> BaseEntityTest > entityIsNotNewAfterPersist() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> BaseEntityTest > auditingFillsOnlyCreatedAtOnPersist() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> BaseEntityTest > softDeletedRowIsVisibleToNativeQuery() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> BaseEntityTest > loadedEntityKeepsStoredId() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> BaseEntityTest > modifiedAtIsFilledOnUpdateAndCreatedAtDoesNotChange() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> BaseEntityTest > newEntityHasUuidV7Id() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> BaseEntityTest > jpaConstructorAlsoGeneratesId() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> BaseEntityTest > saveExecutesOnlyInsertWithoutSelect() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> BaseEntityTest > newEntityIsNew() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> BaseEntityTest > idsAreTimeOrdered() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> BaseEntityTest > softDeletedRowIsExcludedFromRepositoryFindAll() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> BaseEntityTest > softDeletedRowIsExcludedFromFindById() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> BaseEntityTest > notDeletedRowIsVisible() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> DdlScriptTest > authorityCodeIsUnique() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:180
>         Caused by: org.springframework.beans.factory.BeanCreationException at AbstractAutowireCapableBeanFactory.java:614
>             Caused by: org.springframework.beans.factory.BeanCreationException at AbstractAutowireCapableBeanFactory.java:614
>                 Caused by: java.lang.IllegalStateException at DockerClientProviderStrategy.java:229
>
> DdlScriptTest > agreementHasNoTypeColumn() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> DdlScriptTest > memberEmailIsUnique() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> DdlScriptTest > deletingReferencedAgreementAnswerIsRestricted() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> DdlScriptTest > deletingAuthorityCascadesToPermissions() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> DdlScriptTest > everyTableHasCommonColumns() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> DdlScriptTest > deletingMenuWithChildrenIsRestricted() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> DdlScriptTest > memberAgreementIsUniquePerMemberAndAnswer() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> DdlScriptTest > ddlIsIdempotent() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> DdlScriptTest > memberNicknameIsUnique() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> DdlScriptTest > permissionActionIsUniquePerPermissionAndAction() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> DdlScriptTest > menuCodeIsUnique() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> DdlScriptTest > permissionIsUniquePerAuthorityAndMenu() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> DdlScriptTest > deletingMemberCascadesToOwnedRows() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> DdlScriptTest > memberAuthorityIsUnique() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> DdlScriptTest > allTablesExist() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> DdlScriptTest > timeColumnsAreTimestamptz() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> DdlScriptTest > deletingMenuReferencedByPermissionIsRestricted() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> DdlScriptTest > followCannotTargetSelf() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> DdlScriptTest > followIsUnique() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> DdlScriptTest > idColumnsAreVarchar36() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> DdlScriptTest > deletingAssignedAuthorityIsRestricted() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> DdlScriptTest > deletingAgreementCascadesToAnswers() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> DdlScriptTest > enumColumnsAreVarchar() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> MemberDtosTest > fromWithoutChildrenLeavesCollectionsNullAndDoesNotInitializeThem() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> MemberDtosTest > fromConvertsLazyCollectionsInsideTransaction() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> MemberDtosTest > followDtoContainsMembersWithoutChildren() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> MemberDtosTest > memberDtoCopiesAuditFields() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> MemberDtosTest > memberDtoCopiesOwnFieldsAndOwnedChildren() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> MemberDtosTest > toBuilderCopiesDtoWithChanges() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> MemberDtosTest > childDtosDoNotContainParent() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> MemberEntitiesTest > updateMethodsChangeFields() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> MemberEntitiesTest > followIsHardDeleted() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> MemberEntitiesTest > removingMemberCascadesToOwnedChildren() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> MemberEntitiesTest > savedMemberIsLoadedWithSameValues() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> MemberEntitiesTest > savedFollowIsLoadedWithMembers() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> MemberEntitiesTest > duplicateFollowIsRejected() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> MemberEntitiesTest > softDeletedImageIsExcludedFromCollection() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> MemberEntitiesTest > memberSuspensionIsRegisteredAndCascaded() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> MemberEntitiesTest > duplicateEmailIsRejected() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> MemberEntitiesTest > createSetsFields() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> MemberEntitiesTest > softDeletedMemberIsExcluded() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> MemberEntitiesTest > selfFollowIsRejectedByDatabaseCheck() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> MemberEntitiesTest > builderDoesNotRegisterToParentUntilAdded() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> MemberEntitiesTest > memberImageIsRegisteredAndCascaded() FAILED
>     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:145
>
> 107 tests completed, 83 failed
>
>
> > Task :member-domain:test FAILED
> gradle/actions: Writing build results to /home/runner/work/_temp/.gradle-actions/build-results/__run-1794758192539.json
>
> FAILURE: Build failed with an exception.
> [Incubating] Problems report is available at: file:///home/runner/work/choimory-dev-member-api/choimory-dev-member-api/build/reports/problems/problems-report.html
> 8 actionable tasks: 8 executed
>
> * What went wrong:
> Execution failed for task ':member-domain:test'.
> > There were failing tests. See the report at: file:///home/runner/work/choimory-dev-member-api/choimory-dev-member-api/member-domain/build/reports/tests/test/index.html
>
> * Try:
> > Run with --scan to get full insights.
>
> BUILD FAILED in 1m 23s
> Error: Process completed with exit code 1.
> ```

## 답변

- 이번 로그에서 member-api 모듈 테스트는 러너에서 통과함. `1.21.4`로 올린 효과가 실제 러너에서 확인됨
- 실패한 것은 member-domain 모듈(107개 중 83개)이며 원인은 같은 `DockerClientProviderStrategy` 오류
- 버전 지정 위치를 루트 `build.gradle`로 옮겨 두 모듈에 함께 적용함. 커밋은 하지 않음

## 원인

- member-domain도 Testcontainers를 쓰는데, 버전 지정을 member-api `build.gradle`에만 넣어 member-domain은 `1.21.3` 그대로였음
- 지난번에는 member-api가 먼저 실패해 member-domain 테스트까지 가지 않아 이번에 드러남

## 수정 내용

| 파일 | 변경 |
|---|---|
| 루트 `build.gradle` | `subprojects` 블록에 `ext['testcontainers.version'] = '1.21.4'`와 주석 추가. 모든 모듈에 적용 |
| `member-api/build.gradle` | 지난번에 넣은 같은 설정 삭제 (루트로 이동) |

- 처음에는 루트 파일 수정이 한 번 반영되지 않아 두 모듈 모두 `1.21.3`인 상태가 있었음. 다시 적용 후 아래처럼 확인

## 검증

| 모듈 | Testcontainers | 로컬 테스트 |
|---|---|---|
| member-api | `1.21.4` | 92개 통과 (실패 0) |
| member-domain | `1.21.4` | 107개 통과 (실패 0) |

- `--rerun-tasks`로 캐시 없이 다시 실행해 확인

## 다음 (직접)

1. 커밋 → `dev` push → release로 `stg`, `main` 머지
2. push된 Actions에서 test job 성공 확인
3. 성공하면 `stg`로 Run workflow 실행
4. 첫 빌드 후 GHCR `choimory-dev-member-api` 패키지 public 전환

- member-api Secret `GITOPS_DEPLOY_KEY`를 BEGIN/END 줄 포함해 다시 등록했는지 확인. 안 되어 있으면 deploy job의 gitops 체크아웃에서 실패함

---

# 루트 설정만으로 충분한지

> 루트말고 양쪽 gradle에 다 해야하는거 아닌가? 루트에만 하면 됨?

## 답변

- 루트에만 하면 됨. 루트 `build.gradle`의 `subprojects { ... }` 블록 안에 있어 양쪽 `build.gradle`에 각각 넣은 것과 같은 효과

## 동작 방식

- `subprojects { ... }`는 모든 하위 모듈(member-api, member-domain)에 같은 설정을 하나씩 적용하는 블록
  - 이미 이 블록으로 `group`, `version`, Java 21 toolchain을 두 모듈에 함께 적용하고 있음
- `ext['testcontainers.version']`도 이 블록 안에 있어 실제로는 각 모듈에 따로 값이 설정됨
- 각 모듈의 의존성 관리 플러그인(`io.spring.dependency-management`)이 자기 모듈의 이 값을 읽어 Testcontainers 버전을 정함

## 확인한 근거

| 모듈 | 선택된 버전 |
|---|---|
| member-api | `1.21.4` |
| member-domain | `1.21.4` |

- 루트 수정이 반영되기 전에는 둘 다 `1.21.3`이었으므로, 루트 설정이 두 모듈에 적용되는 것이 확인됨

## 루트에 둔 이유

- 버전을 한 곳에서만 관리하고, 나중에 지울 때도 한 곳만 지우면 됨
- 모듈이 추가돼도 자동으로 적용됨
- 양쪽 모듈 파일에 각각 적어도 동작은 같지만 두 곳을 함께 관리해야 함. 루트 방식 추천

---

# ext의 의미

> ext[]이건 뭐야?

## 답변

- Gradle 프로젝트에 이름을 붙인 값을 저장하는 공간. 정식 이름은 extra properties

## 기본 개념

```groovy
ext['testcontainers.version'] = '1.21.4'
```

- `ext`는 Gradle 프로젝트마다 하나씩 있는 key-value 저장소
- 위 줄은 이 프로젝트에 `testcontainers.version`이라는 이름으로 `1.21.4`를 저장한다는 뜻
- 이름에 점(`.`)이 들어 있어 `ext.testcontainers.version = ...` 대신 대괄호 문법을 씀. 점으로 쓰면 Gradle이 이름을 나눠서 해석함

## 이 값이 버전을 바꾸는 이유

- `ext`에 값을 넣는다고 저절로 버전이 바뀌는 것이 아니라, Spring 의존성 관리 플러그인이 이 이름을 찾아 읽기 때문에 동작함
  1. Spring Boot의 라이브러리 버전 목록(BOM, `spring-boot-dependencies`)에 `testcontainers.version = 1.21.3` 변수가 있고, Testcontainers 라이브러리들은 이 변수로 버전을 정함
  2. `io.spring.dependency-management` 플러그인은 버전을 정할 때 프로젝트에 같은 이름의 값이 있으면 그 값을 우선함
  3. 그래서 `ext['testcontainers.version'] = '1.21.4'`를 넣으면 BOM의 `1.21.3` 대신 `1.21.4`가 쓰임

```
BOM:     testcontainers.version = 1.21.3
프로젝트: testcontainers.version = 1.21.4   ← 이 값이 우선
결과:    org.testcontainers:* 전부 1.21.4
```

## 이 방식의 장점

- `testImplementation 'org.testcontainers:postgresql:1.21.4'`처럼 라이브러리마다 버전을 적지 않아도 됨
- `junit-jupiter`, `postgresql`, `testcontainers` 본체 등 관련 라이브러리가 한꺼번에 같은 버전으로 맞춰짐
- Spring Boot 공식 문서에서 관리 버전을 바꿀 때 안내하는 방법

## 참고

- `gradle.properties`에 `testcontainers.version=1.21.4`로 적어도 똑같이 동작함
- 이름은 BOM의 변수 이름과 정확히 같아야 하며, 다르면 아무 효과가 없음

---

# member-api stg 빌드 실패 (Maven Central 429)

> ```
> Run docker/build-push-action@v7
> GitHub Actions runtime token ACs
> Docker info
> Proxy configuration
> Buildx version
> Builder info
> /usr/bin/docker buildx build --cache-from type=gha,scope=stg --cache-to type=gha,mode=max,scope=stg --iidfile /home/runner/work/_temp/docker-actions-toolkit-RYp8rq/build-iidfile-6d42331f83.txt --label org.opencontainers.image.source=https://github.com/choimory/choimory-dev-member-api --attest type=provenance,mode=max,builder-id=https://github.com/choimory/choimory-dev-member-api/actions/runs/<RUN_ID>/attempts/1 --tag ghcr.io/choimory/choimory-dev-member-api:stg-ee78d4a2f210e6a18b1dc2f7c8e5fb61396506ce --metadata-file /home/runner/work/_temp/docker-actions-toolkit-RYp8rq/build-metadata-de7676b1ed.json --push .
> #0 building with "builder-<BUILDER_ID>" instance using docker-container driver
>
> #1 [internal] load build definition from Dockerfile
> #1 transferring dockerfile: 1.78kB done
> #1 DONE 0.0s
>
> #2 [auth] library/eclipse-temurin:pull token for registry-1.docker.io
> #2 DONE 0.0s
>
> #3 [internal] load metadata for docker.io/library/eclipse-temurin:21.0.13_9-jdk-noble
> #3 ...
>
> #4 [internal] load metadata for docker.io/library/eclipse-temurin:21.0.13_9-jre-noble
> #4 DONE 0.5s
>
> #3 [internal] load metadata for docker.io/library/eclipse-temurin:21.0.13_9-jdk-noble
> #3 DONE 0.5s
>
> #5 [internal] load .dockerignore
> #5 transferring context: 634B done
> #5 DONE 0.0s
>
> #6 importing cache manifest from gha:6682993889981640469
> #6 DONE 0.0s
>
> #7 [internal] load build context
> #7 transferring context: 535.92kB 0.0s done
> #7 DONE 0.0s
>
> #8 [builder  1/10] FROM docker.io/library/eclipse-temurin:21.0.13_9-jdk-noble@sha256:06c1f157dacf6b68198bd9a4186dd4c34ef24e65d7bbbe2a0cf8e62571ad38c9
> #8 resolve docker.io/library/eclipse-temurin:21.0.13_9-jdk-noble@sha256:06c1f157dacf6b68198bd9a4186dd4c34ef24e65d7bbbe2a0cf8e62571ad38c9 done
> #8 sha256:9c5ce926342e071ca535336de23071b008aaa605d482645bae5c6fc69438cc14 2.46kB / 2.46kB 0.0s done
> #8 sha256:434838bce9699a284575898e53a7f4a9d91d1b4231ef849e18f1f48fcca6a555 156B / 156B 0.0s done
> #8 sha256:a9c23a815aca30918a3a75634adfe592b3e195a18be43414471d6a1917a955ec 12.58MB / 158.42MB 0.2s
> #8 sha256:869bc9612dbb81db3b3831d2e4992a242c2231d03c9935e9ac3197a266c4c6f8 22.95MB / 22.95MB 0.3s done
> #8 sha256:26538842dfbfed820e62531e047bc6cfd3359a7e0a52f59c1a951ccc6ca787fe 13.63MB / 29.79MB 0.2s
> #8 sha256:a9c23a815aca30918a3a75634adfe592b3e195a18be43414471d6a1917a955ec 33.55MB / 158.42MB 0.5s
> #8 sha256:26538842dfbfed820e62531e047bc6cfd3359a7e0a52f59c1a951ccc6ca787fe 29.79MB / 29.79MB 0.3s done
> #8 extracting sha256:26538842dfbfed820e62531e047bc6cfd3359a7e0a52f59c1a951ccc6ca787fe
> #8 sha256:a9c23a815aca30918a3a75634adfe592b3e195a18be43414471d6a1917a955ec 56.62MB / 158.42MB 0.6s
> #8 ...
>
> #9 [stage-1 1/4] FROM docker.io/library/eclipse-temurin:21.0.13_9-jre-noble@sha256:0d57a8e90c2bb844c5a8e1b2b71e11eff06dad7edcfd803cd21ed4ae203b2515
> #9 resolve docker.io/library/eclipse-temurin:21.0.13_9-jre-noble@sha256:0d57a8e90c2bb844c5a8e1b2b71e11eff06dad7edcfd803cd21ed4ae203b2515 done
> #9 sha256:5be1a0be4759630c40b3498a7435e9298a9b0eb9dd301840ef0d4045435982f9 159B / 159B 0.0s done
> #9 sha256:bb7f4827ee3752ab5c37e1891efee826e0d7360c168a346b54fbc7d378f903cb 53.21MB / 53.21MB 0.6s done
> #9 sha256:98af0c3f7e450bcc48d25b4c827fc472d525977f134fed39f289c7c698f7fefd 2.46kB / 2.46kB 0.0s done
> #9 sha256:16d7fe0cce39bf1577f26fa0c6d7ba9911b11fc24c23e6f4bcba9caeeff61563 16.97MB / 16.97MB 0.2s done
> #9 DONE 0.7s
>
> #8 [builder  1/10] FROM docker.io/library/eclipse-temurin:21.0.13_9-jdk-noble@sha256:06c1f157dacf6b68198bd9a4186dd4c34ef24e65d7bbbe2a0cf8e62571ad38c9
> #8 sha256:a9c23a815aca30918a3a75634adfe592b3e195a18be43414471d6a1917a955ec 66.06MB / 158.42MB 0.8s
> #8 sha256:a9c23a815aca30918a3a75634adfe592b3e195a18be43414471d6a1917a955ec 83.89MB / 158.42MB 0.9s
> #8 extracting sha256:26538842dfbfed820e62531e047bc6cfd3359a7e0a52f59c1a951ccc6ca787fe 0.4s done
> #8 extracting sha256:869bc9612dbb81db3b3831d2e4992a242c2231d03c9935e9ac3197a266c4c6f8
> #8 sha256:a9c23a815aca30918a3a75634adfe592b3e195a18be43414471d6a1917a955ec 99.61MB / 158.42MB 1.1s
> #8 sha256:a9c23a815aca30918a3a75634adfe592b3e195a18be43414471d6a1917a955ec 117.44MB / 158.42MB 1.2s
> #8 sha256:a9c23a815aca30918a3a75634adfe592b3e195a18be43414471d6a1917a955ec 133.17MB / 158.42MB 1.4s
> #8 extracting sha256:869bc9612dbb81db3b3831d2e4992a242c2231d03c9935e9ac3197a266c4c6f8 0.5s done
> #8 sha256:a9c23a815aca30918a3a75634adfe592b3e195a18be43414471d6a1917a955ec 147.85MB / 158.42MB 1.5s
> #8 sha256:a9c23a815aca30918a3a75634adfe592b3e195a18be43414471d6a1917a955ec 158.42MB / 158.42MB 1.6s done
> #8 ...
>
> #9 [stage-1 1/4] FROM docker.io/library/eclipse-temurin:21.0.13_9-jre-noble@sha256:0d57a8e90c2bb844c5a8e1b2b71e11eff06dad7edcfd803cd21ed4ae203b2515
> #9 extracting sha256:16d7fe0cce39bf1577f26fa0c6d7ba9911b11fc24c23e6f4bcba9caeeff61563 0.4s done
> #9 extracting sha256:bb7f4827ee3752ab5c37e1891efee826e0d7360c168a346b54fbc7d378f903cb 0.7s done
> #9 DONE 1.7s
>
> #8 [builder  1/10] FROM docker.io/library/eclipse-temurin:21.0.13_9-jdk-noble@sha256:06c1f157dacf6b68198bd9a4186dd4c34ef24e65d7bbbe2a0cf8e62571ad38c9
> #8 extracting sha256:a9c23a815aca30918a3a75634adfe592b3e195a18be43414471d6a1917a955ec
> #8 ...
>
> #9 [stage-1 1/4] FROM docker.io/library/eclipse-temurin:21.0.13_9-jre-noble@sha256:0d57a8e90c2bb844c5a8e1b2b71e11eff06dad7edcfd803cd21ed4ae203b2515
> #9 extracting sha256:5be1a0be4759630c40b3498a7435e9298a9b0eb9dd301840ef0d4045435982f9 done
> #9 extracting sha256:98af0c3f7e450bcc48d25b4c827fc472d525977f134fed39f289c7c698f7fefd done
> #9 DONE 1.7s
>
> #10 [stage-1 2/4] WORKDIR /app
> #10 DONE 0.9s
>
> #8 [builder  1/10] FROM docker.io/library/eclipse-temurin:21.0.13_9-jdk-noble@sha256:06c1f157dacf6b68198bd9a4186dd4c34ef24e65d7bbbe2a0cf8e62571ad38c9
> #8 extracting sha256:a9c23a815aca30918a3a75634adfe592b3e195a18be43414471d6a1917a955ec 1.3s done
> #8 DONE 2.9s
>
> #11 [stage-1 3/4] RUN groupadd --system app && useradd --system --gid app app
> #11 ...
>
> #8 [builder  1/10] FROM docker.io/library/eclipse-temurin:21.0.13_9-jdk-noble@sha256:06c1f157dacf6b68198bd9a4186dd4c34ef24e65d7bbbe2a0cf8e62571ad38c9
> #8 extracting sha256:434838bce9699a284575898e53a7f4a9d91d1b4231ef849e18f1f48fcca6a555 0.1s done
> #8 DONE 3.1s
>
> #8 [builder  1/10] FROM docker.io/library/eclipse-temurin:21.0.13_9-jdk-noble@sha256:06c1f157dacf6b68198bd9a4186dd4c34ef24e65d7bbbe2a0cf8e62571ad38c9
> #8 extracting sha256:9c5ce926342e071ca535336de23071b008aaa605d482645bae5c6fc69438cc14 0.1s done
> #8 DONE 3.2s
>
> #11 [stage-1 3/4] RUN groupadd --system app && useradd --system --gid app app
> #11 ...
>
> #12 [builder  2/10] WORKDIR /workspace
> #12 DONE 0.7s
>
> #11 [stage-1 3/4] RUN groupadd --system app && useradd --system --gid app app
> #11 DONE 1.2s
>
> #13 [builder  3/10] COPY gradlew settings.gradle build.gradle gradle.properties ./
> #13 DONE 0.0s
>
> #14 [builder  4/10] COPY gradle ./gradle
> #14 DONE 0.0s
>
> #15 [builder  5/10] COPY member-domain/build.gradle ./member-domain/
> #15 DONE 0.0s
>
> #16 [builder  6/10] COPY member-api/build.gradle ./member-api/
> #16 DONE 0.0s
>
> #17 [builder  7/10] COPY member-domain/src ./member-domain/src
> #17 DONE 0.0s
>
> #18 [builder  8/10] COPY member-api/src ./member-api/src
> #18 DONE 0.0s
>
> #19 [builder  9/10] COPY docs/flyway ./docs/flyway
> #19 DONE 0.0s
>
> #20 [builder 10/10] RUN --mount=type=cache,target=/root/.gradle     ./gradlew :member-api:bootJar -x test --no-daemon     && cp "$(ls member-api/build/libs/*.jar | grep -v -- '-plain.jar')" app.jar
> #20 0.107 Downloading https://services.gradle.org/distributions/gradle-8.14.2-bin.zip
> #20 0.578 .............10%.............20%.............30%.............40%.............50%.............60%.............70%.............80%.............90%..............100%
> #20 2.585 
> #20 2.585 Welcome to Gradle 8.14.2!
> #20 2.585 
> #20 2.585 Here are the highlights of this release:
> #20 2.585  - Java 24 support
> #20 2.585  - GraalVM Native Image toolchain selection
> #20 2.585  - Enhancements to test reporting
> #20 2.585  - Build Authoring improvements
> #20 2.585 
> #20 2.585 For more details see https://docs.gradle.org/8.14.2/release-notes.html
> #20 2.585 
> #20 2.684 To honour the JVM settings for this build a single-use Daemon process will be forked. For more on this, please refer to https://docs.gradle.org/8.14.2/userguide/gradle_daemon.html#sec:disabling_the_daemon in the Gradle documentation.
> #20 3.385 Daemon will be stopped at the end of the build 
> #20 22.88 
> #20 22.88 FAILURE: Build failed with an exception.
> #20 22.88 
> #20 22.88 * What went wrong:
> #20 22.88 A problem occurred configuring root project 'choimory-dev-member-api'.
> #20 22.88 > Could not resolve all artifacts for configuration 'classpath'.
> #20 22.88    > Could not resolve com.diffplug.durian:durian-core:1.2.0.
> #20 22.88      Required by:
> #20 22.88          root project : > com.diffplug.spotless:com.diffplug.spotless.gradle.plugin:8.10.2 > com.diffplug.spotless:spotless-plugin-gradle:8.10.2
> #20 22.88       > Could not resolve com.diffplug.durian:durian-core:1.2.0.
> #20 22.88          > Could not get resource 'https://plugins.gradle.org/m2/com/diffplug/durian/durian-core/1.2.0/durian-core-1.2.0.pom'.
> #20 22.88             > Could not GET 'https://repo.maven.apache.org/maven2/com/diffplug/durian/durian-core/1.2.0/durian-core-1.2.0.pom'. Received status code 429 from server: Too Many Requests
> #20 22.88    > Could not resolve com.diffplug.durian:durian-io:1.2.0.
> #20 22.88      Required by:
> #20 22.88          root project : > com.diffplug.spotless:com.diffplug.spotless.gradle.plugin:8.10.2 > com.diffplug.spotless:spotless-plugin-gradle:8.10.2
> #20 22.88       > Could not resolve com.diffplug.durian:durian-io:1.2.0.
> #20 22.88          > Could not get resource 'https://plugins.gradle.org/m2/com/diffplug/durian/durian-io/1.2.0/durian-io-1.2.0.pom'.
> #20 22.88             > Could not GET 'https://repo.maven.apache.org/maven2/com/diffplug/durian/durian-io/1.2.0/durian-io-1.2.0.pom'. Received status code 429 from server: Too Many Requests
> #20 22.88    > Could not resolve com.diffplug.durian:durian-collect:1.2.0.
> #20 22.88      Required by:
> #20 22.88          root project : > com.diffplug.spotless:com.diffplug.spotless.gradle.plugin:8.10.2 > com.diffplug.spotless:spotless-plugin-gradle:8.10.2
> #20 22.88       > Could not resolve com.diffplug.durian:durian-collect:1.2.0.
> #20 22.88          > Could not get resource 'https://plugins.gradle.org/m2/com/diffplug/durian/durian-collect/1.2.0/durian-collect-1.2.0.pom'.
> #20 22.88             > Could not GET 'https://repo.maven.apache.org/maven2/com/diffplug/durian/durian-collect/1.2.0/durian-collect-1.2.0.pom'. Received status code 429 from server: Too Many Requests
> #20 22.89    > Could not resolve org.eclipse.jgit:org.eclipse.jgit:7.7.1.202607240634-r.
> #20 22.89      Required by:
> #20 22.89          root project : > com.diffplug.spotless:com.diffplug.spotless.gradle.plugin:8.10.2 > com.diffplug.spotless:spotless-plugin-gradle:8.10.2
> #20 22.89       > Could not resolve org.eclipse.jgit:org.eclipse.jgit:7.7.1.202607240634-r.
> #20 22.89          > Could not get resource 'https://plugins.gradle.org/m2/org/eclipse/jgit/org.eclipse.jgit/7.7.1.202607240634-r/org.eclipse.jgit-7.7.1.202607240634-r.pom'.
> #20 22.89             > Could not GET 'https://repo.maven.apache.org/maven2/org/eclipse/jgit/org.eclipse.jgit/7.7.1.202607240634-r/org.eclipse.jgit-7.7.1.202607240634-r.pom'. Received status code 429 from server: Too Many Requests
> #20 22.89    > Could not resolve com.diffplug.spotless:spotless-lib:4.10.2.
> #20 22.89      Required by:
> #20 22.89          root project : > com.diffplug.spotless:com.diffplug.spotless.gradle.plugin:8.10.2 > com.diffplug.spotless:spotless-plugin-gradle:8.10.2
> #20 22.89       > Could not resolve com.diffplug.spotless:spotless-lib:4.10.2.
> #20 22.89          > Could not get resource 'https://plugins.gradle.org/m2/com/diffplug/spotless/spotless-lib/4.10.2/spotless-lib-4.10.2.pom'.
> #20 22.89             > Could not GET 'https://repo.maven.apache.org/maven2/com/diffplug/spotless/spotless-lib/4.10.2/spotless-lib-4.10.2.pom'. Received status code 429 from server: Too Many Requests
> #20 22.89    > Could not resolve com.diffplug.spotless:spotless-lib-extra:4.10.2.
> #20 22.89      Required by:
> #20 22.89          root project : > com.diffplug.spotless:com.diffplug.spotless.gradle.plugin:8.10.2 > com.diffplug.spotless:spotless-plugin-gradle:8.10.2
> #20 22.89       > Could not resolve com.diffplug.spotless:spotless-lib-extra:4.10.2.
> #20 22.89          > Could not get resource 'https://plugins.gradle.org/m2/com/diffplug/spotless/spotless-lib-extra/4.10.2/spotless-lib-extra-4.10.2.pom'.
> #20 22.89             > Could not GET 'https://repo.maven.apache.org/maven2/com/diffplug/spotless/spotless-lib-extra/4.10.2/spotless-lib-extra-4.10.2.pom'. Received status code 429 from server: Too Many Requests
> #20 22.89    > Could not resolve com.google.guava:guava:33.6.0-jre.
> #20 22.89      Required by:
> #20 22.89          root project : > com.palantir.java-format:com.palantir.java-format.gradle.plugin:2.97.0 > com.palantir.javaformat:gradle-palantir-java-format:2.97.0
> #20 22.89       > Could not resolve com.google.guava:guava:33.6.0-jre.
> #20 22.89          > Could not get resource 'https://plugins.gradle.org/m2/com/google/guava/guava/33.6.0-jre/guava-33.6.0-jre.pom'.
> #20 22.89             > Could not GET 'https://repo.maven.apache.org/maven2/com/google/guava/guava/33.6.0-jre/guava-33.6.0-jre.pom'. Received status code 429 from server: Too Many Requests
> #20 22.89    > Could not resolve com.palantir.javaformat:palantir-java-format-spi:2.97.0.
> #20 22.89      Required by:
> #20 22.89          root project : > com.palantir.java-format:com.palantir.java-format.gradle.plugin:2.97.0 > com.palantir.javaformat:gradle-palantir-java-format:2.97.0
> #20 22.89       > Could not resolve com.palantir.javaformat:palantir-java-format-spi:2.97.0.
> #20 22.89          > Could not get resource 'https://plugins.gradle.org/m2/com/palantir/javaformat/palantir-java-format-spi/2.97.0/palantir-java-format-spi-2.97.0.pom'.
> #20 22.89             > Could not GET 'https://repo.maven.apache.org/maven2/com/palantir/javaformat/palantir-java-format-spi/2.97.0/palantir-java-format-spi-2.97.0.pom'. Received status code 429 from server: Too Many Requests
> #20 22.89    > Could not resolve com.palantir.javaformat:palantir-java-format-jdk-bootstrap:2.97.0.
> #20 22.89      Required by:
> #20 22.89          root project : > com.palantir.java-format:com.palantir.java-format.gradle.plugin:2.97.0 > com.palantir.javaformat:gradle-palantir-java-format:2.97.0
> #20 22.89       > Could not resolve com.palantir.javaformat:palantir-java-format-jdk-bootstrap:2.97.0.
> #20 22.89          > Could not get resource 'https://plugins.gradle.org/m2/com/palantir/javaformat/palantir-java-format-jdk-bootstrap/2.97.0/palantir-java-format-jdk-bootstrap-2.97.0.pom'.
> #20 22.89             > Could not GET 'https://repo.maven.apache.org/maven2/com/palantir/javaformat/palantir-java-format-jdk-bootstrap/2.97.0/palantir-java-format-jdk-bootstrap-2.97.0.pom'. Received status code 429 from server: Too Many Requests
> #20 22.89    > Could not resolve com.palantir.gradle.utils:platform:0.27.0.
> #20 22.89      Required by:
> #20 22.89          root project : > com.palantir.java-format:com.palantir.java-format.gradle.plugin:2.97.0 > com.palantir.javaformat:gradle-palantir-java-format:2.97.0
> #20 22.89       > Could not resolve com.palantir.gradle.utils:platform:0.27.0.
> #20 22.89          > Could not get resource 'https://plugins.gradle.org/m2/com/palantir/gradle/utils/platform/0.27.0/platform-0.27.0.pom'.
> #20 22.89             > Could not GET 'https://repo.maven.apache.org/maven2/com/palantir/gradle/utils/platform/0.27.0/platform-0.27.0.pom'. Received status code 429 from server: Too Many Requests
> #20 22.89    > Could not resolve com.palantir.gradle.idea-configuration:gradle-idea-configuration:0.8.0.
> #20 22.89      Required by:
> #20 22.89          root project : > com.palantir.java-format:com.palantir.java-format.gradle.plugin:2.97.0 > com.palantir.javaformat:gradle-palantir-java-format:2.97.0
> #20 22.89       > Could not resolve com.palantir.gradle.idea-configuration:gradle-idea-configuration:0.8.0.
> #20 22.89          > Could not get resource 'https://plugins.gradle.org/m2/com/palantir/gradle/idea-configuration/gradle-idea-configuration/0.8.0/gradle-idea-configuration-0.8.0.pom'.
> #20 22.89             > Could not GET 'https://repo.maven.apache.org/maven2/com/palantir/gradle/idea-configuration/gradle-idea-configuration/0.8.0/gradle-idea-configuration-0.8.0.pom'. Received status code 429 from server: Too Many Requests
> #20 22.89    > Could not resolve org.springframework.boot:spring-boot-buildpack-platform:3.5.4.
> #20 22.89      Required by:
> #20 22.89          root project : > org.springframework.boot:org.springframework.boot.gradle.plugin:3.5.4 > org.springframework.boot:spring-boot-gradle-plugin:3.5.4
> #20 22.89       > Could not resolve org.springframework.boot:spring-boot-buildpack-platform:3.5.4.
> #20 22.89          > Could not get resource 'https://plugins.gradle.org/m2/org/springframework/boot/spring-boot-buildpack-platform/3.5.4/spring-boot-buildpack-platform-3.5.4.pom'.
> #20 22.89             > Could not GET 'https://repo.maven.apache.org/maven2/org/springframework/boot/spring-boot-buildpack-platform/3.5.4/spring-boot-buildpack-platform-3.5.4.pom'. Received status code 429 from server: Too Many Requests
> #20 22.89    > Could not resolve org.springframework.boot:spring-boot-loader-tools:3.5.4.
> #20 22.89      Required by:
> #20 22.89          root project : > org.springframework.boot:org.springframework.boot.gradle.plugin:3.5.4 > org.springframework.boot:spring-boot-gradle-plugin:3.5.4
> #20 22.89       > Could not resolve org.springframework.boot:spring-boot-loader-tools:3.5.4.
> #20 22.89          > Could not get resource 'https://plugins.gradle.org/m2/org/springframework/boot/spring-boot-loader-tools/3.5.4/spring-boot-loader-tools-3.5.4.pom'.
> #20 22.89             > Could not GET 'https://repo.maven.apache.org/maven2/org/springframework/boot/spring-boot-loader-tools/3.5.4/spring-boot-loader-tools-3.5.4.pom'. Received status code 429 from server: Too Many Requests
> #20 22.89    > Could not resolve org.apache.commons:commons-compress:1.27.1.
> #20 22.89      Required by:
> #20 22.89          root project : > org.springframework.boot:org.springframework.boot.gradle.plugin:3.5.4 > org.springframework.boot:spring-boot-gradle-plugin:3.5.4
> #20 22.89       > Could not resolve org.apache.commons:commons-compress:1.27.1.
> #20 22.89          > Could not get resource 'https://plugins.gradle.org/m2/org/apache/commons/commons-compress/1.27.1/commons-compress-1.27.1.pom'.
> #20 22.89             > Could not GET 'https://repo.maven.apache.org/maven2/org/apache/commons/commons-compress/1.27.1/commons-compress-1.27.1.pom'. Received status code 429 from server: Too Many Requests
> #20 22.89    > Could not resolve org.springframework:spring-core:6.2.9.
> #20 22.89      Required by:
> #20 22.89          root project : > org.springframework.boot:org.springframework.boot.gradle.plugin:3.5.4 > org.springframework.boot:spring-boot-gradle-plugin:3.5.4
> #20 22.89       > Could not resolve org.springframework:spring-core:6.2.9.
> #20 22.89          > Could not get resource 'https://plugins.gradle.org/m2/org/springframework/spring-core/6.2.9/spring-core-6.2.9.pom'.
> #20 22.89             > Could not GET 'https://repo.maven.apache.org/maven2/org/springframework/spring-core/6.2.9/spring-core-6.2.9.pom'. Received status code 429 from server: Too Many Requests
> #20 22.89 
> #20 22.89 * Try:
> #20 22.89 > Run with --stacktrace option to get the stack trace.
> #20 22.89 > Run with --info or --debug option to get more log output.
> #20 22.89 > Run with --scan to get full insights.
> #20 22.89 > Get more help at https://help.gradle.org.
> #20 22.89 
> #20 22.89 BUILD FAILED in 22s
> #20 ERROR: process "/bin/sh -c ./gradlew :member-api:bootJar -x test --no-daemon     && cp \"$(ls member-api/build/libs/*.jar | grep -v -- '-plain.jar')\" app.jar" did not complete successfully: exit code: 1
> ------
>  > [builder 10/10] RUN --mount=type=cache,target=/root/.gradle     ./gradlew :member-api:bootJar -x test --no-daemon     && cp "$(ls member-api/build/libs/*.jar | grep -v -- '-plain.jar')" app.jar:
> 22.89          > Could not get resource 'https://plugins.gradle.org/m2/org/springframework/spring-core/6.2.9/spring-core-6.2.9.pom'.
> 22.89             > Could not GET 'https://repo.maven.apache.org/maven2/org/springframework/spring-core/6.2.9/spring-core-6.2.9.pom'. Received status code 429 from server: Too Many Requests
> 22.89 
> 22.89 * Try:
> 22.89 > Run with --stacktrace option to get the stack trace.
> 22.89 > Run with --info or --debug option to get more log output.
> 22.89 > Run with --scan to get full insights.
> 22.89 > Get more help at https://help.gradle.org.
> 22.89 
> 22.89 BUILD FAILED in 22s
> ------
> Dockerfile:21
> --------------------
>   20 |     # bootJar 결과물 중 plain jar(의존성 미포함)는 제외하고 실행 가능한 jar만 app.jar로 꺼낸다.
>   21 | >>> RUN --mount=type=cache,target=/root/.gradle \
>   22 | >>>     ./gradlew :member-api:bootJar -x test --no-daemon \
>   23 | >>>     && cp "$(ls member-api/build/libs/*.jar | grep -v -- '-plain.jar')" app.jar
>   24 |     
> --------------------
> ERROR: failed to build: failed to solve: process "/bin/sh -c ./gradlew :member-api:bootJar -x test --no-daemon     && cp \"$(ls member-api/build/libs/*.jar | grep -v -- '-plain.jar')\" app.jar" did not complete successfully: exit code: 1
> Reference
> Check build summary support
> Error: buildx failed with: failed to build: failed to solve: process "/bin/sh -c ./gradlew :member-api:bootJar -x test --no-daemon     && cp \"$(ls member-api/build/libs/*.jar | grep -v -- '-plain.jar')\" app.jar" did not complete successfully: exit code: 1
> ```
>
> member-api 수동 빌드가 main은 성공했고 stg는 실패했어

## 답변

- 코드나 설정 문제가 아니라 Maven Central이 일시적으로 요청을 막은 것(`429 Too Many Requests`). 다시 실행하면 됨

## stg 실패 원인

- 이미지 빌드 중 `./gradlew :member-api:bootJar`가 Gradle 플러그인 의존성(spotless, palantir-java-format, spring-boot-gradle-plugin 등)을 받다가 Maven Central(`repo.maven.apache.org`)에서 `429 Too Many Requests`를 받음
- GitHub 러너는 여러 사용자가 같은 IP를 나눠 써서, 그 IP의 요청이 많으면 Maven Central이 잠시 막음
- 같은 코드로 `main` 빌드는 성공했으므로 일시적인 현상

## 해결 (직접)

- 실패한 실행에서 Re-run failed jobs를 누르거나 `stg`로 Run workflow 다시 실행
- 계속 429가 나면 몇 분 뒤 다시 시도

## 반복되면 (나중에 개선)

- 지금 Dockerfile은 이미지를 빌드할 때마다 Gradle 의존성을 전부 새로 받음. `--mount=type=cache` 캐시가 있지만 Actions의 빌더는 매번 새로 만들어져 실행이 끝나면 캐시가 사라짐
- 개선 방법: 러너에서 jar를 먼저 빌드하고(test job처럼 Gradle 캐시 사용), Dockerfile은 jar만 복사하도록 바꾸면 다운로드가 크게 줄어듦
- 자주 반복될 때 진행. 지금은 다시 실행으로 충분

## 참고: prod도 이미 배포가 진행됨

| 커밋 | 내용 |
|---|---|
| `117c9a2` | front stg |
| `0803820` | front prod |
| `ee71f1d` | member-api prod |

- ArgoCD가 이 커밋을 받아 prod에도 새 이미지를 배포 중
- 원래 계획은 stg 확인 후 prod였지만, 외부 공개 전이라 문제없음
- prod도 GHCR 패키지가 public이어야 Pod가 뜸. stg와 prod는 같은 패키지를 쓰므로 한 번만 바꾸면 됨

## 다음

1. member-api `stg` 다시 실행
2. GHCR 패키지 2개(`choimory-dev-front`, `choimory-dev-member-api`) public 전환
3. 서버에서 stg, prod 함께 확인

```bash
kubectl -n choimory-infra get applications | grep -E '(stg|prod)-(front|member-api)'
kubectl -n choimory-dev-stg get pods
kubectl -n choimory-dev-prod get pods
```

---

# stg, prod 배포 확인과 stg API 확인

> stg rerun job 성공했어
>
> ```
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ 
> kubectl -n choimory-infra get applications | grep -E '(stg|prod)-(front|member-api)'
> kubectl -n choimory-dev-stg get pods
> kubectl -n choimory-dev-prod get pods
> choimory-dev-prod-front        Synced        Healthy
> choimory-dev-prod-member-api   Synced        Healthy
> choimory-dev-stg-front         Synced        Healthy
> choimory-dev-stg-member-api    Synced        Healthy
> NAME                                      READY   STATUS    RESTARTS   AGE
> choimory-dev-front-77ffd9c5c5-98xz5       1/1     Running   0          29m
> choimory-dev-member-api-666b96c6d-vtd88   1/1     Running   0          42s
> choimory-dev-postgres-1                   1/1     Running   0          5d3h
> choimory-dev-redis-0                      1/1     Running   0          5d3h
> NAME                                       READY   STATUS    RESTARTS   AGE
> choimory-dev-front-5bddcc6c5f-kngnc        1/1     Running   0          24m
> choimory-dev-member-api-558c9f689d-2jbqk   1/1     Running   0          10m
> choimory-dev-postgres-1                    1/1     Running   0          5d3h
> choimory-dev-redis-0                       1/1     Running   0          5d3h
> ```

## 답변

- member-api stg 재실행 성공. stg, prod 모두 배포 완료
- 다음은 stg에서 API가 실제로 동작하는지 확인

## 현재 상태

| 항목 | 결과 |
|---|---|
| ArgoCD 앱 (stg, prod의 front, member-api) | 4개 모두 `Synced`, `Healthy` |
| Pod | stg, prod 모두 front, member-api, postgres, redis `1/1 Running`, 재시작 0 |
| 이미지 | GHCR에서 정상으로 받아옴 (패키지 public 전환 완료) |
| 자동 배포 흐름 | Actions 태그 커밋 → ArgoCD 반영 → 새 Pod. 전체가 동작함 |

- member-api가 Flyway 마이그레이션까지 마치고 readiness를 통과해야 `Running`, `Healthy`가 되므로 DB 연결과 테이블 생성도 성공한 것으로 봄

## stg 1차 확인 (서버)

- 확인한 API 형식
  - 회원가입 `POST /v1/member/join` (`email`, `password`, `nickname`, `introduce`)
  - 로그인 `POST /v1/login` (`email`, `password`)
  - 재발급 `POST /v1/member/reissue` (`refreshToken`)
  - 로그아웃 `POST /v1/logout` (`refreshToken`)
  - 응답은 `{code, name, message, data}` 형식
  - Redis 키 접두어: `refresh:`, `blacklist:`

```bash
NS=choimory-dev-stg
EMAIL=stgcheck1@example.com
PW='Check1234!'

# member-api 8080(API), 8081(actuator)과 front 3000 연결
kubectl -n $NS port-forward deploy/choimory-dev-member-api 8080:8080 8081:8081 >/dev/null 2>&1 &
kubectl -n $NS port-forward svc/choimory-dev-front 3000:3000 >/dev/null 2>&1 &
sleep 4

echo "== actuator";   curl -s localhost:8081/actuator/health/liveness; echo; curl -s localhost:8081/actuator/health/readiness; echo
echo "== front";      curl -s -o /dev/null -w "%{http_code}\n" localhost:3000/

echo "== join";       curl -s -X POST localhost:8080/v1/member/join -H 'Content-Type: application/json' -d "{\"email\":\"$EMAIL\",\"password\":\"$PW\",\"nickname\":\"stgcheck1\"}"; echo

echo "== login"
LOGIN=$(curl -s -X POST localhost:8080/v1/login -H 'Content-Type: application/json' -d "{\"email\":\"$EMAIL\",\"password\":\"$PW\"}")
echo "$LOGIN" | python3 -c 'import sys,json; d=json.load(sys.stdin); print(d["code"], d["name"], "token:", bool(d["data"] and d["data"]["accessToken"]))'
REFRESH=$(echo "$LOGIN" | python3 -c 'import sys,json; print(json.load(sys.stdin)["data"]["refreshToken"])')

echo "== redis refresh keys"
kubectl -n $NS exec choimory-dev-redis-0 -- sh -c 'redis-cli -a "$REDIS_PASSWORD" --no-auth-warning --scan --pattern "refresh:*" | wc -l'

echo "== reissue"
REISSUE=$(curl -s -X POST localhost:8080/v1/member/reissue -H 'Content-Type: application/json' -d "{\"refreshToken\":\"$REFRESH\"}")
echo "$REISSUE" | python3 -c 'import sys,json; d=json.load(sys.stdin); print(d["code"], d["name"])'
NEW_REFRESH=$(echo "$REISSUE" | python3 -c 'import sys,json; print(json.load(sys.stdin)["data"]["refreshToken"])')

echo "== logout"; curl -s -o /dev/null -w "%{http_code}\n" -X POST localhost:8080/v1/logout -H 'Content-Type: application/json' -d "{\"refreshToken\":\"$NEW_REFRESH\"}"

kill %1 %2
```

## 기대 결과

| 단계 | 기대 결과 |
|---|---|
| actuator | `{"status":"UP"}` 두 번 |
| front | `200` |
| join | `"code":201` |
| login | `200 OK token: True` |
| redis refresh keys | `1` 이상. Redis에 Refresh Token 저장됨 |
| reissue | `200 OK` |
| logout | `200` 또는 `204` |

- 토큰 값은 출력하지 않도록 함
- 테스트 계정(`stgcheck1@example.com`)이 stg DB에 남음. 다시 실행할 때는 `EMAIL`과 닉네임을 바꿔야 중복 오류(409)가 나지 않음
- prod는 stg 결과를 보고 같은 방식으로 진행. prod에는 테스트 계정을 남기지 않는 쪽 추천

---

# stg API 확인 결과와 prod 확인

> ```
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ 
> NS=choimory-dev-stg
> EMAIL=stgcheck1@example.com
> PW='Check1234!'
>
> # member-api 8080(API), 8081(actuator)과 front 3000 연결
> kubectl -n $NS port-forward deploy/choimory-dev-member-api 8080:8080 8081:8081 >/dev/null 2>&1 &
> kubectl -n $NS port-forward svc/choimory-dev-front 3000:3000 >/dev/null 2>&1 &
> sleep 4
>
> echo "== actuator";   curl -s localhost:8081/actuator/health/liveness; echo; curl -s localhost:8081/actuator/health/readiness; echo
> echo "== front";      curl -s -o /dev/null -w "%{http_code}\n" localhost:3000/
>
> echo "== join";       curl -s -X POST localhost:8080/v1/member/join -H 'Content-Type: application/json' -d "{\"email\":\"$EMAIL\",\"password\":\"$PW\",\"nickname\":\"stgcheck1\"}"; echo
>
> echo "== login"
> LOGIN=$(curl -s -X POST localhost:8080/v1/login -H 'Content-Type: application/json' -d "{\"email\":\"$EMAIL\",\"password\":\"$PW\"}")
> echo "$LOGIN" | python3 -c 'import sys,json; d=json.load(sys.stdin); print(d["code"], d["name"], "token:", bool(d["data"] and d["data"]["accessToken"]))'
> REFRESH=$(echo "$LOGIN" | python3 -c 'import sys,json; print(json.load(sys.stdin)["data"]["refreshToken"])')
>
> echo "== redis refresh keys"
> kubectl -n $NS exec choimory-dev-redis-0 -- sh -c 'redis-cli -a "$REDIS_PASSWORD" --no-auth-warning --scan --pattern "refresh:*" | wc -l'
>
> echo "== reissue"
> REISSUE=$(curl -s -X POST localhost:8080/v1/member/reissue -H 'Content-Type: application/json' -d "{\"refreshToken\":\"$REFRESH\"}")
> echo "$REISSUE" | python3 -c 'import sys,json; d=json.load(sys.stdin); print(d["code"], d["name"])'
> NEW_REFRESH=$(echo "$REISSUE" | python3 -c 'import sys,json; print(json.load(sys.stdin)["data"]["refreshToken"])')
>
> echo "== logout"; curl -s -o /dev/null -w "%{http_code}\n" -X POST localhost:8080/v1/logout -H 'Content-Type: application/json' -d "{\"refreshToken\":\"$NEW_REFRESH\"}"
>
> kill %1 %2
> [1] 3282035
> [2] 3282045
> [2]+  Exit 1                     kubectl -n $NS port-forward svc/choimory-dev-front 3000:3000 > /dev/null 2>&1
> == actuator
> {"status":"UP"}
> {"status":"UP"}
> == front
> 200
> == join
> {"code":201,"name":"CREATED","message":"Created","data":{"id":"<MEMBER_ID>","email":"stgcheck1@example.com","nickname":"stgcheck1","status":"ACTIVATE"}}
> == login
> 200 OK token: True
> == redis refresh keys
> 1
> == reissue
> 200 OK
> == logout
> 200
> -bash: kill: %2: no such job
> ```

## 답변

- stg API 흐름(회원가입, 로그인, Redis 저장, 재발급, 로그아웃)은 모두 정상
- front 확인은 무효라 다시 확인. 출력에 front port-forward가 바로 종료된 흔적이 있어 `200`은 front가 아닌 다른 프로그램의 응답으로 판단
- 회원 ID는 `<MEMBER_ID>`로 마스킹함

## 결과

| 단계 | 결과 | 판단 |
|---|---|---|
| actuator liveness, readiness | `UP`, `UP` | 정상 |
| join | `201 CREATED`, `status: ACTIVATE` | 정상. DB 저장 확인 |
| login | `200 OK`, 토큰 발급 | 정상 |
| Redis `refresh:*` 키 | `1` | 정상. Refresh Token이 Redis에 저장됨 |
| reissue | `200 OK` | 정상 |
| logout | `200` | 정상 |
| front | `200` | 확인 무효 |

## front 확인이 무효인 이유

- 출력의 `[2]+ Exit 1 kubectl ... port-forward svc/choimory-dev-front 3000:3000`은 front port-forward가 시작하자마자 실패했다는 뜻. 마지막의 `kill: %2: no such job`도 이미 종료됐기 때문
- 서버의 3000번 포트를 이미 다른 프로그램(4차에서 설치한 Docker 앱 등)이 쓰고 있었을 가능성이 큼
- 그래서 `curl localhost:3000`의 `200`은 그 프로그램의 응답

## front 다시 확인 (서버)

```bash
kubectl -n choimory-dev-stg port-forward svc/choimory-dev-front 13000:3000 >/dev/null 2>&1 &
sleep 3
curl -s -o /dev/null -w "%{http_code}\n" localhost:13000/
curl -s localhost:13000/ | grep -o '<title>[^<]*</title>'
kill %1

# 3000번을 쓰는 프로그램 확인 (참고용)
sudo ss -tlnp | grep ':3000 '
```

- `200`과 front의 `<title>`이 나오면 정상

## prod 확인

- prod에는 테스트 계정을 남기지 않도록 읽기만 하는 확인만 함
  - actuator 상태
  - front 응답
  - 없는 계정으로 로그인 시 `401` (API, DB 조회, 보안 설정이 동작한다는 확인)
- 포트도 겹치지 않게 변경

```bash
NS=choimory-dev-prod
kubectl -n $NS port-forward deploy/choimory-dev-member-api 18080:8080 18081:8081 >/dev/null 2>&1 &
kubectl -n $NS port-forward svc/choimory-dev-front 13001:3000 >/dev/null 2>&1 &
sleep 4

echo "== actuator"; curl -s localhost:18081/actuator/health/liveness; echo; curl -s localhost:18081/actuator/health/readiness; echo
echo "== front";    curl -s -o /dev/null -w "%{http_code}\n" localhost:13001/; curl -s localhost:13001/ | grep -o '<title>[^<]*</title>'
echo "== login (없는 계정)"; curl -s -o /dev/null -w "%{http_code}\n" -X POST localhost:18080/v1/login -H 'Content-Type: application/json' -d '{"email":"nobody@example.com","password":"Nobody1234!"}'

jobs
kill %1 %2
```

| 단계 | 기대 결과 |
|---|---|
| actuator | `UP` 두 번 |
| front | `200`과 `<title>` |
| login (없는 계정) | `401` |
| `jobs` | 두 port-forward 모두 `Running` (중간에 실패하지 않았다는 확인) |

---

# front stg 재확인

> ```
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ 
> kubectl -n choimory-dev-stg port-forward svc/choimory-dev-front 13000:3000 >/dev/null 2>&1 &
> sleep 3
> curl -s -o /dev/null -w "%{http_code}\n" localhost:13000/
> curl -s localhost:13000/ | grep -o '<title>[^<]*</title>'
> kill %1
>
> # 3000번을 쓰는 프로그램 확인 (참고용)
> sudo ss -tlnp | grep ':3000 '
> [1] 3282889
> 200
> <title>choimory.dev</title>
> [sudo: authenticate] Password:       
> LISTEN 0      4096                                  0.0.0.0:3000       0.0.0.0:*    users:(("docker-proxy",pid=3268,fd=8))                    
> LISTEN 0      4096                                     [::]:3000          [::]:*    users:(("docker-proxy",pid=3279,fd=8))                    
> [1]+  Terminated                 kubectl -n choimory-dev-stg port-forward svc/choimory-dev-front 13000:3000 > /dev/null 2>&1
> ```

## 답변

- front stg 정상. 서버 3000번 포트는 `docker-proxy`(Docker Compose 앱)가 쓰고 있었음

## 결과

| 항목 | 결과 | 판단 |
|---|---|---|
| front stg 응답 | `200` | 정상 |
| `<title>` | `choimory.dev` | front 앱이 응답한 것이 맞음 |
| port-forward | 끝까지 유지된 뒤 `kill`로 종료 (`Terminated`) | 중간 실패 없음 |
| 서버 3000번 포트 | `docker-proxy` 사용 중 | Docker Compose 앱 중 하나가 3000번을 공개 중. 앞선 `200`은 이 앱의 응답 |

- 3000번을 쓰는 Docker 앱은 기존 홈서버 앱이라 건드리지 않음
- 앞으로 서버에서 port-forward할 때는 3000, 8080처럼 흔한 포트를 피해 13000, 18080 같은 포트 사용
- 어떤 앱인지는 `docker ps --format '{{.Names}}: {{.Ports}}' | grep 3000`으로 확인 가능

## stg 1차 확인 완료

| 항목 | 상태 |
|---|---|
| Pod, ArgoCD 앱 | 정상 |
| member-api actuator | `UP` |
| front | `200`, 제목 `choimory.dev` |
| 회원가입, 로그인, 재발급, 로그아웃 | 정상 |
| Redis 토큰 저장 | 정상 |
| Actions → gitops 커밋 → ArgoCD 자동 배포 | 정상 |

## 다음

- prod 확인 명령어 실행 (actuator, front, 없는 계정 로그인 `401`)
- prod까지 확인되면 7단계(앱 배포와 1차 확인)를 마무리하고 2차 작업(Cloudflare Tunnel)으로 넘어감

---

# prod 확인과 7단계 완료

> ```
> choimory@choimory-server:/data/dev/choimory/repo/choimory-gitops$ 
> NS=choimory-dev-prod
> kubectl -n $NS port-forward deploy/choimory-dev-member-api 18080:8080 18081:8081 >/dev/null 2>&1 &
> kubectl -n $NS port-forward svc/choimory-dev-front 13001:3000 >/dev/null 2>&1 &
> sleep 4
>
> echo "== actuator"; curl -s localhost:18081/actuator/health/liveness; echo; curl -s localhost:18081/actuator/health/readiness; echo
> echo "== front";    curl -s -o /dev/null -w "%{http_code}\n" localhost:13001/; curl -s localhost:13001/ | grep -o '<title>[^<]*</title>'
> echo "== login (없는 계정)"; curl -s -o /dev/null -w "%{http_code}\n" -X POST localhost:18080/v1/login -H 'Content-Type: application/json' -d '{"email":"nobody@example.com","password":"Nobody1234!"}'
>
> jobs
> kill %1 %2
> [1] 3283222
> [2] 3283232
> == actuator
> {"status":"UP"}
> {"status":"UP"}
> == front
> 200
> <title>choimory.dev</title>
> == login (없는 계정)
> 401
> [1]-  Running                    kubectl -n $NS port-forward deploy/choimory-dev-member-api 18080:8080 18081:8081 > /dev/null 2>&1 &
> [2]+  Running                    kubectl -n $NS port-forward svc/choimory-dev-front 13001:3000 > /dev/null 2>&1 &
> [1]-  Terminated                 kubectl -n $NS port-forward deploy/choimory-dev-member-api 18080:8080 18081:8081 > /dev/null 2>&1
> [2]+  Terminated                 kubectl -n $NS port-forward svc/choimory-dev-front 13001:3000 > /dev/null 2>&1
> ```

## 답변

- prod도 모두 정상. 7단계(앱 배포와 1차 확인) 완료

## prod 확인 결과

| 항목 | 결과 | 판단 |
|---|---|---|
| actuator liveness, readiness | `UP`, `UP` | 정상 |
| front | `200`, `<title>choimory.dev</title>` | 정상 |
| 없는 계정 로그인 | `401` | 정상. API, DB 조회, 보안 설정이 동작함 |
| port-forward | 두 개 모두 `Running`이었다가 `kill`로 종료 | 중간 실패 없음. 응답은 모두 prod 앱의 응답 |

- prod에는 테스트 계정을 만들지 않음

## 7단계 완료 정리

| 항목 | stg | prod |
|---|---|---|
| ArgoCD 앱 | `Synced`, `Healthy` | `Synced`, `Healthy` |
| member-api actuator | `UP` | `UP` |
| front | `200` | `200` |
| API | 회원가입, 로그인, 재발급, 로그아웃, Redis 저장 확인 | 인증 실패 응답 확인 |
| 배포 흐름 | Actions 수동 실행 → GHCR → gitops 커밋 → ArgoCD 자동 배포 | 같음 |

- 7단계 중 고친 것
  - Deploy Key 개인키에 BEGIN/END 줄을 포함해 다시 등록
  - member-api Testcontainers `1.21.3` → `1.21.4` (러너의 최신 Docker 대응)
  - Maven Central `429`는 일시적인 현상이라 다시 실행으로 해결

## 남은 정리 (직접)

- sealed-secrets 키 백업: `~/sealed-secrets-key-backup.yaml`을 서버 밖에 보관하고 서버의 파일 삭제 (task8에서 남은 일)
- stg 테스트 계정: `stgcheck1@example.com`은 stg에 남아 있음. 앞으로 테스트용으로 계속 사용 가능
- gitops 로컬 레포 pull: Actions의 태그 커밋 3개가 gitops `main`에 있으므로, 다음 gitops 수정 전에 `main`을 pull 받아 `dev`에 반영

## 다음: 2차 Cloudflare Tunnel

- plan10에서 정한 순서대로 다음은 Cloudflare Tunnel로 도메인 연결. 계획은 새 plan 문서에서 정리
- 먼저 확인할 것
  1. `choimory.dev` 도메인 구매 여부와 Cloudflare 등록(네임서버 연결) 여부
  2. Cloudflare Zero Trust(Tunnel 관리 화면) 사용 경험
