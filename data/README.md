# data: 로컬 데이터 규칙

데이터 본체는 크기·라이선스·비밀정보 때문에 Git에 올리지 않는다. 저장소에는 폴더 구조만 유지한다.

| 폴더 | 내용 | 취급 |
|---|---|---|
| `raw/` | API 응답과 내려받은 원본 | 수정·덮어쓰기 금지 |
| `external/` | 외부 표, 수동 검수 평가셋 | 출처와 라이선스 필수 |
| `interim/` | 임베딩·캐시·가공 중간 결과 | 코드로 재생성 가능해야 함 |
| `processed/` | 모델·분석 입력용 최종 결과 | 생성 코드와 함께 관리 |

## 서비스 핵심 자산

```text
interim/clip/emb_kr.npz             국내 관광 사진 CLIP 벡터
interim/clip/kr/                    국내 관광 사진 캐시
interim/clip/emb_scenes.npz         탐색용 해외 장면 벡터
interim/clip/scene_recs.csv         장면별 추천 결과
interim/app/                        경비·동네·사진 선택 캐시
raw/tourapi/                        관광지·사진·축제·코스 원본
raw/datalab/locgoRegnVisitrDDList/  방문자 수 원본
external/                           평가 정답·공휴일·장면 메타데이터
```

## 재생성 흐름

1. `src/collect/`에서 원본을 `raw/`에 수집한다.
2. `src/model/clip_proto.py`와 `scene_catalog.py`로 사진과 임베딩을 만든다.
3. `src/forecast/`와 `src/cost/`로 혼잡도·경비 결과를 만든다.
4. `src/ingest/publish_tour.py`로 원본을 MinIO와 PostgreSQL에 게시한다.
5. API를 재시작해 파일 기반 인덱스를 다시 읽는다.

날짜가 포함된 실제 데이터 디렉터리는 수집 시점에 따라 달라질 수 있으므로 다른 사람의 파일명을 그대로 만들지 않는다.

## 공유 체크리스트

- 압축을 풀었을 때 최상위가 바로 `data/raw`, `data/interim`인지 확인한다.
- API 키, 이메일, 사용자 업로드, `.env`를 포함하지 않는다.
- 사진마다 사용 허가 코드·작가·원본 URL을 보존한다.
- 수동 작성 평가셋은 버전과 dev·holdout 구분을 기록한다.
- 공유 전 체크섬과 생성 일자를 함께 전달한다.

원본 출처는 한국관광공사 TourAPI·DataLab, Open-Meteo, 행정안전부, 한국천문연구원, Wikimedia Commons 등이다. 실제 사용 시 각 응답과 사진에 기록된 라이선스를 우선한다.
