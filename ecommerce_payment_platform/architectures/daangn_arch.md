# 당근마켓 추정 아키텍처

> 참조일: 2026-06-30 | 출처: medium.com/daangn(공식 기술 블로그) + 채용공고 기반 추정.
> 상세 근거: [02_service_selection.md](../report/02_service_selection.md) #5

---

## 1. 전체 아키텍처 개요

```
┌──────────────────────────────────────────────────────────────────────────┐
│                         당근마켓 아키텍처 전체상                           │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  [초기 — Rails 모놀리스 + PostgreSQL 단일 DB]                             │
│         │                                                                │
│         ▼ (채팅 데이터가 메인 DB의 60% 차지 — MSA 분리 직접 동인)            │
│  [MSA 전환 — 폴리글랏: Java/Kotlin/Go/TS, 서비스 간 gRPC]                  │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────────────┐             │
│  │ 채팅 서비스   │ │ 회원/인증     │ │ Identity Service      │             │
│  │ (Go+DynamoDB)│ │ (1년+ 장기   │ │ (Go/gRPC/MySQL/Redis/  │             │
│  │              │ │  프로젝트)    │ │  Kafka/k8s/**Istio**)  │             │
│  └──────────────┘ └──────────────┘ └──────────────────────┘             │
│                                          ▲                                │
│                              공식 채용공고에서 Istio 명시 확인              │
│                              (일부 도메인만, 전사 범위는 불명확)             │
│         │                                                                │
│         ▼                                                                │
│  [당근페이 — MSA 대신 모노레포 채택(대조 사례)]                            │
│  ┌────────────────────────────────────────────────────────────┐          │
│  │ Hexagonal → Clean Architecture 모노레포                       │          │
│  │ Strangler Fig + Feature Toggle 점진 전환                      │          │
│  └────────────────────────────────────────────────────────────┘          │
│         │                                                                │
│         ▼                                                                │
│  [CD 계층 — 검색 플랫폼팀 사례]                                           │
│  GitHub Actions(빌드) + Kustomize(이미지 태그) + ArgoCD(GitOps Sync)      │
│  ASG 수동 조절 대비 배포시간 5시간 → 30분 이내 단축                       │
│  SRE팀: Cloud/Cluster/Delivery 3파트 조직, 메시 로드맵 전담                │
│         │                                                                │
│         ▼                                                                │
│  [관찰성]                                                                │
│  검색 플랫폼: Metricbeat/Filebeat/Kibana + Prometheus Exporter+Grafana    │
│  SRE팀: kube-apiserver 감사로그 + AWS EventBridge + Slack                │
│  (모든 kubectl 액션 실시간 투명성 — 독특한 보안 가시성 사례)                │
│         │                                                                │
│         ▼                                                                │
│  [복원력 — 6개사 중 가장 투명한 장애 공개]                                 │
│  실 사례: Elastic Operator 롤링재시작 중 "Cold New Node" 문제              │
│  (검색 API 에러율 60%, 레이턴시 3초 급증)                                 │
│  → search-coordinator 프록시 + Redis 분산락 워밍업 시스템 구축             │
│  배포 6시간→1~2시간, p99 1초 이내 안정화 (1인칭 상세 회고 공개)            │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```
<img width="1086" height="1448" alt="image" src="https://github.com/user-attachments/assets/d6afc4d8-7fac-4d7c-bb20-2ef3816fa3d6" />


## 2. 5축 요약

| 축 | 내용 |
|---|---|
| 아키텍처 | MSA(폴리글랏, Go+DynamoDB 채팅) + 당근페이는 모노레포(대조 사례) |
| CI/CD | GitHub Actions+Kustomize+ArgoCD(GitOps), 5시간→30분 단축 |
| 관찰성 | Prometheus+Grafana + kube-apiserver 감사로그(독특한 보안 가시성) |
| 복원력 | 실 장애 사례 기반 워밍업 시스템 — 6개사 중 가장 투명한 포스트모템급 공개 |
| 서비스 메시 | Istio(Identity Service 등 일부 도메인, 채용공고로 확인) |

## 3. 출처

- [회원 시스템 MSA 전환기](https://medium.com/daangn/%ED%9A%8C%EC%9B%90-%EC%8B%9C%EC%8A%A4%ED%85%9C-msa-%EC%A0%84%ED%99%98-%EB%8F%84%EC%A0%84%EA%B8%B0-mau-1-900%EB%A7%8C-%EB%8B%B9%EA%B7%BC-%EC%9C%A0%EC%A0%80%EB%A5%BC-%EC%9C%84%ED%95%9C-%EC%84%A0%ED%83%9D-43993c582f69)
- [당근페이 아키텍처 여정](https://medium.com/daangn/%EB%8B%B9%EA%B7%BC%ED%8E%98%EC%9D%B4-%EB%B0%B1%EC%97%94%EB%93%9C-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98%EA%B0%80-%EA%B1%B8%EC%96%B4%EC%98%A8-%EC%97%AC%EC%A0%95-98615d5a6b06)
- [검색 엔진 쿠버네티스 운영](https://medium.com/daangn/%EB%8B%B9%EA%B7%BC%EB%A7%88%EC%BC%93-%EA%B2%80%EC%83%89-%EC%97%94%EC%A7%84-%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4%EB%A1%9C-%EC%89%BD%EA%B2%8C-%EC%9A%B4%EC%98%81%ED%95%98%EA%B8%B0-bdf2688df267)
- [kube-apiserver 감사 로그](https://medium.com/daangn/kubectl-create-pod%EB%A5%BC-%EC%8B%A4%ED%96%89%ED%95%98%EB%A9%B4-%EB%B0%9C%EC%83%9D%ED%95%98%EB%8A%94-%EC%9D%BC-kube-apiserver-%EA%B0%90%EC%82%AC-%EB%A1%9C%EA%B7%B8-audig-log-%EB%A1%9C-%EC%97%BF%EB%B3%B4%EA%B8%B0-6f01487abdda)
- [Elasticsearch Warm-Up Part 2](https://medium.com/daangn/running-elasticsearch-on-kubernetes-the-easy-way-part-2-data-node-warm-up-0d81d433c5c1)
- [당근마켓 채용 - Identity Service](https://careers.daangn.com/jobs/role/5046759003/)
