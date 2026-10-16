# src: 데이터·모델 파이프라인

`src/`는 서버 요청 중 실행하는 코드가 아니라 데이터 수집, 임베딩 생성, 평가, DB 게시에 사용하는 배치 스크립트다. 모든 명령은 저장소 루트에서 실행한다.

| 폴더 | 역할 |
|---|---|
| `collect/` | TourAPI·DataLab·날씨·인구·음식점 사진 수집 |
| `model/` | CLIP 임베딩, 후보 집계, 모델 비교, 홀드아웃 평가 |
| `forecast/` | 월·일 혼잡도 기준선과 모델 비교 |
| `cost/` | 국민여행조사 기반 예상 경비 표 생성 |
| `ingest/` | 원본을 MinIO에 보관하고 PostgreSQL에 멱등 게시 |
| `mlops/` | MLflow 기록 공통 함수 |
| `ops/` | 만료된 업로드·보관 데이터 정리 |

## 대표 실행 순서

```bash
.venv/bin/python src/collect/tour_attractions.py
.venv/bin/python src/model/clip_proto.py download
.venv/bin/python src/model/clip_proto.py embed
.venv/bin/python src/model/clip_proto.py evaluate
.venv/bin/python src/model/holdout_pilot.py
.venv/bin/python src/forecast/p1_congestion.py
.venv/bin/python src/ingest/publish_tour.py --logical-date "$(date +%F)" --dag-id manual
```

각 스크립트의 정확한 인수는 파일 상단 docstring과 `--help`를 먼저 확인한다.

## 재현 규칙

- `data/raw`는 덮어쓰거나 수동 수정하지 않는다.
- 변환 결과는 `data/interim` 또는 `data/processed`에 쓴다.
- 날짜가 들어간 입력 경로는 새 데이터 수집 시 설정이나 인수로 교체한다.
- 평가셋과 합격선은 결과 확인 전에 고정한다.
- 추천 방식을 바꾸면 Hit@K·MRR, 사람 평가, 편향을 함께 확인한다.
- NPZ 키나 인덱스 계약을 바꾸면 `app/recommender.py`도 함께 수정한다.

현재 서비스는 CLIP ViT-B/32를 추가 학습 없이 사용한다. 국내 사진은 미리 벡터화하고, 사용자 사진이나 영어 시각 묘사는 요청 시 벡터화한다.
