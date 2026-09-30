# project-2

삼성중공업 스마트 조선소 AI전문가 양성과정 2차 프로젝트

AI 분석·모델링 결과를 웹으로 제공하는 팀 프로젝트다.

- **프로젝트 기간:** 2026년 10월 1일 ~ 10월 16일
- **현재 상태:** 🔍 주제·데이터셋 선정 중 (확정 목표: **10월 2일**)

> 주제가 확정되면 이 문서의 "프로젝트 개요", "기술 스택", "실행 방법"을 채운다.

---

## 처음 오셨나요? (팀원 필독)

Git이 처음이어도 괜찮습니다. 아래 순서대로만 하면 됩니다.

1. **[`docs/GIT_GUIDE.md`](docs/GIT_GUIDE.md) → 1. 최초 1회 세팅** 을 따라 환경을 만든다.
2. **[`AGENTS.md`](AGENTS.md) → 0장 "항상 지킬 10가지"** 를 읽는다. (1분)
3. **[`docs/tasks/`](docs/tasks/)** 에서 본인 역할 문서를 읽는다.
4. 작업 시작. 막히면 혼자 명령어를 더 시도하지 말고 [GIT_GUIDE 10번](docs/GIT_GUIDE.md#10-막혔을-때)을 본다.

---

## 문서 안내

| 문서 | 내용 | 언제 보는가 |
|---|---|---|
| [`AGENTS.md`](AGENTS.md) | 전원 공통 규칙 (사람 + AI 도구) | 작업 전 |
| [`docs/GIT_GUIDE.md`](docs/GIT_GUIDE.md) | Git 명령어, 충돌 해결, 실수 복구 | Git이 막힐 때 |
| [`docs/DECISIONS.md`](docs/DECISIONS.md) | 결정 기록과 보류 중인 질문 | 결정할 때 / 발표 준비 |
| [`docs/tasks/`](docs/tasks/) | 역할별 작업 가이드 4개 | 작업 전 |
| [`data/README.md`](data/README.md) | 데이터 폴더 규칙, 출처 기록 | 데이터 추가 시 |
| [`models/README.md`](models/README.md) | 모델 저장 규칙, 성능 기록 | 모델 저장 시 |

> 모든 문서는 **`main` 브랜치의 것이 기준**이다. 역할 브랜치에 문서 사본을 만들지 않는다.

---

## 팀 구성과 브랜치

| 브랜치 | 역할 | 담당자 | 작업 가이드 |
|---|---|---|---|
| `main` | 검토가 끝난 통합 결과 | 조장 (관리) | — |
| `eda` | EDA·전처리 | 이주연, 최인영, 김현수 | [EDA_TASKS](docs/tasks/EDA_TASKS.md) |
| `modeling` | 모델링·평가 | 최진실, 하지훈 | [MODELING_TASKS](docs/tasks/MODELING_TASKS.md) |
| `web` | 프론트엔드·백엔드 | 조장 | [WEB_TASKS](docs/tasks/WEB_TASKS.md) |
| `data-infra` | 데이터 인프라·DB·배포 | 조장 | [DATA_INFRA_TASKS](docs/tasks/DATA_INFRA_TASKS.md) |

**협업 흐름**

```
eda / modeling / web / data-infra  ──[Pull Request + 조장 승인]──▶  main
                 ▲
                 └── 매일 아침 1회:  git merge origin/main
```

- `main`에 직접 Push하지 않는다. 반드시 Pull Request.
- **매일 아침 `main` 동기화는 필수다.** 미루면 마지막 날 통합이 불가능해진다.
- 자세한 규칙: [`AGENTS.md` 12장](AGENTS.md#12-git-및-협업-규칙)

---

## 일정

| 날짜 | 마일스톤 | 완료 기준 |
|---|---|---|
| **10/1 (목)** | 환경 세팅 | 전원 clone·브랜치 체크아웃·패키지 설치 완료, 후보 주제 3개 이상 정리 |
| **10/2 (금)** | 🚩 **주제·데이터셋 확정** | 데이터 실물 확보, 타깃·문제 유형 결정, `DECISIONS.md`에 기록 |
| **10/7 (수)** | EDA 1차 완료 | 데이터 품질 점검 끝, 컬럼 설명서 작성, **모델링팀에 전처리 데이터 전달** |
| **10/9 (금)** | 기준 모델 + API 명세 | baseline 성능 확보, 프론트-백엔드 요청·응답 형식 합의 |
| **10/13 (화)** | 개선 모델 완료 | 최종 모델 확정, 예측 함수 제공, **웹 연동 시작** |
| **10/15 (목)** | 통합 완료 | 전체 `main` 병합, 웹에서 실제 결과 표시, 발표 자료 초안 |
| **10/16 (금)** | 최종 점검·발표 | 새 환경에서 재현 확인, 임시 코드 제거, 발표 |

> ⏰ **10/2 주제 확정이 가장 중요한 데드라인이다.** 전체 기간이 2주뿐이라 여기서 하루가 밀리면 뒷단 전체가 밀린다.
>
> ⚠️ 10/5(월)·10/9(금)은 공휴일 여부를 확인해서 일정을 조정한다. 확정되면 이 표를 수정한다.

---

## 폴더 구조

```
samsung-project2/
├── AGENTS.md                # 공통 작업 규칙
├── README.md                # 이 문서
├── requirements.txt         # Python 의존성
├── .env.example             # 환경변수 예시 (실제 .env는 커밋 금지)
├── .gitignore
│
├── docs/
│   ├── GIT_GUIDE.md         # Git 실전 가이드
│   ├── DECISIONS.md         # 의사결정 기록
│   └── tasks/               # 역할별 작업 가이드
│
├── data/                    # ⚠️ 데이터 본체는 Git에 올리지 않음
│   ├── raw/                 #   원본 (수정 금지)
│   ├── external/            #   외부 참고 데이터
│   ├── interim/             #   전처리 중간 결과
│   └── processed/           #   학습용 최종 데이터
│
├── notebooks/               # 분석·실험 노트북 (1인 1파일)
├── src/                     # 재사용 코드 (전처리, 학습, 예측)
├── models/                  # ⚠️ 모델 파일은 Git에 올리지 않음
├── reports/figures/         # 그래프 이미지
│
├── backend/                 # 백엔드
└── front/                   # 프론트엔드
```

**파일명 규칙**

| 대상 | 규칙 | 예시 |
|---|---|---|
| 노트북 | `<번호>_<용도>_<이름>.ipynb` (1인 1파일) | `01_eda_이주연.ipynb` |
| 재사용 코드 | 소문자 + 밑줄 | `src/preprocess.py` |
| 모델 파일 | `<모델>_<버전>.pkl` | `rf_baseline_v1.pkl` |
| 그래프 | `<주제>_<내용>.png` | `target_distribution.png` |

---

## 실행 방법

### 사전 요구사항

| 항목 | 버전 |
|---|---|
| Python | **3.11 / 3.12 / 3.13** — ⚠️ 3.14 이상은 설치 실패 |
| Node.js | _(웹 스택 확정 후 기재)_ |
| DB | _(사용 여부 확정 후 기재)_ |

### 1. 저장소 준비

```bash
git clone https://github.com/smartyard-p2/samsung-project2.git
cd samsung-project2
git switch <본인-역할-브랜치>
```

### 2. Python 환경

> ⚠️ **파이썬 버전을 먼저 확인하세요.** `python --version`이 3.14 이상이면 패키지 설치가 실패합니다.
> python.org에서 최신 버전을 그대로 받으면 3.14 이상이 설치됩니다. **3.12 또는 3.13**을 골라 설치하세요.

```bash
python --version        # 3.11 / 3.12 / 3.13 인지 확인

python -m venv .venv

# Windows (PowerShell)
.venv\Scripts\Activate.ps1
# Mac / Linux
source .venv/bin/activate

pip install -r requirements.txt
```

### 3. 환경변수

```bash
cp .env.example .env        # Windows: copy .env.example .env
```

`.env` 안의 값을 채운다. 이 파일은 커밋되지 않는다.

### 4. 데이터 준비

데이터 파일은 Git에 없다. 팀 공유 드라이브에서 받아 `data/raw/`에 넣는다.
(공유 위치는 주제 확정 후 조장이 안내)

### 5. 노트북 실행

```bash
jupyter lab
```

### 6. 웹 실행

_(스택 확정 후 작성)_

---

## 프로젝트 개요

_(주제 확정 후 작성: 문제 정의, 데이터, 접근 방법, 결과)_

## 기술 스택

_(확정 후 작성)_

## 결과

_(작성 예정)_
