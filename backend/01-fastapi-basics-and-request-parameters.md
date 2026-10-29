# FastAPI 기초: 요청 흐름, Parameter, Pydantic, 상태 코드

- 학습일: 2026-10-29
- 주제: 브라우저에서 데이터베이스까지의 흐름을 이해하고, FastAPI로 회원 조회·등록 API를 만들어 Swagger로 확인하기

## 1. 브라우저부터 데이터베이스까지

브라우저가 요청을 보내면 API 서버가 요청을 처리하고, 필요하면 데이터베이스에 질문한 뒤 응답을 돌려준다.

```text
Browser → HTTP Request → Uvicorn → FastAPI Route → Python 함수
                                      ↓ 필요하면
                                 Database Query
Browser ← JSON Response ← FastAPI 응답
```

| 용어 | 의미 |
|---|---|
| **API** | 프로그램끼리 요청과 응답을 주고받는 약속 |
| **Endpoint** | Method와 URL로 정해지는 API의 한 지점. 예: `GET /members` |
| **Uvicorn** | FastAPI 애플리케이션을 HTTP 서버로 실행 |
| **FastAPI** | 요청을 적절한 Python 함수에 연결하고 응답을 만드는 도구 |
| **Swagger UI** | `/docs`에서 API를 살펴보고 직접 실행해 볼 수 있는 화면 |
| **cURL** | 터미널에서 HTTP 요청을 보내는 도구. Swagger가 실행한 요청을 cURL 명령으로도 보여준다 |

## 2. 요청 방식과 입력값

- **GET**: 주로 데이터를 **조회**할 때 사용
- **POST**: 주로 **새 데이터를 보낼 때** 사용. 회원 등록처럼 필드가 여러 개면 보통 JSON Request Body를 쓴다

| 입력값 | 위치 | 예시 | 역할 |
|---|---|---|---|
| **Path Parameter** | URL 경로 | `/members/3` | 특정 자원을 가리킴 (`3`은 회원 ID) |
| **Query Parameter** | URL의 `?` 뒤 | `?name=kim&min_age=20` | 검색·필터 조건 |
| **Request Body** | 요청 본문 | `{"name":"Kim","email":"kim@example.com","age":25}` | 새로 만들거나 수정할 데이터 |

> 기억하기: **Path는 "누구?", Query는 "어떤 조건?", Body는 "어떤 데이터를 보낼까?"**

## 3. FastAPI 서버 실행과 Swagger 확인

`main.py`가 있는 폴더에서 가상환경을 활성화한 뒤 실행한다.

```powershell
python -m uvicorn main:app --reload --port 8010
```

- `main:app`: `main.py` 파일 안의 `app` 객체
- `--reload`: 코드를 저장하면 개발 서버가 자동으로 재시작
- 서버 실행 중 `http://127.0.0.1:8010/docs`를 열면 Swagger UI가 보인다.
- `--reload` 서버가 켜져 있으면 다시 실행하지 않고, **파일 저장 후 Swagger를 새로고침**하면 된다. 서버가 종료됐다면 실행 명령을 다시 입력한다.

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello FastAPI"}
```

## 4. 회원 조회 API

메모리의 회원 목록을 쓰면 DB 없이 조회 흐름을 연습할 수 있다.

```python
members = [
    {"id": 1, "name": "Kim", "email": "kim@example.com", "age": 25},
    {"id": 2, "name": "Lee", "email": "lee@example.com", "age": 19},
    {"id": 3, "name": "Park", "email": "park@example.com", "age": 31},
]
```

| Endpoint | 하는 일 |
|---|---|
| `GET /members` | 전체 회원 목록 |
| `GET /members/{member_id}` | ID로 회원 한 명 조회 (Path Parameter) |
| `GET /members/search?name=kim` | 이름 검색 (Query Parameter) |
| `GET /members/filter/?name=k&min_age=20` | 이름과 최소 나이 조건으로 필터링 |

`min_age=20`은 "나이가 **20 이상**"이라는 뜻이다. 19살 회원은 제외되고 20살 회원은 포함된다.

### 경로 등록 순서와 슬래시

- `/members/search` 같은 **고정 경로를 `/members/{member_id}`보다 먼저** 등록해야 한다. 순서가 반대이면 `search`가 회원 ID로 해석된다.
- 코드의 경로와 실제 요청 URL의 **마지막 슬래시(`/`)** 도 맞춰야 한다.

## 5. 상태 코드 구분

| 코드 | 의미 | 예시 |
|---|---|---|
| **200** | 조회 요청 성공 | `GET /members` |
| **201** | 새 데이터 생성 성공 | `POST /members` |
| **404** | 입력 형식은 맞지만 해당 자원을 찾지 못함 | `/members/99` |
| **422** | 입력값의 타입이나 구조가 약속과 다름 | 정수 ID 자리에 `abc` |
| **500** | 서버 코드에서 예기치 않은 오류 발생 | 터미널 traceback의 마지막 줄을 확인 |

### 404 vs 422 구분법

- `{"detail":"Member not found"}` → 코드가 회원을 찾지 못해 직접 반환한 **404**.
- `loc: ["path", "member_id"]`가 포함된 정수 변환 오류 → Path Parameter **검증 실패(422)**.

한 줄로: **404는 "형식은 맞는데 없다", 422는 "형식부터 틀렸다."**

## 6. Pydantic과 POST Request Body

Pydantic 모델은 **요청 데이터의 모양과 타입을 정하는 설계도**다. FastAPI는 함수가 실행되기 전에 JSON을 읽어 이 모델에 맞는지 **자동으로 검증**한다.

```python
from fastapi import FastAPI, status
from pydantic import BaseModel

class MemberCreate(BaseModel):
    name: str
    email: str
    age: int

app = FastAPI()

@app.post("/members", status_code=status.HTTP_201_CREATED)
def create_member(member: MemberCreate):
    return member
```

Swagger의 **POST `/members` → Try it out**에서 아래 JSON을 보내면 201과 회원 정보가 응답된다.

```json
{
  "name": "Hong Gil Dong",
  "email": "hong@example.com",
  "age": 30
}
```

- 이 연습 코드에는 DB 저장이 없어서 데이터를 실제로 보관하지는 않는다.
- 필수 필드를 빼거나 `age`에 숫자가 아닌 값을 보내면 일반적으로 **422**가 나온다. (TypeScript의 `interface`로 데이터 모양을 정하는 것과 비슷한 역할이다: [nextjs/01](../nextjs/01-app-router-and-typescript-basics.md))

## 7. 임의 ID를 응답에 붙이기

요청 모델에 `id`를 선언하면 `member.id`에 값을 넣을 수 있다.

```python
import random

class MemberCreate(BaseModel):
    id: int
    name: str
    email: str
    age: int

@app.post("/members", status_code=status.HTTP_201_CREATED)
def create_member(member: MemberCreate):
    member.id = random.randint(1, 100)
    return member
```

- Swagger 요청 JSON에 `id`를 **포함해야** 한다 (`id`가 필수 필드이기 때문).
- 서버는 입력된 ID를 난수로 **덮어써서** 응답한다. 이 예시도 DB에 저장하지는 않는다.
- 실제 서비스에서는 ID를 클라이언트가 보내지 않고, DB가 자동 생성(`GENERATED ... AS IDENTITY`)하는 것이 맞다. ([database/04](../database/04-postgresql-practice.md))

## 오늘 확인한 실습 결과

- FastAPI 서버를 실행하고 Swagger UI를 열었다.
- 전체 회원, 특정 회원, 검색·필터 API의 응답을 확인했다.
- 404(없는 회원)와 422(잘못된 타입)를 구분했다.
- POST `/members`로 JSON Body를 보내 201 응답을 확인했다.
- 난수 ID를 만들어 응답에 넣는 방식도 살펴봤다.

## 다음에 이어서 볼 것

- 메모리 목록 대신 PostgreSQL에 실제로 저장·조회하기 (오늘 흐름도의 `Database Query` 구간)
- 수정(PUT/PATCH)과 삭제(DELETE) API
- 회원이 없을 때 `HTTPException(404)`을 직접 발생시키는 방법
