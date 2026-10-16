# web: React 프론트엔드

React 19, TypeScript, Vite로 만든 모바일 우선 웹 화면이다.

## 개발 서버

```bash
cd web
npm ci
npm run dev
```

기본 주소는 `http://localhost:5173`이며 API 요청은 `http://localhost:8000`으로 전달한다. API 주소를 바꾸려면 Git에 올리지 않는 `web/.env.local`을 만든다.

```dotenv
VITE_API_TARGET=http://localhost:8000
```

## 빌드와 검사

```bash
npm run build
npm run lint
npm run preview
```

`npm run build`는 `web/dist`를 만든다. 운영 컨테이너 반영은 저장소 루트에서 `docker compose up -d --build --no-deps api`를 실행한다.

## 구조

| 파일 | 화면·역할 |
|---|---|
| `src/FeedApp.tsx` | 앱 뼈대와 하단 탭·브라우저 이력 |
| `src/FeedExplore.tsx` | 해외 사진 탐색 |
| `src/FeedSearch.tsx` | 사진 업로드·자르기·추천 결과 |
| `src/FeedAsk.tsx` | AI 여행 문장 검색 |
| `src/FeedRegion.tsx` | 지역 상세 |
| `src/FeedMy.tsx`, `AccountSettings.tsx` | 저장·반응·계정 설정 |
| `src/api.ts` | API 함수와 TypeScript 응답 타입 |
| `src/components/` | 지도, 코스, 음식점, 사진, 비 예보 |

## 작업 규칙

- API 응답 타입은 `app/schemas.py`와 동시에 수정한다.
- 정적 이미지와 아이콘은 `web/public/`에 둔다.
- 비밀키는 프론트 환경 변수에 넣지 않는다. 브라우저에 들어간 값은 공개 정보다.
- 로그인·쿠키 확인은 Vite 주소와 실제 8000번 주소에서 모두 수행한다.
- `node_modules`와 `web/dist`는 커밋하지 않는다.
