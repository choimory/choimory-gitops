# 목차

- [개요](#개요)
- [Gateway 이름 변경](#gateway-이름-변경)
  - [답변](#답변)
  - [변경 파일](#변경-파일)
  - [검증](#검증)
  - [push 후 확인](#push-후-확인)
  - [다음 작업](#다음-작업)

---

# 개요

- plan1에서 확정한 이름 변경에 따라 도메인 연동 구현을 시작함
- 이 문서는 공용 Gateway 이름 변경(`shared-gateway` → `gateway`)을 기록함
- calendar 상태를 `TASK:도메인 연동 작업(domain-connect)`으로 변경함
- 참고 문서: [plan1.md](./plan1.md)(이름 변경 범위 확정)

---

# Gateway 이름 변경

> 응 작업 시작해줘

## 답변

- Gateway 리소스 이름만 `shared-gateway`에서 `gateway`로 변경
- Application `gateway`, `envoy-gateway`, GatewayClass `envoy`, EnvoyProxy `envoy-proxy`는 유지
- Envoy 프록시 Service 이름에 Gateway 이름이 들어가므로 cloudflared 배포 전에 진행

## 변경 파일

| 파일 | 변경 |
|---|---|
| `infrastructure/gateway/gateway.yaml` | `metadata.name: shared-gateway` → `gateway` |
| `projects/choimory-dev/choimory-dev-front/base/httproute.yaml` | `parentRefs[0].name` → `gateway` |
| `projects/choimory-dev/choimory-dev-member-api/base/httproute.yaml` | `parentRefs[0].name` → `gateway` |
| `README.md` | 구성 표, sync wave 표의 `shared-gateway` 2곳 → `gateway` |
| `.agents/histories/calendar.md` | `PLAN:도메인 연동 작업(domain-connect)` → `TASK:도메인 연동 작업(domain-connect)` |

- 커밋과 push는 직접 진행

## 검증

- 저장소 전체(`.agents` 제외)에서 `shared-gateway` 참조가 남아 있지 않음 확인
- `kubectl kustomize`로 `infrastructure/gateway`, front stg overlay, member-api prod overlay 빌드 후 Gateway 이름과 `parentRefs`가 `gateway`로 나오는 것 확인

## push 후 확인

- `main` push 후 ArgoCD 동작
  1. 새 Gateway `gateway` 생성, 옛 `shared-gateway`는 prune으로 삭제
  2. Envoy Gateway가 프록시를 `envoy-choimory-infra-gateway-<해시>`로 새로 생성
  3. HTTPRoute가 새 Gateway에 연결
- 서버에서 확인

```bash
kubectl -n choimory-infra get gateway
kubectl -n choimory-infra get deploy,svc | grep envoy-choimory-infra
kubectl get httproute -A -o custom-columns='NS:.metadata.namespace,NAME:.metadata.name,ACCEPTED:.status.parents[0].conditions[?(@.type=="Accepted")].status'
```

- 기대 결과: Gateway `gateway`가 `PROGRAMMED True`, 프록시 이름이 `envoy-choimory-infra-gateway-*`, HTTPRoute 4개가 `ACCEPTED True`
- 옛 `envoy-choimory-infra-shared-gateway-*`가 남아 있으면 Envoy Gateway가 정리하기 전이므로 잠시 후 다시 확인

## 다음 작업

- 도메인 확인 (가비아 구매, 네임서버를 Cloudflare로 변경)
- cloudflared 배포와 Cloudflare Tunnel 연결
