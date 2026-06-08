# Modiba Record

AI 구간 탐색, 자동 견적, 속기사 검수, 최종 녹취록 납품까지 연결하는 녹취록 의뢰 플랫폼입니다.

초기 목표는 완전 자동 녹취 서비스가 아니라, 기존 고객이 음성파일을 쉽게 올리고 재생시간 기준 견적을 확인한 뒤 결제하고, 속기사가 AI 초안을 검수해 최종 녹취록을 납품하는 운영형 웹/PWA 서비스입니다.

## 핵심 컨셉

```text
의뢰인
-> 사건 생성
-> 음성/증거자료 업로드
-> 전체 녹취, 지정 구간 녹취, AI 구간 탐색 중 선택
-> 자동 견적 확인
-> 결제
-> 속기사 배정/검수
-> 최종 녹취록 납품
```

이 서비스의 차별점은 단순 STT가 아니라 다음입니다.

```text
AI가 중요한 구간을 찾고,
속기사가 수정해야 할 위치를 줄이며,
관리자가 사건/고객/결제/납품을 통합 관리한다.
```

## 기술 방향

- Main Language: Python
- Backend: FastAPI
- Frontend/PWA: React 또는 Next.js
- Database: MariaDB
- Storage: Cloudflare R2
- Deploy: Railway
- Container: Docker
- STT: Whisper + Soniox
- AI 후처리: GPT 계열 모델

## 주요 정책

- 파일은 DB가 아니라 R2에 저장한다.
- DB에는 storage key와 metadata를 저장한다.
- 긴 AI 작업은 API 요청 안에서 직접 처리하지 않고 job/worker 구조로 분리한다.
- Whisper와 Soniox는 병렬 처리한다.
- 실제 API 연동 전 mock harness로 전체 흐름을 검증한다.
- 4,900원 AI 구간 탐색은 구간을 모르는 의뢰인을 위한 선택 옵션이다.
- AI 구간 탐색비는 최종 녹취록 의뢰 시 차감하는 방향으로 설계한다.
- 의뢰인, 관리자, 속기사는 접속환경과 권한을 분리한다.
- 장기적으로 사건 자료 관리 플랫폼으로 확장할 수 있게 `사건 -> 자료 -> 의뢰` 구조를 전제로 둔다.

## 문서 목록

- [MVP 기획 및 실행 계획](docs/01-mvp-project-plan.md)
- [Cursor 개발 지시서](docs/02-cursor-agent-brief.md)
- [개발 환경, Cursor 작업계획, Railway 세팅 분담](docs/03-development-setup-and-cursor-plan.md)
- [역할별 접속환경, 사건/파일 구조, 회원가입/결제 UX 계획](docs/04-role-workflows-cases-and-payment-plan.md)
- [요금표 기반 자동 견적 및 결제 유도 계획](docs/05-pricing-and-payment-automation-plan.md)
- [관리자 CRM, 의뢰 목적 관리, 재의뢰 혜택 및 결제 연동 계획](docs/06-admin-crm-and-repeat-customer-benefits.md)
- [속기사 배정 방식, 기량 관리, 급행 요금 정책](docs/07-transcriber-assignment-and-rush-fee-plan.md)

## 1차 MVP 우선순위

1. 파일 업로드
2. 재생시간 자동 분석
3. 요금표 기반 자동 견적
4. 결제창 연결
5. 관리자 접수 확인
6. AI 구간 탐색 옵션
7. 속기사 작업 화면

## 내일 이어갈 작업 계획

내일은 기획 추가보다 실제 개발 착수를 위한 뼈대를 만드는 것이 좋습니다.

추천 순서:

1. FastAPI backend 기본 구조 생성
2. `.env.example` 작성
3. `Dockerfile`, `docker-compose.yml` 작성
4. MariaDB 연결 설정
5. `/health` API 생성
6. SQLAlchemy/Alembic 초기 세팅
7. 요금표 기반 `pricing.py` 설계
8. mock harness 폴더와 샘플 JSON 구조 생성

Cursor에게 시작 지시를 줄 때는 `docs/02-cursor-agent-brief.md`와 `docs/03-development-setup-and-cursor-plan.md`를 먼저 읽게 한 뒤, backend 기본 구조부터 만들도록 지시합니다.

## 오늘 작업 상태

- 기획 문서 7개 작성 완료
- GitHub 저장소 문서 업로드 완료
- README 생성 완료
- 다음 단계는 개발 환경 및 backend scaffold 생성
