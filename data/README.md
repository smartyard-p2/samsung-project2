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

## 데이터를 받는 방법

Git 저장소에는 대용량 원본·사진·임베딩이 포함되지 않는다. 다음 두 방법 중 하나를 선택한다.

### 방법 A: 준비된 데이터 묶음 사용

발표 결과를 그대로 실행하려면 프로젝트 관리자에게 **최종 `data/` 묶음**과 SHA-256 체크섬을 전달받는 것이 가장 빠르다. 현재 이 저장소에는 공개 다운로드 주소가 없으므로 Git clone만으로 추천 기능까지 실행되지는 않는다.

압축 파일 이름과 저장 위치는 자유롭게 정한다. 압축을 풀었을 때 저장소 안의 경로가 `data/raw`, `data/external`, `data/interim`으로 바로 이어져야 한다.

```bash
# 저장소 루트에서 실행하는 예시. 실제 압축 파일 형식에 맞는 명령을 사용한다.
tar -xf <데이터-압축파일>.tar.gz
find data -maxdepth 2 -type d | sort
```

Windows에서는 탐색기의 **압축 풀기** 또는 PowerShell의 `Expand-Archive`를 사용해도 된다. `data/data/raw`처럼 `data`가 두 번 생겼다면 안쪽 `data`의 내용을 저장소 루트 `data/`로 옮긴다.

받은 뒤 최소 자산을 확인한다.

```bash
test -f data/interim/clip/emb_kr.npz
test -d data/interim/clip/kr
test -d data/raw/tourapi
test -d data/raw/datalab/locgoRegnVisitrDDList
```

### 방법 B: 공식 출처에서 직접 수집·생성

1. [공공데이터포털](https://www.data.go.kr/) 계정을 만들고 다음 활용 신청을 한다.
   - [한국관광공사 국문 관광정보 서비스 TourAPI](https://www.data.go.kr/data/15101578/openapi.do): 관광지·사진·음식점·축제·여행코스
   - 한국관광공사 관광빅데이터 지역별 방문자 수: 혼잡도 학습 데이터. 포털에서 `locgoRegnVisitrDDList`로 검색한다.
   - [한국천문연구원 특일 정보](https://www.data.go.kr/data/15012690/openapi.do): 공휴일 데이터
   - 행정안전부 행정동별 주민등록 인구: 포털에서 `admmPpltnHhStus`로 검색한다.
2. 발급된 일반 인증키를 저장소 루트 `.env`에 넣는다. 포털에 표시된 값을 다시 URL 인코딩하지 않는다.

```dotenv
TOUR_API_KEY=<발급받은-일반-인증키>
```

TourAPI 목록 외에 법정동 코드와 관광 분류 코드 원문도 필요하다. 아래 명령은 실행한 날짜로 파일명을 자동 생성한다. `.env` 값을 셸 변수로 직접 불러오기 어려우면 첫 줄의 값만 발급받은 키로 바꾼다.

```bash
export TOUR_API_KEY='<발급받은-일반-인증키>'
STAMP=$(date +%Y%m%d)
mkdir -p data/raw/tourapi

curl --fail --get "https://apis.data.go.kr/B551011/KorService2/ldongCode2?serviceKey=${TOUR_API_KEY}" \
  --data-urlencode 'MobileOS=ETC' --data-urlencode 'MobileApp=pickorea' \
  --data-urlencode '_type=json' --data-urlencode 'numOfRows=1000' --data-urlencode 'pageNo=1' \
  -o "data/raw/tourapi/ldongCode2_list_${STAMP}.json"

curl --fail --get "https://apis.data.go.kr/B551011/KorService2/lclsSystmCode2?serviceKey=${TOUR_API_KEY}" \
  --data-urlencode 'MobileOS=ETC' --data-urlencode 'MobileApp=pickorea' \
  --data-urlencode '_type=json' --data-urlencode 'numOfRows=1000' --data-urlencode 'pageNo=1' \
  -o "data/raw/tourapi/lclsSystmCode2_${STAMP}.json"
```

3. Python 환경을 만들고 기본 원본을 수집한다. 아래 명령이 날짜와 content id를 사용해 출력 이름을 자동으로 만든다. 다른 사람의 날짜나 파일명을 직접 따라 만들 필요가 없다.

```bash
python3.12 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

python src/collect/tour_attractions.py 12  # 관광지
python src/collect/tour_attractions.py 28  # 레포츠
python src/collect/tour_attractions.py 39  # 음식점
python src/collect/tour_attractions.py 25  # 여행코스
python src/collect/datalab_visitors.py 201801 202608
python src/collect/population.py <수집할-YYYYMM>
```

4. 목록 수집이 끝나면 상세 정보와 사진을 이어받는다. TourAPI 일일 한도에 걸리면 다음 날 같은 명령을 다시 실행한다. 이미 저장된 항목은 건너뛴다.

```bash
python src/collect/tour_images.py
python src/collect/tour_images.py --food
python src/collect/tour_food_intro.py
python src/collect/tour_courses.py
python src/collect/tour_course_stops.py
```

5. 날씨와 해외 장면 사진을 받고 CLIP 임베딩을 생성한다. Open-Meteo 날씨 수집에는 API 키가 필요하지 않다. CLIP 모델과 Wikimedia Commons 사진을 처음 받을 때 인터넷 연결이 필요하다.

```bash
python src/collect/region_context.py climate 1 2 3 4 5 6 7 8 9 10 11 12
python src/model/clip_proto.py kr_images ov_images embed
python src/model/scene_catalog.py download embed recommend
```

국민여행조사 기반 경비 데이터가 필요하면 [국민여행조사 원자료 다운로드](https://know.tour.go.kr/stat/nRawDataDownloadDis19Re.do)에서 2023~2025 국내여행 SAV 파일을 각각 내려받는다. 폴더 이름은 자유롭게 정하고 그 폴더를 명령의 마지막 인수로 전달한다.

```bash
pip install pyreadstat
python src/cost/build_cost_table.py <국민여행조사-원자료-폴더>
```

행정동 경계가 필요하면 [vuski/admdongkor 릴리스](https://github.com/vuski/admdongkor/releases)에서 사용할 버전의 GeoJSON을 받고, 출처와 버전을 함께 기록한다. 평가 정답표와 수동 검수 상태는 외부 API로 재생성할 수 없으므로 저장소에 포함된 표를 사용하거나 새 평가셋을 별도로 작성한다.

### 새로 수집할 때 주의할 점

- 수집기가 만든 날짜 폴더와 원본 파일명은 바꾸지 않는다. 개인이 정하는 이름은 저장소 폴더와 전달용 압축 파일 이름이다.
- 일부 실험 스크립트의 상단에는 발표 당시 데이터 날짜가 고정되어 있다. 새 데이터로 재현할 때는 `RAW_TOUR`, `RAW_LIST`, `FOOD_LIST`, `LDONG` 상수가 새로 생성된 경로를 가리키는지 확인한다.
- TourAPI의 목록, 추가 사진, 소개 정보는 호출 한도가 서로 다를 수 있다. 오류 메시지가 `LIMITED_NUMBER_OF_SERVICE_REQUESTS`이면 파일을 지우지 말고 다음 날 이어서 실행한다.
- 사진은 응답의 `cpyrhtDivCd`를 확인한다. 이 프로젝트는 공공누리 1·3유형만 사용하며, 3유형 이미지는 자르거나 편집하지 않는다.

## 재생성 흐름

1. `src/collect/`에서 원본을 `raw/`에 수집한다.
2. `src/model/clip_proto.py`와 `scene_catalog.py`로 사진과 임베딩을 만든다.
3. `src/forecast/`와 `src/cost/`로 혼잡도·경비 결과를 만든다.
4. `src/ingest/publish_tour.py`로 원본을 MinIO와 PostgreSQL에 게시한다.
5. API를 재시작해 파일 기반 인덱스를 다시 읽는다.

날짜가 포함된 실제 데이터 디렉터리는 수집기가 자동으로 만든다. 다른 사람의 날짜를 파일명에 복사하지 않는다.

## 공유 체크리스트

- 압축을 풀었을 때 최상위가 바로 `data/raw`, `data/interim`인지 확인한다.
- API 키, 이메일, 사용자 업로드, `.env`를 포함하지 않는다.
- 사진마다 사용 허가 코드·작가·원본 URL을 보존한다.
- 수동 작성 평가셋은 버전과 dev·holdout 구분을 기록한다.
- 공유 전 체크섬과 생성 일자를 함께 전달한다.

원본 출처는 한국관광공사 TourAPI·DataLab, Open-Meteo, 행정안전부, 한국천문연구원, Wikimedia Commons 등이다. 실제 사용 시 각 응답과 사진에 기록된 라이선스를 우선한다.
