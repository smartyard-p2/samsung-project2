# pickorea 실행·재현 가이드

이 저장소는 해외 여행지 사진과 분위기가 비슷한 국내 관광지를 추천하는 `pickorea`의 전체 소스 코드다. 이 문서는 처음 저장소를 받은 사람이 폴더 이름이나 개인 PC 경로에 의존하지 않고 개발 환경을 재현할 수 있도록 작성했다.

프로젝트 소개와 모델 결과보다 **설치와 실행 절차**가 목적이다. 서비스 구조와 실험 결과는 [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md), [`docs/HANDOFF.md`](docs/HANDOFF.md)를 참고한다.

## 1. 실행 전에 알아둘 점

- 저장소에는 소스 코드만 있다. `.env`, 원본 데이터, CLIP 임베딩, 사용자 업로드는 Git에 포함하지 않는다.
- 전체 서비스 실행에는 Docker Compose와 별도로 전달받거나 재생성한 `data/`가 필요하다.
- 저장소를 내려받을 때 로컬 폴더 이름은 자유롭게 정할 수 있다. 코드가 특정 사용자명이나 다운로드 폴더를 가정하지 않는다.
- `docker compose down -v`는 PostgreSQL·Redis·MinIO·Ollama 볼륨을 삭제한다. 데이터가 필요하면 실행하지 않는다.

## 2. 구성 요소

| 영역 | 기술 | 역할 |
|---|---|---|
| Web | React 19, TypeScript, Vite, Leaflet | 사진 검색, AI 여행, 지역 상세, MY |
| API | FastAPI, Pydantic, SQLAlchemy | 추천·회원·관광 콘텐츠 API와 정적 화면 제공 |
| 추천 | CLIP ViT-B/32, PyTorch | 이미지·문장을 512차원 벡터로 변환하고 유사도 계산 |
| 자연어 | Ollama, Qwen 2.5 3B | 한국어 요청을 영어 시각 묘사로 변환 |
| Data | PostgreSQL, Redis, MinIO | 서비스 데이터, 세션·캐시, 객체 저장 |
| Pipeline | Airflow, Alembic, MLflow | 수집·게시·DB 변경·실험 기록 |

## 3. 저장소 받기

아래 마지막 인수는 원하는 로컬 폴더 이름으로 바꿔도 된다.

```bash
git clone https://github.com/smartyard-p2/samsung-project2.git <원하는-폴더명>
cd <원하는-폴더명>
```

필수 프로그램:

- Docker Desktop 또는 Docker Engine + Compose v2
- Git
- 로컬 개발 시 Python 3.12, Node.js 24
- 첫 이미지 빌드와 모델 다운로드를 위한 인터넷 연결

권장 자원은 여유 디스크 20GB 이상, 메모리 16GB 이상이다. CLIP과 Ollama를 CPU로 실행하므로 첫 시작은 수 분 걸릴 수 있다.

## 4. 환경 변수 만들기

저장소 루트에 `.env`를 만들고 아래 예시를 복사한다. `change-me`는 각자 만든 값으로 바꾼다.

```dotenv
POSTGRES_PASSWORD=change-me-postgres
APP_DB_PASSWORD=change-me-app
AIRFLOW_DB_PASSWORD=change-me-airflow
MLFLOW_DB_PASSWORD=change-me-mlflow
MINIO_ROOT_USER=minioadmin
MINIO_ROOT_PASSWORD=change-me-minio
AIRFLOW_UID=1000
AIRFLOW_JWT_SECRET=change-me-with-a-long-random-string
OLLAMA_MODEL=qwen2.5:3b
OLLAMA_TIMEOUT_SECONDS=60
PUBLIC_BASE_URL=http://localhost:8000
COOKIE_SECURE=false

# 선택 기능. 사용하지 않으면 비워 둔다.
TOUR_API_KEY=
NAVER_CLIENT_ID=
NAVER_CLIENT_SECRET=
KAKAO_REST_API_KEY=
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
KAKAO_CLIENT_ID=
KAKAO_CLIENT_SECRET=
RESEND_API_KEY=
RESEND_FROM_EMAIL=
```

Linux·WSL에서는 다음 결과를 각각 `AIRFLOW_UID`, `AIRFLOW_JWT_SECRET`에 넣는다.

```bash
id -u
python -c "import secrets; print(secrets.token_urlsafe(48))"
```

## 5. 데이터 준비

폴더 구조는 Git에 있지만 데이터 본체는 없다. 팀에서 받은 데이터 묶음이 있다면 저장소 루트의 `data/` 아래에 그대로 풀어 놓는다. 임의의 상위 폴더 이름으로 한 번 더 감싸지 않도록 주의한다.

서비스 핵심 실행에 필요한 대표 자산:

```text
data/
├─ interim/clip/emb_kr.npz
├─ interim/clip/kr/
├─ interim/clip/emb_scenes.npz
├─ interim/clip/scene_recs.csv
├─ raw/tourapi/
├─ raw/datalab/locgoRegnVisitrDDList/
└─ external/
```

데이터는 두 방식으로 준비한다. 발표 결과를 그대로 실행하려면 프로젝트 관리자에게 최종 `data/` 묶음과 체크섬을 받고, 직접 재현하려면 공공데이터포털에서 TourAPI 등을 활용 신청한 뒤 수집 스크립트를 실행한다. 신청 링크, 정확한 명령, 일일 한도 대응과 수동 다운로드 항목은 [`data/README.md`](data/README.md)에 정리했다. `data/`가 불완전해도 일부 컨테이너는 뜰 수 있지만 추천 API와 지역 상세는 정상 동작하지 않는다.

## 6. 전체 서비스 실행

```bash
docker compose config --quiet
docker compose up -d ollama
docker compose exec ollama ollama pull qwen2.5:3b
docker compose up --build -d
docker compose ps -a
```

처음 받은 TourAPI 원본을 DB에 게시해야 하는 환경이라면 한 번 실행한다.

```bash
docker compose exec airflow-api-server bash -lc \
  'cd /workspace && python src/ingest/publish_tour.py --logical-date "$(date +%F)" --dag-id manual_backfill'
docker compose restart api
```

정상 여부:

```bash
curl --fail http://localhost:8000/api/health
docker compose exec postgres pg_isready -U postgres
docker compose exec redis redis-cli ping
docker compose exec ollama ollama list
```

| 화면 | 주소 |
|---|---|
| Web/API | `http://localhost:8000` |
| OpenAPI 문서 | `http://localhost:8000/docs` |
| Airflow | `http://localhost:8080` |
| MLflow | `http://localhost:5000` |
| MinIO Console | `http://localhost:9001` |

`migrate`, `airflow-init`, `mlflow-db-init`은 성공 후 `Exited (0)`이어야 한다. PostgreSQL·Redis·MLflow·Ollama는 `healthy`, API·Airflow·MinIO는 `Up` 상태가 정상이다.

## 7. 코드 수정 후 반영

```bash
docker compose up -d --build --no-deps api
docker compose logs --tail=120 api
```

DB 모델이나 마이그레이션을 바꿨다면:

```bash
docker compose run --rm migrate alembic -c infra/alembic.ini upgrade head
docker compose run --rm migrate alembic -c infra/alembic.ini check
```

## 8. 로컬 개발

API는 Docker로 실행한 상태에서 Vite 개발 서버만 따로 띄우는 방법이 가장 빠르다.

```bash
cd web
npm ci
npm run dev
```

다른 API 주소를 쓸 때만 Git에 올리지 않는 `web/.env.local`에 `VITE_API_TARGET`을 설정한다.

백엔드를 로컬 Python으로 실행하려면:

```bash
python3.12 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cd web && npm ci && npm run build && cd ..
uvicorn app.main:app --host 0.0.0.0 --port 8001
```

## 9. 테스트

```bash
.venv/bin/python -m pytest tests -q
cd web && npm run build && npm run lint
docker compose config --quiet
```

일부 추천·통합 테스트는 로컬 `data/`와 PostgreSQL을 사용한다. 필요한 자산이 없을 때의 실패를 코드 오류로 오해하지 말고 [`tests/README.md`](tests/README.md)의 전제 조건을 먼저 확인한다.

## 10. 자주 막히는 문제

| 증상 | 확인할 것 |
|---|---|
| API 시작이 오래 걸림 | 첫 CLIP 다운로드와 임베딩 로딩은 수십 초 이상 걸린다. `docker compose logs -f api` 확인 |
| `emb_kr.npz` 없음 | 핵심 CLIP 자산이 빠졌다. 데이터 묶음을 복원하거나 모델 스크립트로 재생성 |
| Airflow `PermissionError` | `.env`의 `AIRFLOW_UID`를 `id -u` 결과로 설정 |
| Airflow 서명 오류 | 모든 Airflow 컨테이너가 같은 `AIRFLOW_JWT_SECRET`을 쓰는지 확인 후 재생성 |
| OAuth callback 오류 | 공급자 콘솔 callback URL과 `PUBLIC_BASE_URL` 일치 확인 |
| 로그인 쿠키가 남지 않음 | HTTPS 운영은 `COOKIE_SECURE=true`; iframe 안에서는 새 창으로 서비스 열기 |
| Ollama 연결 실패 | `docker compose exec ollama ollama list`, 모델명과 `OLLAMA_MODEL` 확인 |
| 빈 관광 콘텐츠 | `publish_tour.py` 백필과 PostgreSQL의 `places` 건수 확인 |
| 포트 충돌 | 8000·8080·5000·5434·6379·9000·9001·11434 사용 프로세스 확인 |

## 11. 프로젝트 구조

```text
app/                 FastAPI, 추천, 인증, 지역 상세, 캐시
web/                 React + TypeScript 화면
src/collect/         외부 데이터 수집
src/model/           CLIP 임베딩·추천 평가
src/forecast/        혼잡도 예측
src/ingest/          MinIO·PostgreSQL 게시
src/ops/             보관 기한 정리 작업
infra/               Docker, Airflow, Alembic, PostgreSQL, MLflow
tests/               API·인프라·LLM·MLflow 테스트
tools/               평가와 발표자료 보조 도구
docs/                설계, 운영, 실험 근거
data/                로컬 데이터, Git에는 구조만 포함
```

세부 설명은 각 폴더 README에서 이어진다: [`app`](app/README.md), [`web`](web/README.md), [`src`](src/README.md), [`infra`](infra/README.md), [`data`](data/README.md), [`tests`](tests/README.md), [`tools`](tools/README.md), [`docs`](docs/README.md).

## 12. 종료와 백업

```bash
docker compose stop
docker compose start
```

운영 데이터를 보존하려면 PostgreSQL과 MinIO를 같은 시점에 백업한다. 상세 절차는 [`docs/INFRASTRUCTURE.md`](docs/INFRASTRUCTURE.md)를 따른다.
