# 서브쿼리 (Subquery)

## 서브쿼리란

하나의 SQL 쿼리 안에 포함된 또 다른 `SELECT` 쿼리. 괄호 `()`로 감싸며(중괄호가 아니다), **외부 쿼리보다 먼저 실행되어** 그 결과를 외부 쿼리에 전달한다. `JOIN`은 테이블을 합치는 것이고 서브쿼리와는 다른 개념이다.

```sql
SELECT name
FROM members
WHERE member_id IN (
    SELECT member_id FROM loans WHERE returned = false  -- 서브쿼리
);
```

## WHERE절에서 여러 행을 반환할 때: `=` 대신 `IN`

서브쿼리가 **여러 행(값)**을 반환하면, 단일 값 비교 연산자인 `=`을 쓸 수 없다. 목록 중 **하나라도 일치**하면 참이 되는 `IN`을 사용한다. 반대로 목록에 포함되지 않는 경우는 `NOT IN`을 사용한다.

```sql
-- ❌ 서브쿼리가 여러 행을 반환하면 에러
WHERE member_id = (SELECT member_id FROM loans WHERE ...)

-- ✅
WHERE member_id IN (SELECT member_id FROM loans WHERE ...)
```

## `NOT IN`보다 `NOT EXISTS`를 권장하는 이유: NULL 함정

서브쿼리 결과에 `NULL`이 하나라도 포함되면, `NOT IN`은 **전체 결과가 빈 값으로** 나올 수 있다. `NOT EXISTS`는 NULL에 영향을 받지 않기 때문에 더 안전하다.

```sql
-- NULL이 섞여 있으면 예상과 다르게 빈 결과가 나올 위험
WHERE member_id NOT IN (SELECT member_id FROM loans)

-- 더 안전한 방식
WHERE NOT EXISTS (
    SELECT 1 FROM loans WHERE loans.member_id = members.member_id
)
```

> `NOT IN`이 문법 오류를 내는 것은 아니다 — 조용히 **의도와 다른(잘못된) 결과**를 내는 것이 더 위험한 지점이다. `NOT EXISTS`는 `WHERE`절에서 사용하며 `FROM`절 전용은 아니다.

## FROM절의 서브쿼리 = 인라인 뷰 (Inline View)

`FROM`절에 위치해서 마치 하나의 임시 테이블처럼 사용되는 서브쿼리를 **인라인 뷰(Inline View)** 라고 부른다. 데이터베이스에 저장되지 않는 일회성 임시 테이블처럼 동작하며, **반드시 별칭(alias)을 붙여야 한다.**

```sql
SELECT category, avg_price
FROM (
    SELECT category, AVG(price) AS avg_price
    FROM products
    GROUP BY category
) AS category_summary   -- 별칭 필수
WHERE avg_price > 10000;
```

## SELECT절의 서브쿼리 = 스칼라 서브쿼리 (Scalar Subquery)

`SELECT`절에 위치하는 서브쿼리로, **반드시 하나의 행, 하나의 열(단일 값)만 반환해야 한다.** 여러 행을 반환하면 "첫 번째 행만 자동으로 사용"되는 게 아니라 **PostgreSQL에서 오류가 발생**한다.

```sql
SELECT
    name,
    (SELECT COUNT(*) FROM loans WHERE loans.member_id = members.member_id) AS loan_count
FROM members;
```

이렇게 외부 쿼리의 각 행마다 값이 달라지도록, 외부 테이블의 컬럼을 참조하는 서브쿼리를 **상관 서브쿼리(Correlated Subquery)** 라고 한다.
