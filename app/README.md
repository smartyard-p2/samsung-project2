# app: FastAPI 서비스

`app/`은 추천 엔진과 서비스 API를 묶고, 빌드된 `web/dist`를 같은 8000번 포트에서 제공한다.

## 시작

```bash
docker compose up -d --build --no-deps api
docker compose logs -f api
curl --fail http://localhost:8000/api/health
```

로컬 실행은 저장소 루트에서 한다.

```bash
source .venv/bin/activate
uvicorn app.main:app --reload --port 8001
```

로컬 실행도 `.env`, PostgreSQL·Redis·MinIO, `data/`의 모델 자산이 필요하다. CLIP을 처음 받으면 시작이 오래 걸린다.

## 주요 모듈

| 파일 | 역할 |
|---|---|
| `main.py` | 앱 수명주기, API 라우팅, 정적 화면 제공 |
| `recommender.py` | CLIP 이미지·텍스트 임베딩, 후보 30곳, 재정렬 |
| `llm.py` | Ollama 영어 시각 묘사와 한국어 규칙 조건 추출 |
| `context.py`, `regions.py` | 혼잡도·기후·거리·지역 조건 |
| `activities.py`, `courses.py`, `neighborhoods.py` | 관광지·축제·음식점·코스·읍면동 |
| `auth.py`, `personal.py` | 로그인, 저장, 피드백, 개인 취향 |
| `models.py`, `db.py`, `catalog.py` | SQLAlchemy 모델과 관광 콘텐츠 조회 |
| `media.py` | MinIO 사용자 사진과 프로필 사진 |
| `query_cache.py` | Redis 검색 결과 캐시와 메모리 대체 경로 |
| `schemas.py` | API 요청·응답 계약. `web/src/api.ts`와 함께 수정 |

## 추천 요청 흐름

1. 사진은 `get_image_features()`, 문장은 `get_text_features()`로 512차원 벡터가 된다.
2. L2 정규화된 국내 사진 벡터와 내적해 코사인 유사도를 구한다.
3. 관광지별 최고 사진 한 장을 남기고 상위 100개 유사도를 시군구별로 합산한다.
4. 상위 30개 후보 안에서 조건·반응·개인 취향으로 순서만 조정한다.

## 변경할 때 함께 확인할 파일

- API 응답 변경: `app/schemas.py`, `web/src/api.ts`, `tests/test_api.py`
- DB 변경: `app/models.py`, `infra/migrations/versions/`, `tests/test_infrastructure.py`
- 추천 변경: `app/recommender.py`, `src/model/`, 평가 JSON과 회귀 테스트
- 환경 변수 변경: `app/settings.py`, `docker-compose.yml`, 루트 README

`emb_kr.npz` 오류는 핵심 데이터 자산 누락인 경우가 많다. [`data/README.md`](../data/README.md)를 먼저 확인한다.
