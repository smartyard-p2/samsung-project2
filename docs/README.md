# docs: 문서 읽는 순서

| 순서 | 문서 | 내용 |
|---|---|---|
| 1 | [`../README.md`](../README.md) | 새 환경 설치·데이터 준비·실행·문제 해결 |
| 2 | [`ARCHITECTURE.md`](ARCHITECTURE.md) | 데이터 → 모델 → API → 화면 구조 |
| 3 | [`INFRASTRUCTURE.md`](INFRASTRUCTURE.md) | Docker, DB, Airflow, MLflow, 백업 운영 |
| 4 | [`HANDOFF.md`](HANDOFF.md) | 실험 과정, 결정 근거, 실제 결과 수치 |
| 5 | [`MVP_PLAN.md`](MVP_PLAN.md) | 화면 흐름과 API 계약 |
| 6 | [`PERSONALIZATION_AND_DB_PLAN.md`](PERSONALIZATION_AND_DB_PLAN.md) | 사용자별 재정렬과 DB 설계 |

문서와 코드가 다르면 현재 코드와 마이그레이션을 우선한다. 수치나 운영 절차를 바꿀 때는 관련 README와 근거 문서를 같은 커밋에서 갱신한다.

공유할 새 문서는 개인 경로 대신 이 폴더 아래에서 목적이 드러나는 이름을 정해 추가한다.
