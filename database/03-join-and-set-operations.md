# JOIN과 집합 연산

## UNION vs JOIN — 방향이 다르다

- **UNION**: 같은 구조의 `SELECT` 결과를 **세로(행) 방향**으로 쌓는다. 열 개수와 데이터 타입이 동일해야 한다. `ON` 절이 필요 없다.
- **JOIN**: 공통 키를 기준으로 서로 다른 테이블을 **가로(열) 방향**으로 연결한다. `ON` 절이 필요하다.

열 수·타입 일치 조건은 `UNION`에만 해당되고, `JOIN`에는 해당되지 않는다.

## INNER JOIN — 교집합만

```sql
SELECT *
FROM customers
INNER JOIN orders ON customers.id = orders.customer_id;
```

두 테이블 모두에 일치하는 행만 반환한다. 어느 한쪽에만 존재하는 행은 결과에서 제외된다. 가장 기본적이고 많이 쓰이는 JOIN이다.

## LEFT JOIN — 왼쪽 테이블은 전부 유지

```sql
SELECT *
FROM customers
LEFT JOIN orders ON customers.id = orders.customer_id;
```

왼쪽 테이블(`customers`)의 모든 행을 포함하며, 오른쪽 테이블에 일치하는 행이 없으면 그 열은 **NULL**로 채워진다. (`0`이 아니라 `NULL`이라는 점이 자주 틀리는 포인트!) 즉 주문이 없는 고객도 결과에 포함되고, `orders` 관련 값만 `NULL`이 된다.

## GROUP BY와 HAVING

- `WHERE`절은 `GROUP BY` 전에 실행되므로, **집계 함수의 결과를 조건으로 사용할 수 없다.** 그렇게 작성하면 에러가 난다.
- 집계 결과에 조건을 걸려면 반드시 **HAVING**을 사용해야 한다.

```sql
SELECT category, COUNT(*) AS product_count
FROM products
GROUP BY category
HAVING COUNT(*) > 10;   -- 집계 결과(개수)에 대한 조건이므로 WHERE가 아니라 HAVING
```

## LIKE — 문자열 포함 검색

```sql
SELECT * FROM products
WHERE name LIKE '%Trek%';
```

`LIKE`는 `%`를 활용해 특정 문자열을 포함하는 패턴을 조회할 때 사용한다. `IN`은 정확히 일치하는 값만 찾는 반면, `LIKE`는 앞뒤에 어떤 값이 오더라도 지정한 문자열이 포함되면 조회된다.
