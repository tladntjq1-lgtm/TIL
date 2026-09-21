# SQL 기초 - DDL/DML/DCL과 실행 순서

## AI 시대에도 데이터베이스를 배워야 하는 이유

- AI가 SQL을 만들어줘도, **실행에 성공하는 SQL**과 **요구사항에 맞는 결과를 주는 SQL**은 다르다.
- 잘못된 데이터나 조회 기준은 결과에 직접적인 영향을 준다.
- 현실의 업무를 데이터와 관계로 표현하려면, 파일/스프레드시트/DBMS 중 어떤 방식이 적절한지 판단하는 기준이 필요하다.
- AI가 만든 데이터 구조와 SQL 결과라도 사람이 직접 확인해야 하는 항목이 있다.

## SQL 작성 순서 vs 실행 순서

**이 둘이 다르다는 것이 핵심 개념이다.**

- 작성 순서: `SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY`
- 실행 순서: `FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY`

`WHERE`는 `SELECT`보다 **먼저** 실행되기 때문에, `SELECT`에서 정의한 별칭(alias)을 `WHERE`절에서 사용할 수 없다.

```sql
-- 아래처럼 쓰면 에러가 난다 (total은 SELECT에서 만든 별칭)
SELECT price * quantity AS total
FROM orders
WHERE total > 10000;   -- ❌ WHERE가 SELECT보다 먼저 실행되므로 total을 모른다
```

## SQL 명령어 분류

| 분류 | 역할 | 대표 명령어 |
|---|---|---|
| **DDL** (Data Definition Language) | 데이터베이스 구조(테이블, 스키마)를 정의·변경·삭제 | `CREATE`, `ALTER`, `DROP` |
| **DML** (Data Manipulation Language) | 데이터 자체를 조작 | `SELECT`, `INSERT`, `UPDATE`, `DELETE` |
| **DCL** (Data Control Language) | 권한을 제어 | `GRANT`, `REVOKE` |
| **TCL** (Transaction Control Language) | 트랜잭션을 제어 | `COMMIT`, `ROLLBACK` |

`INSERT INTO`는 데이터를 조작하는 **DML**이다 (구조를 정의하는 DDL이 아니라는 점을 헷갈리기 쉬움).

## PRIMARY KEY

테이블 내에서 각 행(Row)을 유일하게 식별하는 열로, 두 가지 핵심 규칙을 가진다.

- 중복 값을 가질 수 없다.
- `NULL` 값을 가질 수 없다.

### PRIMARY KEY vs UNIQUE

둘 다 "중복 불가"라는 점은 같지만, `NULL` 허용 여부가 다르다.

| 제약 | 중복 | NULL |
|---|---|---|
| `PRIMARY KEY` | 불가 | **불가** |
| `UNIQUE` | 불가 | **허용** (단, NULL은 여러 번 허용) |

한 테이블에 `PRIMARY KEY`는 하나만 지정할 수 있지만, `UNIQUE`는 여러 열에 걸어둘 수 있다 (예: `email` 컬럼).

## 타입이 맞지 않는 값을 넣으면?

`INT` 타입 열에 문자열(`'홍길동'`)을 넣으려 하면 `invalid input syntax for type integer` 같은 오류가 발생하며 해당 `INSERT`가 거부된다. 데이터베이스가 잘못된 타입의 데이터가 저장되는 것을 자동으로 방지해준다.

## INSERT INTO에서 열 이름을 명시하는 이유

```sql
INSERT INTO members (name, email, age)
VALUES ('홍길동', 'hong@example.com', 25);
```

열 이름을 명시하면 열 순서가 바뀌거나 일부 열만 입력해도 오류 없이 안전하게 동작한다. `VALUES` 절은 쉼표로 구분해 여러 행을 한 번에 입력할 수도 있다.
