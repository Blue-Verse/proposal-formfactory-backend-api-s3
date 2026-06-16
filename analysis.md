# 비대면 스마트 서면 설계 웹 시스템 백엔드(API/DB) 및 클라우드(S3) 구축 — 제안 분석 로그

> 생성일: 2026-06-16
> 공고 URL: https://www.wishket.com/project/156123/

## 1. 공고 파싱 결과

```yaml
job:
  title: "비대면 스마트 서면 설계 웹 시스템 백엔드(API/DB) 및 클라우드(S3) 구축"
  category: "웹 / 백엔드 (API·DB·클라우드)"
  budget_range: "5,000,000원"
  duration: "30일"
  tech_stack: [Node.js, Spring Boot, Django, MySQL, PostgreSQL, REST API, AWS(EC2/RDS/S3)]
  description: "행정사사무소 스마트 서면 설계 플랫폼(FORMFACTORY)의 순수 백엔드(API 서버·DB·클라우드) 신규 구축. FE(Vanilla JS) 100% 완료, 20p RFP 첨부."
  requirements:
    - "RDBMS 스키마 설계·구축 (주문, 단가표, 설정)"
    - "LocalStorage 트랜잭션 → REST API 100% 대체 (fetch 1:1 바인딩)"
    - "S3 Presigned URL 다이렉트 업로드 (최대 2GB)"
    - "관리자 대시보드 통계·제어 API + 인증(Auth)"
    - "비즈뿌리오 외부 API 알림톡/SMS 자동 발송"
    - "3대 보안 로직 (UUID v4 브루트포스 방어, 웹쉘 차단, 페이로드 과부하 방지)"
  client_questions:
    - "Q1. 과업지시서 확인 + Vanilla JS FE 로직 100% 보존하며 fetch() REST API 바인딩 경험?"
    - "Q2. S3 Presigned URL 다이렉트 업로드 + UUID v4 파일명 난수화 구현 가능?"
    - "Q3. 관리자 대시보드 50MB 페이로드 과부하 방지를 위한 경량 트리 메타데이터 구현?"
  deadline: "2026년 06월 30일"
  job_post_url: "https://www.wishket.com/project/156123/"
  urls: ["https://www.wishket.com/project/155914/ (이전 공고 — 로그인 필요)"]
  images: ["폼팩토리 백엔드 과업지시서 V10.pdf (1.84MB, 첨부 — 직접 접근 불가)"]
  priority: ["1순위 산출물 완성도", "2순위 일정 준수", "3순위 금액"]
  maintenance: "월 단위 유지보수 예정 (미팅 시 논의)"
```

## 2. URL/이미지 분석

- 첨부 PDF(과업지시서 V10, 1.84MB)는 파일로 전달되지 않아 직접 분석 불가.
- 이전 공고 URL(155914)은 위시켓 로그인이 필요해 접근 불가.
- 공고 본문에 RFP 핵심(DB 스키마, API 매핑, S3 방식, 3대 보안)이 상세 서술되어 있어 본문 기반으로 진행.
- **참고 URL/이미지 직접 분석 없음** — 텍스트 명세 기반 제안.

## 3. 실현 가능성 분석 (내부용)

- **프로젝트 유형**: 순수 백엔드 (API + DB + S3 + 인증 + 외부 API + 보안) → "결제/인증/외부 API 연동" 유형, 보안 리뷰 버퍼 +15%
- **FE/디자인/퍼블리싱**: 0 M/D (명시 제약)
- **기본 공수 (AI 보조 없이)**: 36 M/D
  - DB 스키마·ERD 4 / REST API 11 / S3 업로드 4 / 관리자+인증 5 / 알림 연동 3 / 3대 보안 4 / 명세·QA·배포 5
- **AI 반영 공수 (~50% 절감)**: 18 M/D → +15% 버퍼 ≈ 20.7 M/D
- **1인 달력 환산**: 20.7 × (7/5) ≈ 29일 → 클라이언트 예상 30일 내 충분히 가능
- **판정: 강력 가능** — 다수 인력 투입 시 설계·개발 병행으로 일정 여유, 1순위 "산출물 완성도"에 자원 집중

## 4. 포트폴리오 매칭

| 순위 | 프로젝트 | 매칭 점수 근거 |
|------|---------|--------------|
| 1 | **EZ-Approve (전자결재 SaaS)** | 거의 1:1 — Presigned URL 업로드 + 브루트포스 방어·Rate Limiting(페이로드 과부하 방지) + Ppurio(비즈뿌리오) SMS + 관리자 API/인증. NestJS+MySQL, 120~150 API. |
| 2 | **Series-B (VC 펀드 관리)** | 대규모 REST API + AWS(EC2/RDS/S3) 인프라 + 5중 보안 + Ppurio SMS, 200~300 API, 179 마이그레이션. 인프라·보안·알림 일치. |
| 3 | **P2P-Funding (P2P 크라우드펀딩)** | AWS S3 + SMS 자동 트리거(큐) + 관리자 200+ API 대시보드 + 15+ 외부 연동. S3·외부알림·관리자 일치, 스택 유연성(Django) 입증. |

## 5. 최종 제안 요약

- **지원 금액**: 4,250,000원 (VAT 별도) — 클라이언트 예상 5,000,000원의 85% (다수 인력으로 기간 유지 + 가격 경쟁력)
- **지원 기간**: 30일 (클라이언트 예상 유지)
- **핵심 제안 포인트**:
  1. "프론트엔드 0원" 제약 정확 이해 — 순수 백엔드 + fetch() 1:1 바인딩
  2. Presigned URL + UUID v4 난수화 검증된 구현 경험 (EZ-Approve)
  3. 3대 보안 로직(브루트포스/웹쉘/페이로드) 다계층 보안 구축 경험
  4. 비즈뿌리오(Ppurio) 동일 제공사 연동 경험으로 리스크 최소화
  5. 산출물 완성도(1순위) 집중 — ERD·Swagger/Postman·배포 가이드 완비
- **기술 스택**: Node.js(NestJS) 우선, Spring Boot/Django 협의 가능 · MySQL · AWS(EC2/RDS/S3)
