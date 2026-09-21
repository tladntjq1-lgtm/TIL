# groupby, agg, value_counts

## value_counts() — 빈도 확인

특정 열의 값들이 각각 몇 번씩 나타나는지 빠르게 확인할 때 사용한다.

```python
orders["order_status"].value_counts()
```

주문 상태별(완료/취소/환불 등) 건수를 빠르게 확인할 수 있다. 결측치도 포함해서 보고 싶다면:

```python
orders["order_status"].value_counts(dropna=False)
```

품질 점검이 목적이라면 결측치를 포함할지 여부를 의도적으로 결정해야 한다.

## value_counts vs groupby/agg

- 값별 **단순 건수**만 필요하면 → `value_counts()`
- 여러 지표(합계, 평균 등)를 함께 봐야 하면 → `groupby()` / `agg()`

## groupby — 같은 카테고리를 한 그룹으로

카테고리가 같으면 한 그룹으로 묶이고, 결과의 한 행은 카테고리 하나를 의미한다.

```python
product_summary = products.groupby("category", as_index=False).agg(
    product_count=("product_id", "nunique"),
    average_price=("price", "mean"),
)
```

- `agg()`: 지표 이름(`product_count`, `average_price`)과 계산 함수(`"nunique"`, `"mean"`)를 함께 지정한다.
- `as_index=False`: 그룹 키(`category`)를 인덱스가 아니라 **일반 컬럼**으로 남긴다. 이후 다른 데이터와 저장·병합하기 편해진다.

> ⚠️ 주의: 같은 `category`로 `groupby`를 하더라도, **전체 주문**을 기준으로 그룹화했는지 **completed 주문만** 필터링한 뒤 그룹화했는지에 따라 결과의 의미가 완전히 달라진다. 필터링 순서를 항상 의식해야 한다.

## sum() vs mean()

합계와 평균은 서로 다른 질문에 답한다. 결과를 볼 때는 항상 **어떤 단위인지**를 함께 설명해야 오해가 없다 (예: "매출 합계"와 "평균 주문 금액"은 완전히 다른 지표).

## 데이터 검증 원칙: 바로 삭제하지 않는다

`price`에 음수가 있거나 `age`가 1000살로 찍혀 있어도, **바로 삭제하는 것은 잘못된 방법**이다. 먼저 이상값의 원인을 검증하고, 검증이 끝난 뒤 필요하다면 마지막 단계에서 삭제한다.

## N:M 관계와 중간 테이블 조인 (도서 대여 예시)

```
members (1) ── (N) loans (N) ── (1) books
```

- 한 사람은 여러 책을 빌릴 수 있다.
- (착각하기 쉬운 포인트) 하나의 책도 여러 사람에게 (시간을 두고) 빌려질 수 있다 → **N:M 관계**.
- `members`, `books`, `loans` 세 테이블을 조인해서 "누가 어떤 책을 빌렸는지"를 확인한다.
