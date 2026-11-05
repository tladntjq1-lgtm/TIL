# response_model, N+1 문제, 멱등성

- 학습일: 2026-11-05
- 이어지는 글: [03. 타입 힌트와 Pydantic v2](03-type-hints-and-pydantic-v2.md)

## 1. `response_model`: 응답을 걸러주는 "보안 체"

DB에서 꺼낸 원본 데이터에는 클라이언트에게 보여주면 안 되는 필드(비밀키, 비밀번호 해시 등)가 섞여 있을 수 있다. `response_model`을 지정하면 FastAPI가 **모델에 정의된 필드만 통과**시킨다.

```python
class BookResponse(BaseModel):
    id: int
    title: str

@app.get("/books/{book_id}", response_model=BookResponse)
def get_book(book_id: int):
    return {"id": 1, "title": "FastAPI", "db_secret_key": "abc123"}
    # 응답에는 id, title만 나가고 db_secret_key는 걸러진다
```

| 방식 | 필터링 | 민감 정보 |
|---|---|---|
| `response_model` 지정 | **있음** (정의된 필드만 통과) | 유출 방지 |
| 원본 딕셔너리를 `JSONResponse`에 직접 담아 반환 | **없음** | 그대로 유출될 위험 |

- `response_model`을 지정하면 **OpenAPI 명세(Swagger)에도 자동 등록**되어 응답 형식이 문서에 보인다.
- 앞 글의 `APIResponse[BookResponse]` 같은 제네릭 모델도 `response_model`로 지정할 수 있다.

## 2. 제네릭과 MRO 복습

- **제네릭(Generic)**: 타입을 미리 고정하지 않고 사용 시점에 대입하는 기법. `T = TypeVar("T")`와 함께 `class APIResponse(BaseModel, Generic[T])`로 쓴다.
- **MRO(왼쪽 우선 법칙)**: 다중 상속에서 같은 이름의 기능이 둘 이상이면 **괄호 안 왼쪽 클래스가 우선**한다. `(BaseModel, Generic[T])`에서는 `BaseModel`이 우선권을 가진다.

## 3. N+1 문제와 SQLAlchemy 로딩 옵션

**N+1 문제**: 상위 테이블(Owner)을 1번 조회한 뒤, 반복문에서 각 Owner의 하위 데이터(Pet)에 접근할 때마다 쿼리가 **N번 추가**로 나가는 문제.

```python
owners = session.scalars(select(Owner)).all()   # 쿼리 1번
for owner in owners:
    print(owner.pets)                            # Owner마다 쿼리 1번씩 → N번
```

- 원인은 **지연 로딩(lazy loading)** 이다. 연관 데이터를 실제로 쓰는 순간에야 쿼리를 보낸다.
- 해결은 연관 데이터를 **미리 같이 가져오는** 로딩 옵션을 쓰는 것이다.

```python
from sqlalchemy import select
from sqlalchemy.orm import joinedload

stmt = select(Owner).options(joinedload(Owner.pets))   # LEFT OUTER JOIN 1번으로 함께 조회
owners = session.scalars(stmt).unique().all()
```

| 옵션 | 동작 | N+1 |
|---|---|---|
| **`joinedload()`** | **LEFT OUTER JOIN 1회**로 연관 데이터를 함께 조회 | 해결 |
| `lazyload()` | 쓰는 순간 따로 조회 (지연 로딩) | **원인** |
| `defer()` | 특정 **컬럼**의 로딩을 미룸 | 무관 (연관 테이블이 아니라 컬럼 단위) |

> 위 코드는 개념 이해용 예시다. `.unique()`는 컬렉션을 JOIN으로 가져올 때 중복된 Owner 행을 합치기 위해 붙인다. 교안에는 `joinedload` 외의 옵션(예: `selectinload`)이 나왔는지 확인하지 못했다.

## 4. 멱등성(Idempotency)

**같은 요청을 한 번 보내든 여러 번 보내든 서버의 최종 데이터 상태가 항상 같은** 성질.

| 메서드 | 멱등성 | 이유 |
|---|---|---|
| GET | O | 조회만 하므로 상태가 안 바뀐다 |
| PUT | O | 같은 값으로 덮어쓰면 결과가 같다 |
| DELETE | O | 첫 성공 이후에는 "데이터가 없는 상태"가 계속 유지된다 |
| POST | 일반적으로 X | 보낼 때마다 새 데이터가 생길 수 있다 |

> DELETE를 두 번째 보내면 응답 코드는 달라질 수 있다(예: 404). 하지만 **서버의 최종 상태**는 같으므로 멱등하다.

## 오답노트 (2026-11-05)

| 문항 | 내가 고른 것 | 정답 방향 | 틀린 이유 |
|---|---|---|---|
| 1. `response_model` 동작 | 원본 딕셔너리를 그대로 `JSONResponse`에 반환 | 정의된 필드만 통과시키는 **보안 체** | `response_model`이 막으려는 방식을 거꾸로 고름 |
| 4. N+1 해결 로딩 옵션 | `defer()` | **`joinedload()`** | `defer()`는 컬럼 로딩을 미루는 옵션이라 N+1과 무관 |

- **1번**: `response_model` = 체(필터). "체에 걸러서 안전한 필드만 내보낸다"로 기억. 필터 없이 원본을 그대로 반환하는 것이 오히려 **대비되는 위험한 방식**이다.
- **4번**: N+1의 핵심은 **"연관 테이블을 따로따로 조회한다"** 는 점이다. 해법은 **미리 JOIN으로 같이 가져오기(`joinedload`)**. `lazyload`는 원인, `defer`는 컬럼 지연이라 목적이 다르다.
- 맞힌 문제: 2번(제네릭), 3번(MRO 왼쪽 우선), 5번(멱등성).
