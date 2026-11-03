# 타입 힌트와 Pydantic v2: 검증기, model_dump, 제네릭 응답

- 학습일: 2026-11-03
- 이어지는 글: [02. ASGI vs WSGI, 로그 레벨, --reload](02-asgi-wsgi-logging-and-reload.md)
- 앞선 글에서 `BaseModel`로 요청 Body를 검증했다면([01](01-fastapi-basics-and-request-parameters.md)), 이번에는 Pydantic v2의 검증 도구와 수정(PUT) API, 공통 응답 모델을 다룬다.

## 1. 타입 힌트는 "힌트"일 뿐이다

```python
def add(a: int, b: int) -> int:
    return a + b

add("1", "2")   # 에디터는 경고를 띄우지만, 실행하면 그냥 "12"가 나온다
```

- 타입 힌트는 **VS Code 같은 에디터가 경고를 띄워주는 용도**다. 실행 시에는 **강제력이 없다.**
- 이 한계를 보완하려고 등장한 것이 **Pydantic**이다. 실행 시점에 데이터를 실제로 검사하고 변환하며, 어긋나면 오류를 낸다. (FastAPI에서는 422 응답)

| | 타입 힌트 | Pydantic |
|---|---|---|
| 검사 시점 | 에디터(작성 중) | **실행 시(데이터가 들어올 때)** |
| 강제력 | 없음 | 있음 (검증 실패 시 예외) |
| 타입 변환 | 안 함 | 가능한 범위에서 변환 |

## 2. Pydantic v1 → v2 핵심 변화

| 항목 | v1 | v2 |
|---|---|---|
| 단일 필드 검증기 | `@validator` | **`@field_validator`** |
| 모델 전체 검증기 | `@root_validator` | **`@model_validator`** |
| 객체 → dict 변환 | `.dict()` | **`.model_dump()`** |
| Union 매칭 | `left_to_right` | **`smart`** (의도치 않은 형변환 방지) |

> Union 매칭은 "왼쪽부터 순서대로 시도"에서 "입력에 가장 잘 맞는 타입을 고르는" 방식으로 바뀌었다. 방향을 반대로 서술한 선지는 틀린 설명이다.

## 3. 검증기: field_validator와 model_validator

```python
from pydantic import BaseModel, field_validator, model_validator

class BookCreate(BaseModel):
    title: str
    price: int
    discount_price: int | None = None

    @field_validator("title")                 # 한 필드만 검사
    @classmethod
    def title_not_blank(cls, v: str) -> str:
        if not v.strip():
            raise ValueError("제목은 비어 있을 수 없다")
        return v

    @model_validator(mode="after")            # 모든 필드 검사가 끝난 뒤 실행
    def check_discount_price(self):
        if self.discount_price is not None and self.discount_price >= self.price:
            raise ValueError("할인가는 정가보다 낮아야 한다")
        return self
```

| 데코레이터 | 대상 | 용도 |
|---|---|---|
| `@field_validator("필드명")` | 필드 하나 | 그 필드의 값만 검사·가공 |
| `@model_validator(mode="after")` | **완성된 인스턴스(`self`)** | 여러 필드를 서로 비교하는 **복합 비즈니스 규칙** |

- `mode="after"`는 기본 타입 검사와 `@field_validator`가 **다 끝난 뒤** 작동한다. 그래서 `self.price`, `self.discount_price`처럼 이미 검증된 값을 꺼내 비교할 수 있다.
- `mode="before"`는 필드 검증 **이전**의 원본 입력을 다루는 옵션이다. "수정 API에서 생략된 필드 처리"와는 관련이 없다.

## 4. PUT/PATCH 수정 API: `exclude_unset=True`

수정 요청에서 클라이언트가 `title`만 보내면, 모델의 나머지 필드는 기본값(예: `None`)으로 채워진다. 이 값을 그대로 저장소에 덮으면 **기존 데이터가 `None`으로 오염**된다.

```python
class BookUpdate(BaseModel):
    title: str | None = None
    price: int | None = None
    discount_price: int | None = None

@app.put("/books/{book_id}")
def update_book(book_id: int, payload: BookUpdate):
    update_data = payload.model_dump(exclude_unset=True)   # 보낸 필드만 추출
    books[book_id].update(update_data)
    return books[book_id]
```

| 옵션 | 제외하는 것 | 이 상황에서 |
|---|---|---|
| **`exclude_unset=True`** | 클라이언트가 **보내지 않은** 필드 | 정답. 생략한 필드는 건드리지 않는다 |
| `exclude_none=True` | 값이 `None`인 필드 | 다른 옵션. 클라이언트가 **일부러** `null`을 보낸 필드도 같이 빠진다 |

한 줄로: **"안 보냈다"(unset)와 "None을 보냈다"는 다르다.** 수정 API는 "안 보냈다"를 기준으로 걸러야 한다.

## 5. 제네릭 공통 응답 모델: `Generic[T]`

```python
from typing import Generic, Optional, TypeVar
from pydantic import BaseModel

T = TypeVar("T")

class APIResponse(BaseModel, Generic[T]):
    success: bool
    data: Optional[T] = None

APIResponse[BookResponse]          # 단건 응답
APIResponse[list[BookResponse]]    # 목록 응답
```

- `T`는 **나중에 채워 넣는 타입 자리표시자**다. 사용하는 시점에 `APIResponse[BookResponse]`처럼 타입을 대입한다.
- 응답 형식(`success`, `data`)은 하나로 통일하고, `data` 안에 들어갈 내용만 상황에 따라 바꿀 수 있다.
- `BaseModel`의 기능은 그대로 유지되고, 리스트 형태 응답에도 쓸 수 있다.
- 여러 부모를 상속하면 **MRO(왼쪽 우선)** 에 따라 왼쪽에 적힌 `BaseModel`이 우선권을 가진다. `Generic[T]`가 `BaseModel`을 덮어쓰는 것은 아니다.

## 오답노트 (2026-11-03)

| 문항 | 내가 고른 것 | 정답 방향 | 틀린 이유 |
|---|---|---|---|
| 1. 타입 힌트 설명 | 실행 중 강제 검사 | 에디터 경고 수준, 강제력 없음 | **Pydantic이 하는 일**을 타입 힌트의 기능으로 착각 |
| 2. v2 변화 **옳지 않은 것** | `@validator` → `@field_validator` | Union 매칭 방향을 반대로 쓴 4번 | **부정 문제를 거꾸로 읽음** (선택한 B는 옳은 설명) |
| 4. 생략 필드 덮어쓰기 방지 | `exclude_none=True` | `exclude_unset=True` | "None 제외"와 "안 보낸 것 제외"를 혼동 |

- **1번**: 키워드 정리. 타입 힌트 = 에디터 경고(실행 시 무력) / Pydantic = 실행 시 검증·변환.
- **2번**: 지난 퀘스트(11-02, 3·4번)에 이어 **또 부정 문제를 놓쳤다.** 세 번째 반복이므로 습관으로 고정한다. 문제를 읽자마자 "옳지 **않은** 것"에 동그라미를 치고, 선지마다 "이건 맞는 설명인가?"를 판단한 뒤 **틀린 것 하나만** 고른다.
- **4번**: `unset`(안 보냄)과 `none`(None 값)을 구분하자. 수정 API에서 "일부만 바꾸기"는 `exclude_unset=True`.
- 맞힌 문제: 3번(`@model_validator(mode="after")`), 5번(`Generic[T]`로 타입을 유연하게 대입).
