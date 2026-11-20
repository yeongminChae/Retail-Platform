# 나중에 설계할 것

1단계 설계 중 "지금은 미루기로 한" 항목을 모아 둔다. 정해지면 ADR(시스템 간) 또는 해당 레포 `docs/`(한 시스템 안)로 옮기고 여기서 지운다.

## 시스템 간 (이 레포에서 결정)

| 항목 | 내용 | 관련 | 언제 |
|---|---|---|---|
| 서버 재기동 시 미완료 판매 (POS_APP Q9 재검토) | Payment Server는 따로 돌아 승인 기록이 안전하므로 자동 취소는 불필요. 미완료 판매를 이어서 진행할지, 손님이 떠난 경우 처리 | ADR-0021, architecture.md 5장 Q9 | 장바구니 저장소(Redis 도입 여부)와 함께 |
| PAYMENT-SIMULATOR 운영 화면 (GUI) | posTrx 상태 조회 · 수기 조치(취소 호출)용 화면. VAN사 가맹점 관리 화면 역할 | ADR-0022, ADR-0026 | 운영 조회 API 구현 후 필요해지면 |
| 망취소 거절 시 POS 규칙 | Recovery가 원승인을 먼저 확정하면 망취소가 거절됨 → 재조회 후 승인이면 취소로 전환 | ADR-0003, ADR-0021 | Q9와 함께 |

## POS_APP 안 (POS_APP 레포 `docs/`로 옮길 것)

POS_APP 정리 중 발견 (2026-11-20, POS_APP `main` `fc282eb` 기준).

### 장바구니 저장소 · 동시성
- 장바구니는 인메모리. 같은 거래 키에 동시 요청이 오면 수량 변경 · 취소가 원자적이지 않음 (서로 다른 거래끼리는 안전 — `ConcurrentPosIntegrationTest`).
- 현재 전제: "한 거래는 한 단말(POS/키오스크)이 다룬다" — 코드 주석에 기록됨.
- 장바구니를 Redis · DB로 옮기거나 서버를 여러 대로 늘릴 때 동시성 정책과 함께 결정. 재기동 시 장바구니 유실 여부가 Q9와 연결됨.

### 미사용 코드 정리 (refactor)
- `domain/model/product/Product.java` — 클래스 전체 미사용 (상품 조회는 `ProductQueryDto`). 상품/메뉴 모델 설계 시 재사용 또는 삭제 결정 (ADR-0007: 상품은 ERP 사본).
- `Cart.getCartLineQty()` — 호출처 없음.
- `Payment`의 편의 메서드 `paymentId()` · `posNo()` · `tranNo()` · `requestedAt()` — 호출처 없음 (JSON 직렬화 대상 아님).

### 스키마 주석 불일치
- `schema.sql`의 `SALES.APPROVED_AT` · `RECEIPT_JOURNAL.APPROVED_AT` 주석은 "ISO8601"이지만 실제 저장 형식은 `yyyy-MM-dd HH:mm:ss` (Asia/Seoul).
- 운영 DB에서 추출한 파일이라 운영 스키마 주석부터 고쳐야 함 → 스키마 정리 시 POS_APP `docs/schema-issues.md`와 함께.

### 문서 정정
- POS_APP CLAUDE.md "미완성 부분"의 "`ProductController`의 엔드포인트는 주석 처리됨" → 실제로는 경로(`/api/products`)만 있고 엔드포인트 없음. 상품/메뉴 API 추가 시 갱신.

### 기존 보류 항목 (POS_APP `docs/bugfix-plan.md` · `api-change-plan.md`)
- B12c 성인 인증, B14b 계산원(현재 "테스터" 고정 → ADR-0011), B12d 재고 부족 차단(정책상 허용 → ADR-0010), B17 enum ↔ DB CHECK 불일치, T2 조회 트랜잭션, L5 결제 수단별 저장 정책(카드).
- `CartResDto` · `CartSummaryResDto` 필드가 현재 동일 (분리 유지 결정). 판매 응답 `storeCd` ↔ 장바구니 `strCd` 키 이름 불일치.
