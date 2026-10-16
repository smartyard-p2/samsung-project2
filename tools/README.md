# tools: 보조 도구

## AI 여행 해석 평가

8000번 API가 실행 중이어야 한다.

```bash
.venv/bin/python tools/eval_intent.py
.venv/bin/python tools/eval_intent.py --llm
.venv/bin/python tools/eval_intent.py --llm --raw
```

- 기본: 규칙 기반 해석 평가
- `--llm`: Ollama와 규칙을 합친 실제 서비스 경로
- `--llm --raw`: 규칙 보정 전 LLM 출력만 평가

평가 문장은 `tests/data/intent_cases.json`에 있다. 기존 결과를 본 뒤 같은 문장만 고치면 과적합되므로 새 문장은 별도 홀드아웃으로 추가한다.

## 발표자료 생성

```bash
cd tools/record_deck
npm ci
node build.js <원하는-출력경로.pptx>
```

출력 파일명과 저장 위치는 사용자가 정한다. 개인 사용자명, 다운로드 폴더, 특정 PC의 절대 경로를 코드나 문서에 넣지 않는다.
