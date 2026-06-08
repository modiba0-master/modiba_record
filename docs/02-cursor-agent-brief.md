# Cursor 개발 지시서

## 프로젝트 개요

AI 녹취록 의뢰 및 속기사 검수 플랫폼을 만든다.

기술 스택:

- Main Language: Python
- Backend: FastAPI
- Frontend: React 또는 Next.js 기반 PWA
- Database: MariaDB
- Storage: Cloudflare R2
- Deploy: Railway
- Container: Docker
- STT: Whisper + Soniox
- AI 후처리: GPT 계열 모델

## 반드시 지켜야 할 제품 방향

이 서비스는 STT 도구가 아니라 `AI 구간 탐색 + 속기사 검수 + 최종 녹취록 납품` 서비스다.

초기 MVP에서 가장 중요한 사용자 흐름:

```text
음성 파일 업로드
-> 무료 기본 분석
-> 4,900원 AI 탐색 결제
-> 시간대별 대화 요약
-> 직접 구간 선택 또는 AI 추천 의뢰
-> 최종 견적 확인
-> 결제
-> 속기사 작업
-> 최종 녹취록 납품
```

## 개발 원칙

1. 파일은 R2에 저장하고 DB에는 storage key만 저장한다.
2. R2 버킷은 비공개를 전제로 한다.
3. 다운로드는 만료형 signed URL로 처리한다.
4. Whisper와 Soniox는 병렬 작업으로 설계한다.
5. 긴 작업은 API 요청 안에서 직접 처리하지 말고 worker/job 구조로 분리한다.
6. 모든 주문은 상태값으로 추적한다.
7. AI 결과와 사람이 수정한 결과는 버전으로 남긴다.
8. 개인정보와 원본 음성 접근 기록을 남길 수 있는 구조로 만든다.
9. 핵심 비즈니스 로직은 Python 백엔드에 둔다.
10. STT, Merge Engine, GPT 요약, 문서 생성은 Python service 모듈로 분리한다.

## 추천 백엔드 구조

```text
backend/
  app/
    main.py
    core/
      config.py
      security.py
      storage.py
    db/
      session.py
      models.py
      migrations/
    api/
      routes/
        auth.py
        uploads.py
        orders.py
        ai.py
        admin.py
        transcriber.py
        deliveries.py
    services/
      r2_storage.py
      pricing.py
      stt_whisper.py
      stt_soniox.py
      merge_engine.py
      ai_summary.py
      document_generator.py
    workers/
      tasks.py
```

Python 권장 기준:

```text
Python 3.11 또는 3.12
FastAPI
SQLAlchemy 2.x
Alembic
Pydantic v2
Uvicorn/Gunicorn
httpx
boto3 또는 S3 호환 클라이언트
python-docx
pytest
ruff
```

## 추천 프론트엔드 구조

```text
frontend/
  src/
    app/
    components/
    features/
      upload/
      preview/
      ai-search/
      quote/
      admin/
      transcriber/
      delivery/
    lib/
      api.ts
      format.ts
```

## 핵심 화면

### 고객 화면

1. 파일 업로드
2. 무료 미리보기
3. AI 탐색 결제 안내
4. 시간대별 요약
5. `모르겠어요. AI가 추천해주세요` 버튼
6. 구간/주제 선택
7. 최종 견적
8. 결제/상담 요청
9. 진행 상태
10. 최종 파일 다운로드

### 관리자 화면

1. 주문 목록
2. 주문 상세
3. 원본 파일 확인
4. AI 분석 결과 확인
5. 결제 상태 변경
6. 속기사 배정
7. 최종 파일 업로드
8. 납품 처리

### 속기사 화면

1. 배정된 작업 목록
2. 오디오 플레이어
3. AI 초안 편집기
4. 검토 필요 구간 목록
5. 화자 중첩/신뢰도 낮음/음성 불명확 플래그
6. 최종 제출

## 주요 상태값

```text
uploaded
preview_processing
preview_completed
ai_search_paid
stt_processing
stt_completed
speaker_completed
summary_completed
quote_ready
final_paid
assigned
reviewing
final_uploaded
delivered
cancelled
failed
```

## 가격 정책 초기값

실제 운영 전까지는 설정 파일 또는 DB 테이블로 관리한다.

초기 예시:

```text
AI 탐색비: 4,900원
5분 미만: 30,000원
10분 미만: 60,000원
1시간 기준: 168,985원
긴급 작업: 20~50% 가산
음질 불량/다자간/법원 제출용: 추가요금 또는 상담
```

## API 초안

```text
POST /uploads/presign
POST /orders
GET /orders/{order_id}
POST /orders/{order_id}/preview
POST /orders/{order_id}/ai-search/payment-complete
GET /orders/{order_id}/topics
POST /orders/{order_id}/select-topic
GET /orders/{order_id}/quote
POST /orders/{order_id}/final-payment-complete

GET /admin/orders
GET /admin/orders/{order_id}
PATCH /admin/orders/{order_id}/status
POST /admin/orders/{order_id}/assign
POST /admin/orders/{order_id}/delivery

GET /transcriber/jobs
GET /transcriber/jobs/{order_id}
PATCH /transcriber/jobs/{order_id}/segments
POST /transcriber/jobs/{order_id}/submit
```

## 1차 구현 순서

1. Python/FastAPI 프로젝트 세팅
2. MariaDB 연결 및 마이그레이션
3. R2 presigned upload
4. 주문 생성
5. 무료 미리보기 작업
6. AI 탐색 결제 완료 상태 처리
7. Whisper/Soniox worker stub
8. Python Merge Engine 기본 구현
9. GPT 요약/주제 생성
10. 고객용 주제 선택 화면
11. 견적 계산
12. 관리자 주문 상세
13. 속기사 편집 화면
14. 최종 파일 업로드/납품

## 테스트 기준

1. 10분 파일 업로드가 성공해야 한다.
2. 업로드 후 무료 미리보기가 생성되어야 한다.
3. AI 탐색 결제 완료 후 상세 요약이 보여야 한다.
4. 사용자는 시간 대신 주제 버튼으로 구간을 선택할 수 있어야 한다.
5. `AI 추천 의뢰` 버튼으로 추천 구간을 자동 선택할 수 있어야 한다.
6. 최종 견적이 구간 길이에 따라 계산되어야 한다.
7. 속기사 화면에서 검토 필요 구간이 보여야 한다.
8. 최종 파일 납품 링크는 공개 URL이 아니라 만료형 URL이어야 한다.
