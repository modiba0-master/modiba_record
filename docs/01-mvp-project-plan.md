# AI 녹취록 서비스 MVP 기획 및 실행 계획

## 1. 핵심 방향

이 서비스의 핵심은 단순히 음성을 텍스트로 바꾸는 것이 아니다.

개발의 중심 언어는 Python으로 둔다. 백엔드는 FastAPI 기반으로 만들고, STT 처리, AI 후처리, 문서 생성, 작업 큐 처리는 모두 Python 서비스 계층에서 담당한다. PWA 화면은 브라우저 앱이므로 React 또는 Next.js 같은 프론트엔드가 별도로 필요하지만, 핵심 업무 로직은 Python 백엔드에 둔다.

핵심 가치는 다음 세 가지다.

1. 사용자가 긴 녹음에서 필요한 대화 구간을 쉽게 찾게 한다.
2. AI가 초안을 만들고, 속기사가 검수해 최종 녹취록을 납품한다.
3. 속기사가 전체 파일을 다시 듣는 시간을 줄여 작업 생산성을 높인다.

MVP의 기본 흐름은 다음과 같다.

```text
파일 업로드
-> 무료 기본 분석
-> 4,900원 AI 탐색 결제
-> 시간대별 대화 요약/주제 표시
-> 직접 선택 또는 AI 추천 의뢰
-> 최종 녹취 견적 확인
-> 결제
-> 속기사 검수/작성
-> 최종 녹취록 납품
```

## 2. 가장 중요한 제품 판단

STT 정확도 자체만으로는 차별화가 약하다. 시장에는 이미 Whisper, Soniox, Clova Note 등 음성 인식 도구가 많다.

이 서비스의 차별점은 다음이어야 한다.

```text
음성 -> AI 분석 -> 중요한 구간 추천 -> 속기사가 수정해야 할 위치 표시
```

즉, 사용자에게는 "어느 구간을 의뢰해야 하는지"를 알려주고, 속기사에게는 "어디부터 검토해야 하는지"를 알려주는 서비스로 설계한다.

## 3. 사용자 유형

### 의뢰자

- 개인 분쟁 당사자
- 고령층 사용자
- 변호사/노무사/손해사정사 사무실
- 기업, 병원, 상담/회의 기록 담당자

### 관리자

- 접수 건 확인
- 결제 및 상태 관리
- 속기사 배정
- 최종본 납품 관리

### 속기사

- AI 초안 확인
- 핵심 구간 검수
- 검토 필요 구간 재청취
- 최종 녹취록 작성

## 4. MVP 필수 기능

1. 파일 업로드
2. 무료 기본 분석
3. 4,900원 AI 탐색 결제
4. AI 시간대별 요약
5. AI 추천 의뢰
6. 구간 선택
7. 최종 녹취 견적 계산
8. 최종 결제 또는 상담 접수
9. 관리자 접수 목록
10. 속기사 작업 화면
11. 최종 파일 업로드 및 납품
12. 개인정보 동의 및 파일 접근 보안

추가로, 실제 운영에서는 의뢰인 1명이 여러 사건과 여러 음성파일을 의뢰할 수 있다. 따라서 MVP부터 `회원 -> 사건 -> 파일 -> 녹취 의뢰 -> 납품` 구조를 기본 전제로 둔다. 단순히 주문 1개에 파일 1개를 붙이는 구조로 시작하면, 여러 파일/여러 사건/구간별 의뢰를 처리할 때 DB 구조가 금방 한계에 부딪힌다.

## 5. 고령층 UX 기준

고령층 또는 앱 사용이 불편한 사용자는 "시간 구간" 중심 화면에서 막힐 가능성이 높다.

따라서 화면은 시간 선택보다 목적 선택 중심이어야 한다.

### 권장 화면 흐름

```text
1. 녹음파일 올리기
2. 무료 미리보기 확인
3. AI가 찾은 주요 내용 보기
4. 원하는 내용 버튼 선택
5. 추천 견적 확인
6. 결제
7. 완료 알림 받기
```

### 필수 버튼

```text
[입금 관련]
[환불 관련]
[계약 관련]
[폭언/협박 관련]
[전체 녹취]
[모르겠어요. AI가 추천해주세요]
```

`모르겠어요. AI가 추천해주세요` 버튼은 반드시 넣는다. 실제 사용자는 어느 구간이 중요한지 모르는 경우가 많고, 이 버튼이 결제 전환과 사용성 모두에 중요하다.

## 6. 무료 미리보기와 4,900원 탐색비

업로드 직후 바로 결제를 요구하면 이탈 가능성이 높을 수 있지만, 무료 단계에서 STT나 GPT 요약을 실행하면 비용 누수가 생길 수 있다.

따라서 결제 전에는 AI 토큰이나 STT 비용이 거의 들지 않는 정보만 보여준다.

```text
전체 녹음 길이: 10분
업로드 성공
지원 가능한 의뢰 방식
```

그 다음 CTA를 제공한다.

```text
[어디가 중요한지 모르겠다면 4,900원으로 AI 구간 탐색하기]
```

4,900원은 모든 의뢰인에게 강제하는 비용이 아니다. 전체 파일을 녹취할 사람이나 이미 필요한 구간을 아는 사람은 바로 견적으로 이동한다. 4,900원 AI 탐색은 필요한 구간을 모르는 의뢰인이 전체 녹취보다 저렴한 구간 녹취를 선택할 수 있게 돕는 선택 상품이다.

핵심 메시지는 다음이다.

```text
전체 파일을 맡기면 비용이 커질 수 있습니다.
AI가 먼저 중요한 대화 구간을 찾아드리면, 필요한 부분만 녹취 의뢰하여 최종 비용을 줄일 수 있습니다.
```

## 7. AI 처리 구조

Whisper와 Soniox는 병렬 처리한다.

```text
             ┌─ Whisper: 텍스트/타임스탬프
음성 파일 ───┤
             └─ Soniox: 화자분리/타임스탬프

-> Merge Engine
-> GPT 요약/주제분류/추천구간
```

직렬 처리하지 않는다.

### 상태값

```text
uploaded
preview_completed
ai_search_paid
stt_completed
speaker_completed
summary_completed
quote_ready
final_paid
assigned
reviewing
delivered
```

## 8. AI 분석 결과

AI 탐색 결제 후 사용자에게 다음을 보여준다.

1. 시간대별 요약
2. 주요 주제
3. 추천 의뢰 구간
4. 구간별 예상 녹취 비용
5. 전체 녹취 비용

예시:

```text
00:00~01:30 인사 및 상황 설명
01:30~04:20 입금 관련 대화
04:20~07:10 환불 요청
07:10~10:00 마무리 대화
```

사용자는 시간을 직접 고르는 대신 주제 버튼을 누를 수 있어야 한다.

## 9. 속기사 화면 핵심

속기사 화면은 단순히 음성과 텍스트를 나란히 보여주는 수준이면 부족하다.

반드시 다음 정보를 표시한다.

1. 선택된 의뢰 구간
2. AI 초안
3. 낮은 신뢰도 구간
4. 화자 중첩 가능 구간
5. 음성 불명확 구간
6. 금액/이름/날짜/장소 등 검토 필요 키워드

예시:

```text
검토 필요 8건
화자 중첩 2건
신뢰도 낮음 6건
핵심 구간 04:20~07:10
```

속기사 생산성을 높이는 핵심은 "전체를 다시 듣지 않게 하는 것"이다.

## 10. 데이터 모델 초안

### users

- id
- role: customer, admin, transcriber
- name
- phone
- email
- created_at

### cases

- id
- customer_id
- title
- case_type
- purpose
- description
- status
- created_at
- updated_at

### case_files

- id
- case_id
- customer_id
- original_filename
- storage_key
- mime_type
- size_bytes
- duration_seconds
- file_label
- upload_status
- created_at

### transcription_requests

- id
- case_id
- customer_id
- request_type
- status
- estimated_price
- final_price
- payment_status
- created_at
- completed_at

### transcription_request_files

- id
- request_id
- file_id
- selected_start_ms
- selected_end_ms
- selection_reason

### orders

- id
- customer_id
- status
- service_type
- duration_seconds
- selected_ranges
- estimated_price
- final_price
- payment_status
- deadline_type
- created_at
- completed_at

### files

- id
- order_id
- file_type: original, preview, final
- storage_key
- mime_type
- size_bytes
- duration_seconds
- created_at

### ai_jobs

- id
- order_id
- job_type: preview, whisper, soniox, merge, summary
- status
- started_at
- completed_at
- error_message

### transcript_segments

- id
- order_id
- start_ms
- end_ms
- speaker_label
- text
- confidence
- source
- needs_review

### ai_topics

- id
- order_id
- title
- summary
- start_ms
- end_ms
- recommendation_reason
- estimated_price

### review_flags

- id
- order_id
- segment_id
- flag_type: low_confidence, overlap, unclear_audio, legal_keyword, money, name, date
- severity
- note
- resolved_at

### transcript_versions

- id
- order_id
- version_type: ai_draft, transcriber_edit, final
- content
- created_by
- created_at

### deliveries

- id
- order_id
- file_id
- delivered_at

## 11. 개발 우선순위

### 1차 MVP

1. Python/FastAPI 백엔드 구조 세팅
2. 업로드
3. 무료 미리보기
4. 4,900원 AI 탐색 결제 상태
5. Whisper 연동
6. Soniox 연동
7. Merge Engine 기본형
8. GPT 시간대별 요약
9. AI 추천 의뢰
10. 최종 견적
11. 관리자/속기사 최소 화면

### 2차

1. 결제 자동화 고도화
2. 알림톡/문자/이메일
3. 속기사 배정
4. DOCX/PDF 자동 생성
5. 검토 필요 구간 추천 고도화

### 3차

1. 법원 제출용 양식
2. 속기사 생산성 대시보드
3. 작업자 정산
4. 고객 문서 보관함
5. B2B 제휴 관리

## 12. 예상 일정

### 현실적 MVP

```text
기획/설계: 3~5일
프로젝트 세팅: 2~3일
업로드/주문/견적: 1주
AI 파이프라인: 1~2주
사용자/관리자/속기사 화면: 2주
문서 납품/테스트: 1주
```

총 5~7주를 목표로 한다.

### 베타 운영

출시 후 1~2개월은 월 20~50건을 목표로 한다.

월 200건은 기술적으로 가능하지만, 초기 수요 확보 관점에서는 6개월 이후 목표로 잡는 것이 현실적이다.

## 13. 비용 및 처리 능력 판단

10분 음성 200건 기준:

- AI 비용: 월 10만원 이하 수준으로 관리 가능
- 속기사 작업: AI 보조 기준 건당 평균 10~20분
- 총 작업량: 약 50시간/월
- 숙련 속기사 1명 기준 처리 가능

다만 가장 어려운 것은 처리 능력이 아니라 월 200건의 수요 확보이다.

현실적 목표:

```text
출시 초기: 월 20~50건
6개월: 월 50~100건
1~2년: 월 200~300건
```

## 14. 성공 지표

1. 업로드 후 무료 미리보기까지 도달률
2. 4,900원 AI 탐색 결제 전환율
3. AI 추천 의뢰 선택률
4. 최종 녹취 견적 결제 전환율
5. 속기사 평균 작업 시간
6. 최종본 재수정 요청률
7. 월 반복 의뢰 수

가장 중요한 초기 지표는 `4,900원 AI 탐색 결제 전환율`이다.
