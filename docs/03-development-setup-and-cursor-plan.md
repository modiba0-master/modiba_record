# 개발 환경, Cursor 작업계획, Railway 세팅 분담

## 1. 기본 전제

이 프로젝트는 Python 중심으로 개발한다.

```text
Backend: Python + FastAPI
AI pipeline: Python worker
Database: MariaDB
Storage: Cloudflare R2
STT: Whisper + Soniox
Deploy: Railway
Container: Docker
Frontend/PWA: React 또는 Next.js
```

Cursor는 코드 생성, 구조화, 테스트, Docker/Railway 배포 파일 작성까지 담당한다.

사용자는 외부 계정, 결제수단, API 키, Railway 프로젝트 생성, R2 버킷 생성 같은 권한성 작업을 담당한다.

## 2. 작업 전 룰

Cursor에게 아래 룰을 먼저 전달한다.

```text
1. 임의로 스택을 변경하지 않는다.
2. 백엔드는 Python/FastAPI 중심으로 구현한다.
3. 긴 작업은 API 요청 안에서 직접 처리하지 않고 job/worker 구조로 분리한다.
4. 파일은 DB에 저장하지 않고 R2에 저장한다.
5. DB에는 파일 URL이 아니라 storage key와 metadata를 저장한다.
6. 환경변수는 코드에 하드코딩하지 않는다.
7. .env 파일은 Git에 커밋하지 않는다.
8. 모든 외부 API 연동은 mock mode와 real mode를 분리한다.
9. 먼저 harness로 동작 검증 후 실제 API를 붙인다.
10. 마이그레이션 없이 DB 모델을 직접 바꾸지 않는다.
11. 모든 핵심 API는 테스트를 작성한다.
12. 최종 납품 링크는 공개 URL이 아니라 signed URL로 만든다.
13. 개인정보/음성파일 접근은 추후 감사 로그를 남길 수 있는 구조로 만든다.
```

## 3. 역할 분담

### 사용자가 직접 해야 하는 일

1. GitHub 저장소 생성
2. Railway 계정 생성 및 프로젝트 생성
3. Railway에 MariaDB 서비스 생성
4. Railway에 Backend 서비스 생성
5. Cloudflare R2 버킷 생성
6. R2 Access Key / Secret Key 생성
7. OpenAI API Key 준비
8. Soniox API Key 준비
9. 결제 PG 계정 준비
10. 도메인 구매 또는 연결
11. 개인정보처리방침/이용약관 초안 준비

### Cursor가 해야 하는 일

1. FastAPI 프로젝트 생성
2. Dockerfile 작성
3. docker-compose.yml 작성
4. MariaDB 연결 코드 작성
5. SQLAlchemy 모델 작성
6. Alembic 마이그레이션 작성
7. R2 presigned upload 구현
8. Whisper 연동 service 작성
9. Soniox 연동 service 작성
10. Merge Engine 작성
11. GPT 요약/구간추천 service 작성
12. worker/job 구조 작성
13. 관리자 API 작성
14. 속기사 API 작성
15. 테스트 하네스 작성
16. Railway 배포용 설정 파일 작성

## 4. Cursor에게 실행시킬 초기 명령어

Windows PowerShell 기준이다.

### 프로젝트 기본 확인

```powershell
python --version
git --version
docker --version
docker compose version
```

### Python 가상환경

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
```

### 백엔드 의존성 설치

```powershell
pip install fastapi uvicorn[standard] sqlalchemy alembic pymysql pydantic-settings python-multipart httpx boto3 python-docx pytest pytest-asyncio ruff
pip freeze > backend/requirements.txt
```

Cursor가 폴더 생성 전이라면 먼저 아래 구조를 만든다.

```powershell
New-Item -ItemType Directory -Force backend, frontend, docs
New-Item -ItemType Directory -Force backend/app, backend/app/api, backend/app/core, backend/app/db, backend/app/services, backend/app/workers, backend/tests
```

### Alembic 초기화

```powershell
cd backend
alembic init alembic
cd ..
```

### 로컬 실행

```powershell
cd backend
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### 테스트

```powershell
cd backend
pytest
ruff check .
```

## 5. Docker 범위

Docker는 처음부터 너무 복잡하게 잡지 않는다.

### 1차 Docker 범위

로컬 개발용으로 아래만 Docker Compose로 묶는다.

```text
backend
mariadb
redis 또는 worker queue
```

프론트엔드는 초기에는 로컬 Node 개발 서버로 돌려도 된다. 백엔드 안정화 후 Docker에 포함한다.

### 2차 Docker 범위

Railway 배포용 Dockerfile은 backend 서비스 단위로 만든다.

```text
Dockerfile.backend
```

Railway에서는 MariaDB를 별도 서비스로 두고, backend는 Dockerfile로 배포한다.

### 피해야 할 것

```text
초기부터 Kubernetes 사용
초기부터 복잡한 multi-stage 최적화
DB 데이터를 컨테이너 내부에만 저장
.env 파일을 이미지 안에 복사
API 키를 Dockerfile ARG로 주입
```

Docker는 "같은 환경에서 실행되게 하는 도구"로만 시작한다. 운영 오케스트레이션은 Railway에 맡긴다.

## 6. 기본 docker-compose.yml 목표

Cursor는 아래 목적의 compose 파일을 만든다.

```text
docker compose up --build
```

실행 시 다음이 떠야 한다.

```text
FastAPI: http://localhost:8000
MariaDB: localhost:3306
Redis: localhost:6379
```

초기 서비스:

```yaml
services:
  backend:
    build:
      context: ./backend
    env_file:
      - .env
    ports:
      - "8000:8000"
    depends_on:
      - mariadb
      - redis

  mariadb:
    image: mariadb:11
    environment:
      MARIADB_ROOT_PASSWORD: local_root_password
      MARIADB_DATABASE: modiba_record
      MARIADB_USER: modiba
      MARIADB_PASSWORD: local_password
    ports:
      - "3306:3306"
    volumes:
      - mariadb_data:/var/lib/mysql

  redis:
    image: redis:7
    ports:
      - "6379:6379"

volumes:
  mariadb_data:
```

## 7. 환경변수

`.env.example`만 Git에 커밋한다. `.env`는 커밋하지 않는다.

```text
APP_ENV=local
APP_NAME=modiba-record
API_BASE_URL=http://localhost:8000

DATABASE_URL=mysql+pymysql://modiba:local_password@mariadb:3306/modiba_record

R2_ACCOUNT_ID=
R2_ACCESS_KEY_ID=
R2_SECRET_ACCESS_KEY=
R2_BUCKET_NAME=
R2_ENDPOINT_URL=

OPENAI_API_KEY=
OPENAI_TRANSCRIPTION_MODEL=
OPENAI_SUMMARY_MODEL=

SONIOX_API_KEY=
SONIOX_API_BASE_URL=

REDIS_URL=redis://redis:6379/0

JWT_SECRET_KEY=
SIGNED_URL_EXPIRE_SECONDS=900
```

## 8. Railway 세팅 계획

Railway에서는 아래 서비스로 나눈다.

```text
backend-api
mariadb
worker
```

초기에는 backend-api 안에서 간단한 background task를 쓰고, 실제 운영 전 worker를 별도 서비스로 분리한다.

### 사용자가 Railway에서 할 일

1. New Project 생성
2. GitHub repo 연결
3. MariaDB 템플릿 또는 Database 서비스 생성
4. Backend 서비스 생성
5. Variables 탭에 환경변수 등록
6. Public Domain 발급
7. Healthcheck path 설정: `/health`
8. Deploy 로그 확인

### Cursor가 준비할 Railway 파일

```text
Dockerfile
railway.toml
backend/app/main.py의 /health endpoint
```

### railway.toml 예시

```toml
[build]
builder = "DOCKERFILE"
dockerfilePath = "backend/Dockerfile"

[deploy]
healthcheckPath = "/health"
healthcheckTimeout = 100
restartPolicyType = "ON_FAILURE"
restartPolicyMaxRetries = 3
```

### Railway 환경변수

Railway Variables에 최소 아래 값을 넣는다.

```text
APP_ENV=production
DATABASE_URL=${{MariaDB.MARIADB_PRIVATE_URL}}
R2_ACCOUNT_ID=...
R2_ACCESS_KEY_ID=...
R2_SECRET_ACCESS_KEY=...
R2_BUCKET_NAME=...
R2_ENDPOINT_URL=...
OPENAI_API_KEY=...
SONIOX_API_KEY=...
JWT_SECRET_KEY=...
```

Railway는 Variables 탭에서 값을 넣고, 변경사항을 deploy에 반영해야 한다.

## 9. DB 세팅 계획

### 로컬

로컬은 Docker Compose의 MariaDB를 사용한다.

```powershell
docker compose up -d mariadb
cd backend
alembic upgrade head
```

### Railway

Railway MariaDB 서비스 생성 후 backend의 `DATABASE_URL`에 private URL을 연결한다.

운영 DB에는 직접 테이블을 만들지 않는다. 항상 Alembic migration으로 반영한다.

### 초기 테이블

```text
users
orders
files
ai_jobs
transcript_segments
ai_topics
review_flags
transcript_versions
deliveries
audit_logs
```

## 10. Whisper 연동 계획

Whisper 또는 OpenAI Transcription API 연동은 `services/stt_whisper.py`에 둔다.

초기에는 real API 호출 전에 mock harness를 만든다.

### service 인터페이스

```python
async def transcribe_with_whisper(file_path: str) -> WhisperTranscript:
    ...
```

### 반환 구조

```text
segments:
  - start_ms
  - end_ms
  - text
  - confidence
  - source=whisper
```

### 주의점

1. Whisper 결과는 화자분리 기준으로 쓰지 않는다.
2. 타임스탬프와 텍스트 초안 중심으로 사용한다.
3. 불확실한 단어는 review flag 후보로 남긴다.

## 11. Soniox 연동 계획

Soniox는 화자분리와 타임스탬프 보강에 사용한다.

Soniox 문서 기준으로 화자분리는 지원되며, 정확도는 비동기 transcription에서 더 유리하다. 따라서 녹음파일 업로드 서비스에서는 streaming보다 async transcription을 우선 고려한다.

### service 인터페이스

```python
async def transcribe_with_soniox(file_path: str) -> SonioxTranscript:
    ...
```

### 반환 구조

```text
segments:
  - start_ms
  - end_ms
  - text
  - speaker_label
  - confidence
  - overlap
  - source=soniox
```

## 12. Merge Engine 계획

Whisper와 Soniox 결과는 별도로 저장한 뒤 Python Merge Engine에서 합친다.

```text
Whisper: 텍스트 정확도/타임스탬프
Soniox: 화자분리/겹침/타임스탬프
GPT: 주제 요약/추천 구간/검토 포인트
```

초기 Merge 기준:

1. timestamp 기준으로 segment 정렬
2. 겹치는 구간을 하나의 normalized segment로 합침
3. Soniox speaker_label을 우선 적용
4. Whisper text와 Soniox text가 크게 다르면 review flag 생성
5. confidence가 낮으면 review flag 생성

## 13. 하네스 기본 개념

이 프로젝트에서 하네스는 "실제 API/운영 환경 없이도 전체 흐름을 검증하는 테스트 실행 틀"이다.

하네스가 필요한 이유:

```text
API 비용 절약
외부 API 장애와 무관한 개발
같은 샘플로 반복 테스트
Cursor가 기능을 고쳐도 회귀 확인 가능
Railway 배포 전 로컬 검증 가능
```

### 1차 하네스

```text
sample_audio_10min.mp3
mock_whisper_response.json
mock_soniox_response.json
expected_topics.json
```

명령어:

```powershell
cd backend
python -m app.harness.run_sample --mode mock --sample samples/sample_audio_10min.mp3
```

기대 결과:

```text
1. 주문 생성
2. 파일 등록
3. Whisper mock 로드
4. Soniox mock 로드
5. Merge Engine 실행
6. GPT summary mock 또는 real 실행
7. ai_topics 생성
8. quote 생성
9. review_flags 생성
```

### 2차 하네스

실제 API 키가 있을 때만 실행한다.

```powershell
cd backend
python -m app.harness.run_sample --mode real --sample samples/sample_audio_10min.mp3
```

### 테스트 기준

```text
mock mode는 항상 무료로 성공해야 한다.
real mode는 API 키가 없으면 skip한다.
샘플 결과는 JSON으로 저장한다.
비교 가능한 expected output을 둔다.
```

## 14. Cursor에게 줄 첫 작업 프롬프트

```text
이 프로젝트는 Python/FastAPI 기반 AI 녹취록 의뢰 플랫폼이다.

docs/01-mvp-project-plan.md
docs/02-cursor-agent-brief.md
docs/03-development-setup-and-cursor-plan.md

위 문서를 먼저 읽고, 다음 순서로 작업해라.

1. backend FastAPI 기본 구조를 만든다.
2. Dockerfile과 docker-compose.yml을 만든다.
3. /health API를 만든다.
4. SQLAlchemy + Alembic + MariaDB 연결을 만든다.
5. users, orders, files, ai_jobs, transcript_segments, ai_topics, review_flags 기본 모델을 만든다.
6. R2 presigned upload service 인터페이스를 만든다.
7. Whisper/Soniox 연동 service는 mock mode부터 만든다.
8. harness/run_sample.py를 만들어 mock 데이터로 전체 AI 파이프라인을 실행할 수 있게 한다.
9. pytest로 health, pricing, merge_engine, harness 테스트를 만든다.
10. 실제 외부 API 호출은 환경변수가 있을 때만 실행되게 한다.

작업 전에는 계획을 먼저 보여주고, 파일 생성 후에는 실행 명령과 검증 결과를 보고해라.
```

## 15. 1차 완료 기준

1. `docker compose up --build` 성공
2. `GET /health` 성공
3. `alembic upgrade head` 성공
4. `pytest` 성공
5. mock harness 성공
6. Railway 배포용 Dockerfile/railway.toml 존재
7. 실제 API 키 없이도 개발 가능
8. 실제 API 키를 넣으면 Whisper/Soniox real mode로 전환 가능

