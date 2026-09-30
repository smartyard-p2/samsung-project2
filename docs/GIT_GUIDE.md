# Git 실전 가이드

Git을 처음 쓰는 팀원을 위한 문서다. **명령어를 그대로 복사해서 쓰면 된다.**

- 규칙과 원칙은 [`AGENTS.md`](../AGENTS.md)에 있다. 이 문서는 "어떻게 하는지"만 다룬다.
- 막히면 혼자 해결하려 하지 말고 [10. 막혔을 때](#10-막혔을-때)를 먼저 읽는다.

> **가장 중요한 습관 2가지**
> 1. 작업 시작 전에 **항상** `git pull`
> 2. 명령어를 치기 전에 **항상** `git status`로 지금 어디에 있는지 확인

---

## 목차

1. [최초 1회 세팅](#1-최초-1회-세팅)
2. [매일 작업 시작할 때](#2-매일-작업-시작할-때)
3. [작업 내용 저장하고 올리기](#3-작업-내용-저장하고-올리기)
4. [main의 최신 내용 내려받기 (매일 1회)](#4-main의-최신-내용-내려받기-매일-1회)
5. [Pull Request 만들기](#5-pull-request-만들기)
6. [push가 거부됐을 때](#6-push가-거부됐을-때)
7. [충돌(conflict)이 났을 때](#7-충돌conflict이-났을-때)
8. [실수 복구 모음](#8-실수-복구-모음)
9. [절대 하지 말 것](#9-절대-하지-말-것)
10. [막혔을 때](#10-막혔을-때)
11. [명령어 빠른 참조](#11-명령어-빠른-참조)

---

## 1. 최초 1회 세팅

### 1-1. 저장소 내려받기

```bash
git clone https://github.com/smartyard-p2/samsung-project2.git
cd samsung-project2
```

### 1-2. 이름과 이메일 설정

커밋에 기록되는 정보다. GitHub 계정과 같은 이메일을 쓴다.

```bash
git config user.name "본인이름"
git config user.email "github에등록한@이메일.com"
```

### 1-3. 줄바꿈 설정 (Windows 사용자만)

Windows와 Mac의 줄바꿈 방식이 달라서 생기는 가짜 변경사항을 막는다.

```bash
git config core.autocrlf true
```

### 1-4. 본인 역할 브랜치로 이동

| 역할 | 담당자 | 명령어 |
|---|---|---|
| EDA·전처리 | 이주연, 최인영, 김현수 | `git switch eda` |
| 모델링 | 최진실, 하지훈 | `git switch modeling` |
| 웹 | 조장 | `git switch web` |
| 데이터 인프라 | 조장 | `git switch data-infra` |

```bash
git switch eda        # 예시: EDA 담당
git branch            # * 표시가 eda 에 있으면 성공
```

### 1-5. 실행 환경 만들기

```bash
python -m venv .venv

# Windows (PowerShell)
.venv\Scripts\Activate.ps1
# Mac / Linux
source .venv/bin/activate

pip install -r requirements.txt
```

### 1-6. 환경변수 파일 준비

```bash
cp .env.example .env     # Windows: copy .env.example .env
```

`.env`는 커밋되지 않는다. 본인 PC에만 있는 파일이다.

---

## 2. 매일 작업 시작할 때

**매번 이 3줄로 시작한다.** 이것만 지켜도 충돌 대부분이 사라진다.

```bash
git switch eda          # 본인 역할 브랜치 (본인 것으로 바꿔서)
git pull                # 팀원이 올린 최신 내용 받기
git status              # 깨끗한지 확인
```

`git status`에 `nothing to commit, working tree clean`이 나오면 정상이다.

> `git pull` 에서 에러가 나면 → [7. 충돌](#7-충돌conflict이-났을-때)

---

## 3. 작업 내용 저장하고 올리기

### 3-1. 무엇이 바뀌었는지 확인

```bash
git status              # 바뀐 파일 목록
git diff                # 바뀐 내용 자세히
```

### 3-2. 올릴 파일 고르기

```bash
git add notebooks/eda_이주연.ipynb     # 파일을 하나씩 지정 (권장)
```

> **주의:** `git add .` 은 되도록 쓰지 않는다. 의도하지 않은 파일(대용량 데이터, `.env`)이 같이 올라가는 사고가 여기서 난다.
> 꼭 쓸 거라면 먼저 `git status`로 목록을 눈으로 확인한다.

### 3-3. 커밋하기

```bash
git commit -m "feat(eda): 결측치 처리 함수 추가"
```

**커밋 메시지 형식**

```
<타입>(<범위>): <무엇을 했는지 한 줄>
```

| 타입 | 언제 | 예시 |
|---|---|---|
| `feat` | 기능·분석·모델을 새로 추가 | `feat(modeling): 랜덤포레스트 기준 모델 추가` |
| `fix` | 잘못된 것을 고침 | `fix(eda): 날짜 파싱 오류 수정` |
| `docs` | 문서만 수정 | `docs: EDA 작업 가이드 보완` |
| `refactor` | 동작은 같고 코드 정리 | `refactor(src): 전처리 함수 분리` |
| `chore` | 설정·패키지·기타 | `chore: seaborn 의존성 추가` |
| `data` | 데이터 관련 작업 | `data: 원본 데이터 출처 기록` |

범위(`eda`, `modeling`, `web`, `data-infra`, `src`, `docs`)는 생략해도 된다.

**좋은 예 / 나쁜 예**

| 나쁜 예 | 왜 | 좋은 예 |
|---|---|---|
| `수정` | 무엇을 고쳤는지 알 수 없다 | `fix(eda): 이상치 기준 IQR로 변경` |
| `ㅇㅇ`, `.`, `asdf` | 의미가 없다 | `feat(eda): 컬럼별 분포 시각화 추가` |
| `오늘 작업 전부` | 여러 작업이 섞여 되돌리기 불가능 | 작업 단위로 나눠서 여러 번 커밋 |

### 3-4. 올리기

```bash
git push
```

처음 push할 때 브랜치 연결 에러가 나면:

```bash
git push -u origin eda      # 본인 브랜치 이름으로
```

> push가 거부되면 → [6. push가 거부됐을 때](#6-push가-거부됐을-때)

---

## 4. main의 최신 내용 내려받기 (매일 1회)

**하루 1번, 아침에 반드시 한다.** 이걸 미루면 마지막 날에 통합이 불가능해진다.

```bash
git switch eda              # 본인 브랜치
git pull                    # 내 브랜치 최신화
git fetch origin            # 원격 정보 갱신
git merge origin/main       # main의 변경을 내 브랜치로 가져오기
git push                    # 합친 결과 올리기
```

`Already up to date.`가 나오면 가져올 게 없다는 뜻이고 정상이다.
충돌이 나면 → [7. 충돌](#7-충돌conflict이-났을-때)

---

## 5. Pull Request 만들기

다른 팀이 쓸 수 있는 결과물이 완성되면 `main`으로 병합 요청을 만든다.
**`main`에 직접 push하지 않는다.**

1. 작업을 push한다. (`git push`)
2. GitHub 저장소 페이지 → 상단 **Pull requests** → **New pull request**
3. `base: main` ← `compare: eda` (본인 브랜치)로 맞춘다.
4. 제목은 커밋 메시지처럼 쓴다. 예: `feat(eda): 1차 전처리 파이프라인`
5. 본문은 자동으로 나오는 템플릿을 채운다.
6. **Reviewers**에 조장을 지정한다.
7. 조장 승인 후 병합한다. **본인이 직접 병합하지 않는다.**

병합된 뒤에는 각자 [4번](#4-main의-최신-내용-내려받기-매일-1회)을 실행해서 최신 `main`을 받아온다.

---

## 6. push가 거부됐을 때

거부 메시지는 크게 두 종류다. **메시지를 먼저 읽고** 어느 쪽인지 확인한다.

### 6-1. `fetch first` / `non-fast-forward` — 남이 먼저 올렸다

```
! [rejected]        eda -> eda (fetch first)
error: failed to push some refs to ...
```

**뜻:** 내가 push하기 전에 다른 팀원이 먼저 올렸다. 잘못한 게 아니다.

**해결:**

```bash
git pull
git push
```

`git pull` 중 충돌이 나면 → [7. 충돌](#7-충돌conflict이-났을-때)

> **금지:** 이때 `git push --force`를 쓰면 **다른 사람의 작업이 사라진다.** 절대 쓰지 않는다.

### 6-2. `GH013: Repository rule violations` — main에 직접 올리려 했다

```
remote: error: GH013: Repository rule violations found for refs/heads/main.
remote: - Changes must be made through a pull request.
! [remote rejected] main -> main (push declined due to repository rule violations)
```

**뜻:** `main` 브랜치는 보호되어 있어서 직접 push할 수 없다. 대부분 **역할 브랜치가 아니라 `main`에서 작업한 경우**다.

**이건 저장소가 정상 동작하고 있는 것이다.** 설정을 바꾸려 하지 말고, 커밋을 역할 브랜치로 옮기면 된다.
→ [8. main에 실수로 커밋했다](#main에-실수로-커밋했다)

---

## 7. 충돌(conflict)이 났을 때

```
CONFLICT (content): Merge conflict in notebooks/eda.ipynb
Automatic merge failed; fix conflicts and then commit the result.
```

**뜻:** 같은 파일의 같은 부분을 두 사람이 다르게 고쳤다. Git이 판단할 수 없으니 사람이 골라야 한다.
**당황하지 않아도 된다. 파일이 사라진 게 아니다.**

### 7-1. 어떤 파일이 충돌했는지 확인

```bash
git status              # "both modified" 로 표시된 파일들
```

### 7-2. 파일을 열면 이렇게 되어 있다

```
<<<<<<< HEAD
df = df.dropna()              ← 내가 쓴 코드
=======
df = df.fillna(0)             ← 상대방이 쓴 코드
>>>>>>> origin/main
```

### 7-3. 원하는 내용만 남긴다

`<<<<<<<`, `=======`, `>>>>>>>` **세 줄을 모두 지우고** 최종 코드만 남긴다.

```python
df = df.fillna(0)
```

> 둘 다 필요하면 둘 다 남겨도 된다. **판단이 어려우면 고치지 말고 상대방에게 먼저 물어본다.**

### 7-4. 해결했다고 알려주고 마무리

```bash
git add notebooks/eda.ipynb      # 해결한 파일
git status                       # 남은 충돌 파일이 없는지 확인
git commit                       # 메시지는 기본값 그대로 두고 저장·종료
git push
```

### 7-5. 노트북(.ipynb) 충돌은 해결하지 말 것

노트북은 내부가 JSON이라 충돌 표시가 수백 줄로 나온다. **직접 고치려 하면 파일이 깨진다.**

```bash
git merge --abort         # 일단 되돌리고
```

그리고 조장에게 알린다. 애초에 **노트북은 1인 1파일**로 나눠서 충돌을 만들지 않는 것이 원칙이다.

### 7-6. 도저히 안 되겠을 때 — 되돌리기

```bash
git merge --abort         # 병합 시작 전 상태로 완전히 복귀
```

이 명령은 안전하다. 아무것도 잃지 않는다.

---

## 8. 실수 복구 모음

### 아직 커밋하지 않은 변경을 되돌리고 싶다

```bash
git restore <파일명>           # 파일 하나
git restore .                  # 전체 (주의: 저장 안 한 작업이 사라진다)
```

### `git add`를 취소하고 싶다 (커밋은 아직 안 함)

```bash
git restore --staged <파일명>
```

### 방금 한 커밋 메시지를 고치고 싶다 (push 전)

```bash
git commit --amend -m "fix(eda): 올바른 메시지"
```

> push한 뒤에는 `--amend`를 쓰지 않는다. 조장에게 알린다.

### 커밋을 취소하고 싶다 (push 전, 작업 내용은 유지)

```bash
git reset --soft HEAD~1        # 커밋만 취소, 파일 변경은 그대로
```

### 잘못된 브랜치에서 작업했다 (아직 커밋 안 함)

```bash
git stash                # 작업 내용 임시 보관
git switch eda           # 올바른 브랜치로 이동
git stash pop            # 작업 내용 꺼내기
```

### main에 실수로 커밋했다

역할 브랜치가 아니라 `main`에서 작업하고 커밋한 경우다. push는 거부되지만 **작업 내용은 그대로 있다.**

**1) 확인한다**

```bash
git branch                              # * 가 main 에 있으면 해당
git log --oneline origin/main..HEAD     # main 에만 있는 내 커밋 목록
```

**2) 작업을 역할 브랜치로 옮긴다**

```bash
git switch eda          # 본인 역할 브랜치
git merge main          # main 에 있던 내 커밋을 가져온다
git push
```

**3) 옮겨진 것을 눈으로 확인한 뒤, 로컬 main을 되돌린다**

```bash
git log --oneline -3          # 내 커밋이 여기 보이는지 확인 (지금 eda 브랜치)
git switch main
git reset --hard origin/main  # 로컬 main 을 원격과 동일하게
```

> **주의:** 3단계의 `git reset --hard`는 **2단계의 `git push`가 성공한 것을 확인한 뒤에만** 실행한다.
> 확신이 서지 않으면 여기서 멈추고 조장에게 물어본다. 2단계까지만 해두어도 작업은 안전하다.

### 큰 파일을 실수로 커밋했다 (push 전)

```bash
git reset --soft HEAD~1              # 커밋 취소
git restore --staged <큰파일>        # 그 파일만 제외
git commit -m "원래 메시지"
```

**이미 push했다면 직접 해결하지 말고 조장에게 알린다.** 히스토리 수정이 필요하다.

### `.env`나 비밀키를 커밋했다

```
즉시 조장에게 알린다.
```

1. 해당 키·비밀번호를 **즉시 폐기하고 재발급**한다. (파일을 지워도 히스토리에 남아 있다)
2. 히스토리 정리는 조장이 처리한다.

### 내 작업이 사라진 것 같다

**대부분 사라지지 않았다.** 아무것도 더 하지 말고 아래 결과를 조장에게 보낸다.

```bash
git status
git log --oneline -10
git reflog -20          # 최근 모든 이동 기록 (여기서 복구 가능)
```

---

## 9. 절대 하지 말 것

| 금지 | 이유 |
|---|---|
| `git push --force` / `-f` | 다른 사람의 커밋이 영구 삭제된다 |
| `main` 브랜치에 직접 커밋·push | 검토 없이 반영되어 전체가 깨질 수 있다 |
| `git reset --hard` (뜻을 모를 때) | 저장하지 않은 작업이 복구 불가능하게 사라진다 |
| `git rebase`, `git cherry-pick` | 히스토리가 꼬인다. 필요하면 조장이 한다 |
| 남의 역할 브랜치에 push | 담당자가 예상 못 한 충돌이 발생한다 |
| 대용량 데이터·모델 파일 커밋 | 저장소가 무거워지고 되돌리기 어렵다 |
| `.env`, API 키 커밋 | 보안 사고. 히스토리에 영구히 남는다 |
| 저장소 `Settings`·`Ruleset` 변경 | push가 막히는 건 대부분 정상 동작이다. 설정이 아니라 브랜치를 확인한다 |

---

## 10. 막혔을 때

에러 메시지를 지우거나 다른 명령어를 계속 시도하지 않는다. **추가 명령이 상황을 악화시키는 경우가 많다.**

아래 3개의 결과를 그대로 복사해서 조장에게 보낸다.

```bash
git status
git log --oneline -5
git branch
```

여기에 **띄운 에러 메시지 전문**을 함께 보낸다. 이 정보만 있으면 거의 모든 상황을 복구할 수 있다.

---

## 11. 명령어 빠른 참조

```bash
# 확인
git status                   # 현재 상태 (가장 자주 쓴다)
git branch                   # 브랜치 목록, * 가 현재 위치
git log --oneline -10        # 최근 커밋 10개
git diff                     # 바뀐 내용

# 이동
git switch <브랜치>          # 브랜치 이동

# 저장 · 공유
git pull                     # 받아오기
git add <파일>               # 올릴 파일 선택
git commit -m "메시지"       # 저장
git push                     # 올리기

# main 동기화 (매일 1회)
git fetch origin && git merge origin/main

# 되돌리기
git restore <파일>           # 변경 취소
git restore --staged <파일>  # add 취소
git merge --abort            # 병합 취소
git stash / git stash pop    # 임시 보관 / 꺼내기
git reflog                   # 복구용 기록
```
