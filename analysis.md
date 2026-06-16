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

- **첨부 PDF(과업지시서 V10, 20p) 전문 분석 완료** — pdfjs-dist(Node)로 텍스트 추출(poppler 미설치로 이미지 렌더 불가, py 부재). 추출 결과 `C:/Users/user/AppData/Local/Temp/pdfx/_rfp.txt`.
- 이전 공고 URL(155914)은 위시켓 로그인 필요로 접근 불가.
- **RFP에서 확인된 공고 본문 외 추가 명세 (제안서에 반영)**:
  - 도메인 jyformfactory.com(구매완료), 마스터본 = 3단계 마법사·딥서치·노무계산기·관리자 10탭
  - AS-IS: btoa(encodeURIComponent()) LocalStorage 평문 암호화 → TO-BE: Base64 우회 순수 JSON 송수신(보안 HTTPS 대체)
  - DB 6테이블 정밀 명세: Orders(uuid PK/status 1~5,-1/phone IDX/attachments JSON), Pricing(Team/Center/Items 3단 트리·customGuideObj·isClosed), Reviews/Cases(PK=timestamp BigInt), Marquee, Config(ff_set_*), Popup(신설)
  - API 25+: config/popup/pricing(80트리)/marquee/reviews/cases, SSE·폴링(storage 이벤트 대체, /api/orders/stats), POST orders(kakao-modal·shakeError 보존), track(이름 마스킹 홍*동·30일 블라인드), S3 4단계 업로드+files sync/download-url/hardDelete, admin login/password, admin/orders(연-월-일 트리·경량 메타), 기간필터, status CRUD/restore/hardDelete, draft get/put, pricing PUT /bulk(Replace), CMS(timestamp 키), SMS 장부 soft/restore/hard
  - 알림톡: 비즈뿌리오/알리고, 변수 치환({이름}{금액}{일정}), 트리거(제출/상태변경/수임료), **알림톡 실패 시 LMS/SMS Fallback**
  - **방어 로직 7대** (§8 4대: Key Collision·Rate Limiting/과금폭탄(reCaptcha)·DB Transaction Lock·S3 고아파일 Cron GC + §9 3대: UUID v4 브루트포스·WebShell MIME 차단·50MB Lazy Load) + §10 인프라(HTTPS·SQLi·CORS) — 공고에는 "3대"만 언급되었으나 RFP는 실제 7대
  - **최우선 평가지표: 프론트엔드 파손 제로(콘솔 에러 0건)**, 잔금 요건: GitHub 권한 이관·ERD·Swagger/Postman
- **영향**: 실제 범위가 공고 본문보다 큼(1인 ~37일 산정). 다수 인력 병행으로 30일 유지 + 공수 21→26.5 M/D로 상향, 보안 "3대→7대" 정정 반영.

## 3. 실현 가능성 분석 (내부용)

- **프로젝트 유형**: 순수 백엔드 (API + DB + S3 + 인증 + 외부 API + 보안) → "결제/인증/외부 API 연동" 유형, 보안 리뷰 버퍼 +15%
- **FE/디자인/퍼블리싱**: 0 M/D (명시 제약)
- **기본 공수 (AI 보조 없이, RFP 완독 후 상향)**: ~46 M/D
  - DB 6테이블·ERD / 공개조회·라이브동기화 / 폼·트래커 / S3 4단계·파일관리·GC / 관리자 인증·트리·필터 / 관리자 CRUD·에디터·Bulk·CMS·SMS장부 / 알림톡 치환·트리거·Fallback / 7대 방어+인프라 / QA(파손제로)·문서·배포
- **AI 반영 공수 (~50% 절감)**: ~23 M/D → +15% 버퍼 ≈ 26.5 M/D
- **1인 달력 환산**: 26.5 × (7/5) ≈ 37일 (> 30일)
- **판정: 가능 (다수 인력 필수)** — 2인 병행 시 calendar ~18.5일 → 30일 충분. 1순위 "산출물 완성도"에 자원 집중. 공개 공수계산서는 26.5 M/D로 표기, 30일은 다수 인력 병행으로 충족(1인/AI 언급 금지).

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

## 6. 최종 산출물 (8단계 출력 전문)

### 6.1 제안서 사이트 URL
https://proposal-router.claude-ai-b27.workers.dev/proposal-formfactory-backend-api-s3/

### 6.2 지원 금액
4,250,000원 (VAT 별도) — 클라이언트 예상 5,000,000원의 85%

### 6.3 지원 기간
30일

### 6.4 클라이언트 사전 검증 질문 답변

**Q1. 과업지시서 확인 + Vanilla JS FE 로직 100% 보존하며 fetch() REST API 바인딩 경험?**
네, 20페이지 과업지시서를 완독했습니다. Orders/Pricing 등 6종 테이블 컬럼 매핑, 25개 이상의 API 바인딩 명세, S3 4단계 다이렉트 업로드, AS-IS의 btoa/Base64 평문 암호화 인터셉터, 그리고 §8·§9의 7대 방어 로직까지 모두 확인했습니다. 기존 화면 단 로직(3단계 마법사·딥서치·노무계산기·관리자 10탭)을 일절 수정하지 않고, fetch() 호출 지점에 1:1 대응하는 REST API를 설계·연동한 경험이 있습니다. 무통장 입금 kakao-modal과 shakeError 통제 순서 등 기존 JS 원형을 100% 유지하며, React/Vue 전환 없이 제공된 Vanilla JS 소스를 그대로 보존합니다. 본 과업 최우선 평가지표인 "프론트엔드 파손 제로(콘솔 에러 0건)"를 검수 기준으로 삼겠습니다.

**Q2. S3 Presigned URL 다이렉트 업로드 + UUID v4 파일명 난수화 구현 가능?**
네, 가능합니다. 서버를 경유하지 않고 클라이언트가 S3로 직접 업로드하는 Presigned URL 다이렉트 업로드 아키텍처를 운영 수준으로 구현한 경험이 있습니다. 1회 최대 2GB 대용량 파일은 멀티파트 업로드로 안정 처리하며, 업로드 객체 키는 UUID v4로 난수화하여 Key Collision과 추측 접근을 방어합니다. 발급되는 Presigned URL에는 짧은 만료 시간과 콘텐츠 타입/크기 제약을 적용해 오남용을 차단합니다.

**Q3. 관리자 대시보드 50MB 페이로드 과부하 방지를 위한 경량 트리 메타데이터 구현?**
네, 가능합니다. 관리자 대시보드 응답은 본문/텍스트 등 무거운 필드를 제외하고, 트리 구조와 식별자·요약 정보만 담은 경량 메타데이터로 우선 전달하도록 설계합니다. 상세 본문은 노드 선택 시 별도 API로 지연 로딩하여 초기 페이로드를 최소화하고, 요청 크기 제한과 페이지네이션을 함께 적용해 50MB 과부하 크래시를 방지합니다.

### 6.5 지원 내용 (전체 텍스트)

안녕하세요, 비대면 스마트 서면 설계 웹 시스템 백엔드(API/DB) 및 클라우드(S3) 구축 프로젝트에 지원합니다.

본 프로젝트에 대한 상세 제안서(견적서, 공수계산서, PRD, 일정, 포트폴리오)를 별도 페이지로 준비하였습니다.
▶ 제안서 상세 페이지: https://proposal-router.claude-ai-b27.workers.dev/proposal-formfactory-backend-api-s3/
▶ 위시켓 포트폴리오: https://www.wishket.com/partners/p/blueverse1/

[프로젝트 분석] 순수 백엔드(API·DB·클라우드) 구축. Vanilla JS FE 100% 보존 + LocalStorage→REST API 1:1 대체. FE 공수 0원, PG 제외(무통장 유지).
[핵심 과업] 6종 RDBMS 스키마·ERD / 25+ REST API(라이브싱크·트래커 마스킹 포함) / S3 4단계 업로드(2GB)+UUID / 관리자 대시보드 트리·CRUD·draft·Bulk·CMS·인증 / 비즈뿌리오 알림톡·SMS(Fallback) / 7대 방어 로직.
[일정 30일] P1 분석·DB설계(D1-6) / P2 핵심 API(D7-18) / P3 S3·관리자·연동·보안(D19-26) / P4 통합 QA·배포(D27-30).
[산출물] 백엔드 소스 일체, ERD, REST API 명세서(Swagger/Postman), 배포 가이드.
[협의] 프레임워크 확정(Node.js/NestJS 우선, Spring Boot/Django 협의), AWS·비즈뿌리오 정보 공유, 월 단위 유지보수 범위.

### 6.6 관련 포트폴리오 추천
1. 전자결재·업무관리 SaaS (EZ-Approve) — Presigned URL + 브루트포스/Rate Limiting + 비즈뿌리오 SMS 거의 1:1 일치
2. VC 투자조합 관리 플랫폼 (Series-B) — 대규모 REST API + AWS 인프라 + 다계층 보안 + SMS 연동
3. P2P 크라우드펀딩 플랫폼 (P2P-Funding) — AWS S3 + 외부 API 알림 자동 발송 + 관리자 대시보드 API
