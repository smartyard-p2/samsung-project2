# infra: 배포·운영 구성

전체 스택은 저장소 루트의 `docker-compose.yml`로 실행한다. 상세 운영 기준은 [`docs/INFRASTRUCTURE.md`](../docs/INFRASTRUCTURE.md)에 있다.

| 경로 | 역할 |
|---|---|
| `docker/` | Airflow·MLflow·MinIO 이미지 |
| `airflow/dags/` | 수집·게시·정리 일정 |
| `migrations/` | Alembic DB 변경 이력 |
| `postgres/init/` | 빈 볼륨 최초 DB·계정 생성 |
| `mlflow/` | MLflow 서버 시작 코드 |

## 기본 명령

```bash
docker compose config --quiet
docker compose up --build -d
docker compose ps -a
docker compose logs --tail=200 api
```

DB 스키마:

```bash
docker compose run --rm migrate alembic -c infra/alembic.ini current
docker compose run --rm migrate alembic -c infra/alembic.ini upgrade head
docker compose run --rm migrate alembic -c infra/alembic.ini check
```

Airflow:

```bash
docker compose exec airflow-api-server airflow dags list
docker compose exec airflow-api-server airflow jobs check --job-type SchedulerJob --allow-multiple --limit 100
```

## 운영 전제

- Airflow 시간대는 `Asia/Seoul`이다.
- `AIRFLOW_UID`는 Linux·WSL의 `id -u` 결과를 사용한다.
- scheduler와 API server는 같은 `AIRFLOW_JWT_SECRET`을 사용한다.
- Redis와 Ollama는 인증 없이 로컬 주소에만 공개된다.
- HTTPS 운영에서는 `PUBLIC_BASE_URL`을 실제 도메인으로 바꾸고 `COOKIE_SECURE=true`로 설정한다.
- PostgreSQL과 MinIO는 같은 복구 시점으로 백업한다.

`docker compose down -v`는 서비스 데이터를 삭제하므로 초기화를 의도한 경우에만 사용한다.
