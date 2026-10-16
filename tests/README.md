# tests: 검증 방법

```bash
.venv/bin/python -m pytest tests -q
cd web && npm run build && npm run lint
docker compose config --quiet
```

테스트 개수는 기능 추가에 따라 달라지므로 README에 고정하지 않는다. 실행 결과의 수집 수와 통과·실패·건너뜀 수를 기준으로 판단한다.

| 파일 | 확인 내용 |
|---|---|
| `test_api.py` | API 계약, 추천 회귀, 조건·개인화, AI 여행 규칙 |
| `test_infrastructure.py` | DB 스키마, 인증 기본 함수, 캐시 대체, 게시 구조 |
| `test_email_worker.py` | 메일 작업 검증과 오류 처리 |
| `test_llm.py` | LLM 출력 형식과 규칙 결합 |
| `test_mlflow_tracking.py` | 실험 기록 실패 격리와 메타데이터 |

추천·지역 상세 통합 테스트는 `data/` 자산과 PostgreSQL 백필이 필요하다. Ollama 경로 검증은 모델이 내려받아져 있어야 한다. 실패 시 데이터 파일 없음, DB 연결 실패, 실제 코드 회귀를 구분해 기록한다.
