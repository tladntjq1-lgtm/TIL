# Next.js + React로 FastAPI 회원 목록 연동하기 (Phase 03, 1~4강)

- 학습일: 2026-11-10
- 환경: Windows 11, VS Code, PowerShell, Node v24.19.0, npm 11.17.0, Next.js 16.4.6, React 19.3.0
- 이어지는 글: [01. App Router 동적 라우팅과 TypeScript 선택 속성](01-app-router-and-typescript-basics.md), 백엔드는 [backend/01](../backend/01-fastapi-basics-and-request-parameters.md)

## 한 줄 요약

Phase 02에서 만든 **FastAPI + PostgreSQL(주방)** 에 **Next.js + React 화면(홀)** 을 붙여, 브라우저에서 **실제 DB 회원 8명**을 조회하는 데까지 완성했다.

```text
브라우저 (localhost:3000)
   │ useEffect → fetch GET /members
   ▼
FastAPI (127.0.0.1:8000)  ← CORS로 3000번 허용
   │ SQLAlchemy
   ▼
PostgreSQL → 회원 8명 → JSON 배열 → Member[] State → 카드 표시
```

## 1. 폴더 구조

```text
fullstack-web-course
├─ backend            ← Phase 02 원본 (.venv 공용)
└─ react
   ├─ .env            ← FastAPI용 DB·CORS 설정 (비밀값, 커밋 금지)
   ├─ backend         ← Phase 02 코드 복사본 (Phase 03에서는 여기만 수정)
   └─ member-next-app ← Next.js 프로젝트
      ├─ .env.local   ← NEXT_PUBLIC_API_BASE_URL (커밋 금지)
      ├─ app/page.tsx
      ├─ components/ (MemberCard, MemberList, MemberWorkspace)
      ├─ lib/members-api.ts
      └─ types/member.ts
```

> 기억법: **`npm` 명령은 `member-next-app`에서**, **`uvicorn` 명령은 `react`에서** 실행한다.

## 2. 서버 켜는 순서

터미널을 역할별로 나눈다: **api**(FastAPI), **web**(Next.js), **cmd**(검사·git).

```powershell
# 8000번이 비었는지 확인 (다른 Docker 컨테이너가 쓰고 있으면 충돌)
Get-NetTCPConnection -LocalPort 8000 -State Listen -ErrorAction SilentlyContinue

# api 터미널 (react 폴더에서)
..\backend\.venv\Scripts\Activate.ps1
python -m uvicorn backend.main:app --reload --reload-dir backend --port 8000

# web 터미널 (member-next-app 폴더에서)
npm run dev
```

## 3. 강의별 정리

### 1강: 개발환경과 첫 화면

- `npx create-next-app@latest member-next-app --ts --eslint --app --no-tailwind --use-npm`으로 프로젝트 생성.
- `npm run dev` 후 코드를 저장하면 화면이 자동 반영된다.
- 검사 3종: 명령 뒤에 `; "결과: $LASTEXITCODE"`를 붙여 **0이면 통과**로 판단.

| 명령 | 비유 | 보는 것 |
|---|---|---|
| `npx tsc --noEmit` | 맞춤법 검사 | 타입이 맞는지 |
| `npx eslint ...` | 문장 다듬기 조언 | 나쁜 코드 습관 |
| `npm run build` | 인쇄 테스트 | 실제 배포 가능한지 |

### 2강: Component와 TypeScript

```text
page.tsx (members)
   └─ MemberList (map으로 한 명씩)
        └─ MemberCard × N
```

| 용어 | 비유 |
|---|---|
| Component | 레고 블록 |
| Props | 블록에 넣는 내용물 (부모 → 자식) |
| type | 내용물 규격표 (`Member`) |

- `Member` = 한 명, `Member[]` = 여러 명.
- `map()` = 목록을 하나씩 꺼내 카드로 변환 (Python의 `for`와 비슷). `key={member.member_id}`는 각 카드의 고유 이름표.
- `조건 ? A : B` = Python의 `if / else` 한 줄 버전.
- TypeScript에는 `integer`가 없고 **`number` 하나**로 정수·소수를 모두 표현한다.

```ts
// types/member.ts
export type Member = {
  member_id: number;
  name: string;
  email: string;
  age: number;
};
```

### 3강: Props, State, Event

| | 의미 |
|---|---|
| Props | 부모가 준 택배 (받기만 한다) |
| State | 내가 쓰는 메모장 (바꿀 수 있다) |
| Controlled Form | 입력칸의 `value`와 State를 `onChange`로 연결 |

- 변수를 그냥 바꾸면 화면이 모른다. 반드시 `setXxx()`로 바꿔야 다시 그려진다.
- **Server Component(기본) vs Client Component(`"use client"`)**: `useState`, `onClick`은 Client Component에서만 쓸 수 있다. 사용자가 만지는 부분만 `"use client"`로 떼어낸다.
- `filter()` + `toLowerCase()`로 대소문자 무시 검색, `event.preventDefault()`로 폼 제출 시 새로고침 막기.
- `Number.isInteger()`로 나이 검사. 화면 검사는 **사용자 편의**이고, 진짜 문지기는 **백엔드(Pydantic)** 다.

### 4강: FastAPI에서 회원 목록 가져오기

| 용어 | 비유 |
|---|---|
| fetch | 주방에 주문서 보내기 |
| useEffect | 화면이 열리면 자동으로 주문 |
| Loading / Empty / Error | "조리 중" / "재료 없음" / "주방 문제" 안내판 |
| AbortController | 이전 주문 취소 장치 |
| CORS | 3000번 문에서 오는 요청을 8000번이 허락하는 설정 |

**API 호출 함수**: 응답이 `Member[]` 모양인지 직접 검사한 뒤 반환한다.

```ts
// lib/members-api.ts
export async function getMembers(signal?: AbortSignal): Promise<Member[]> {
  const baseUrl = process.env.NEXT_PUBLIC_API_BASE_URL;
  if (!baseUrl) throw new Error("NEXT_PUBLIC_API_BASE_URL을 설정해 주세요.");

  const response = await fetch(`${baseUrl.replace(/\/$/, "")}/members`, {
    cache: "no-store",
    signal,
  });
  if (!response.ok) throw new Error(`회원 조회 실패: HTTP ${response.status}`);

  const data: unknown = await response.json();
  // Array인지, 각 원소의 필드 타입이 Member와 맞는지 검사 후 return data
  return data as Member[];
}
```

**`useEffect`로 조회하고, 화면이 닫히면 요청 취소**

```tsx
useEffect(() => {
  const controller = new AbortController();

  getMembers(controller.signal)
    .then((data) => {
      if (controller.signal.aborted) return;
      setMembers(data);
      setError("");
    })
    .catch((caught: unknown) => {
      if (controller.signal.aborted) return;
      setError(caught instanceof Error ? caught.message : "알 수 없는 조회 오류");
    })
    .finally(() => {
      if (!controller.signal.aborted) setLoading(false);
    });

  return () => controller.abort();   // 정리(cleanup)
}, [reloadKey]);                      // reloadKey가 바뀌면 다시 조회
```

- "다시 조회" 버튼은 `setReloadKey(v => v + 1)`로 `useEffect`를 다시 실행시킨다.
- 로딩/오류/목록 표시는 `loading ? ... : error ? ... : <MemberList />` 순서로 분기한다.

**설정 파일**

| 파일 | 누가 읽나 | 내용 | 비밀값 |
|---|---|---|---|
| `react\.env` | FastAPI | DB 접속 정보, `CORS_ORIGINS` | 있음 (커밋 금지) |
| `member-next-app\.env.local` | Next.js | `NEXT_PUBLIC_API_BASE_URL=http://127.0.0.1:8000` | 없음 |

- `.env` / `.env.local`은 **서버가 켜질 때만** 읽힌다. 고치면 재시작해야 한다.
- `NEXT_PUBLIC_` 값은 브라우저에 공개되므로 비밀값을 넣지 않는다.
- CORS에는 Next.js 주소(`http://localhost:3000`, `http://127.0.0.1:3000`)를 추가해야 한다. Phase 02는 Live Server(5500)만 허용하고 있었다.

**Network 탭으로 확인한 것**

| 줄 | Status | 의미 |
|---|---|---|
| members | (canceled) | Strict Mode 1회차 요청을 AbortController가 취소 (정상) |
| members | 200 | 실제 조회 성공 |
| members | 200 | "다시 조회" 버튼으로 재요청 |

- 검색어를 입력해도 요청은 나가지 않는다. 받아 둔 데이터를 브라우저에서 필터링하기 때문이다.

## 4. 만난 문제와 해결

| 증상 | 원인 | 해결 |
|---|---|---|
| 붙여 넣은 줄마다 `'✓' 용어가 ... 인식되지 않습니다` | 결과 예시를 터미널에 붙여 넣음 | 명령만 실행하고 결과는 눈으로 비교 |
| `import type ...` 터미널 에러 | 코드를 에디터가 아닌 터미널에 붙임 | 붙이기 전에 커서 위치 확인 |
| `npm error ENOENT ... package.json` | `react` 폴더에서 `npm run dev` 실행 | `member-next-app`으로 이동 후 실행 |
| `/members` 요청에 **Django 404** 화면 | 다른 Docker 컨테이너가 8000번 사용 중 | 해당 컨테이너 `docker stop` |
| `DLL load failed ... 애플리케이션 제어 정책에서 차단` | Windows 보안이 SQLAlchemy 가속(C 확장) 파일을 차단 | `DISABLE_SQLALCHEMY_CEXT=1`로 순수 Python 버전 재설치 |
| `조회 오류: Failed to fetch` + `ERR_CONNECTION_REFUSED` | FastAPI가 꺼져 있음 | api 터미널에서 FastAPI 재실행 |
| 빨간 물결인데 `tsc` 결과는 0 | VS Code 표시 지연 | `TypeScript: Restart TS Server` |
| ESLint `react-hooks/set-state-in-effect` | `useEffect` 안에서 바로 `setLoading(true)` | 버튼 클릭 함수에서 상태 변경, 응답 콜백에서 결과 반영 |
| 다크 모드에서 제목이 안 보임 | 제목 색이 검은 배경과 비슷 | 제목만 `color: "inherit"` |

```powershell
# SQLAlchemy 순수 Python 재설치 (C 확장 DLL이 차단될 때)
$env:DISABLE_SQLALCHEMY_CEXT = "1"
python -m pip install --force-reinstall --no-deps --no-binary sqlalchemy "sqlalchemy==<설치된 버전>"
```

> "스마트 앱 컨트롤 끄기"는 한 번 끄면 Windows를 다시 설치하기 전에는 켤 수 없어서 하지 않았다.

**Phase 02 → Phase 03 백엔드 복사 시 바꾼 것**
- `from app.xxx import` → `from backend.xxx import` (6개 파일)
- `config.py`의 `ENV_PATH`를 `react\.env`를 읽도록 수정
- 실행은 `react` 폴더에서 `python -m uvicorn backend.main:app ...`

## 5. 오늘의 핵심 정리

- **Props는 받기만, State는 바꾸면 화면이 다시 그려진다.**
- 데이터 가져오기는 **fetch + useEffect + Loading/Empty/Error 세 가지 상태 + AbortController** 한 세트.
- 화면의 입력 검사는 편의일 뿐이고, 최종 검증은 **백엔드(Pydantic)** 가 한다. (→ [backend/03](../backend/03-type-hints-and-pydantic-v2.md))
- 환경 설정 파일은 **누가 읽는지**(FastAPI vs Next.js)와 **비밀값이 들어가는지**로 구분하고, 비밀값 파일은 커밋하지 않는다.

## 남은 일 / 다음 강의

- [ ] `git check-ignore -v .env.local ..\.env`로 두 파일이 모두 무시되는지 확인한 뒤 커밋 (`git add .` 대신 파일명을 지정)
- 5강: 회원 등록 폼에서 `POST /members`로 실제 DB에 저장하고, 저장 후 목록을 다시 조회
- 5강에서는 화면 검사(3강)와 백엔드 Pydantic 검사를 함께 사용
