# Retail-Platform

마트 POS · 카드 결제 · ERP를 하나의 유통 업무 흐름으로 연결하는 리테일 플랫폼의 **상위 레포**다.
각 시스템의 코드는 자기 레포에 두고, 이 레포에는 **두 시스템 이상에 걸치는 것**만 둔다.

> 완성 선언 기준: ERP까지 붙은 상태에서 "실제로 상용화해도 되겠다"는 판단이 들 때.
> 판단 근거는 [상용화 기준](docs/standards.md)과 `e2e/` 시나리오 테스트다.

## 구성 레포

| 레포 | 역할 | 상태 |
|---|---|---|
| [Retail-POS](https://github.com/yeongminChae/Retail-POS) | 판매 오케스트레이션 (장바구니 · 판매 · 결제 요청 · 반품) | 리팩토링·버그 수정 완료, API 변경 진행 중 |
| [PAYMENT-SIMULATOR](https://github.com/yeongminChae/PAYMENT-SIMULATOR) | 결제 중계 서버 (승인 · 조회 · 취소 · 망취소 · Recovery) + VAN Simulator | 독립 프로젝트로 완성, 연동 전 |
| Retail-ERP | 기준정보 · 구매 · 입고 · 재고 원장 · 마감 · 대사 | 설계 전 |

로컬에서는 한 폴더 아래 나란히 clone한다 ([ADR-0001](docs/adr/0001-형제-레포-구조.md)).

```text
~/retail/
├── Retail-Platform/
├── Retail-POS/
├── PAYMENT-SIMULATOR/
└── Retail-ERP/
```

## 이 레포의 구조

```text
docs/
├── summary.md        1단계 설계 요약 (지금 유효한 결정만) ← 처음 읽을 문서
├── standards.md      상용화 기준 체크리스트 (모든 설계의 판단 기준)
├── architecture.md   시스템 경계 · 데이터 소유권 · 연계 방식
├── flows/            end-to-end 업무 흐름 (입고 · 판매 · 반품 · 마감 · 대사)
├── adr/              시스템 간 결정 기록
└── backlog.md        나중에 설계할 것 (미룬 항목)
contracts/            시스템 간 API 스펙 · 이벤트 스키마
e2e/                  시스템을 넘나드는 시나리오 테스트
```

## 설계 진행 순서

| 단계 | 내용 | 상태 |
|---|---|---|
| 0 | 상용화 기준 확정 | 진행 중 |
| 1 | 플랫폼 설계 — 경계 · 데이터 소유권 · 연계 계약 · 공통 규칙 · 장애 정책 | **1차 완료** (ADR-0001~0042, 요약: [docs/summary.md](docs/summary.md)) |
| 2 | 모듈별 고도화 — POS · Payment Server · ERP 상세 | 대기 |
| 3 | ERD · API 스펙 → 각 레포의 구현 배치 계획 | 대기 |
