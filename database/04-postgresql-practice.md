# PostgreSQL 실습 (스키마/테이블/CRUD)

## PostgreSQL + DBeaver

- **PostgreSQL**: 데이터를 실제로 저장하고 관리하는 데이터베이스 서버(DBMS). 데이터를 보관하는 창고에 비유할 수 있다.
- **DBeaver**: 데이터베이스를 다루는 GUI 툴. "데이터베이스에서 VS Code 같은 느낌"의 도구.

## 스키마와 테이블 생성

```sql
-- 1. 스키마 생성
CREATE SCHEMA practice;

-- 2. 테이블 생성
CREATE TABLE practice.members (
    member_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    age INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

- `GENERATED ALWAYS AS IDENTITY`: PK를 자동 증가시킨다.
- `UNIQUE NOT NULL`: 이메일처럼 중복 없이 반드시 값이 있어야 하는 컬럼에 사용.
- `DEFAULT CURRENT_TIMESTAMP`: 값을 넣지 않으면 생성 시각이 자동으로 채워진다.

## CRUD 실습

```sql
-- Create
INSERT INTO practice.members (name, email, age)
VALUES
    ('홍길동', 'hong@example.com', 25),
    ('김영희', 'younghee@example.com', 31),
    ('이철수', 'chulsoo@example.com', 28),
    ('박민지', 'minji@example.com', 23);

-- Read (전체 조회)
SELECT * FROM practice.members;

-- Read (조건 조회)
SELECT * FROM practice.members WHERE age >= 30;

-- Update
UPDATE practice.members
SET age = 26
WHERE email = 'hong@example.com';

-- Delete
DELETE FROM practice.members
WHERE email = 'minji@example.com';

-- 최종 확인
SELECT * FROM practice.members ORDER BY member_id;
```

## Django에서 특정 스키마 우선 검색하도록 설정

Django `settings.py`의 `DATABASES["default"]["OPTIONS"]`에 `search_path`를 지정하면, 여러 스키마 중 원하는 스키마를 먼저 검색하도록 만들 수 있다.

```python
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": "...",
        "USER": "...",
        "PASSWORD": "...",   # 실제 값은 .env로 관리하고 절대 커밋하지 않는다
        "HOST": "...",
        "PORT": "...",
        "OPTIONS": {
            "options": "-c search_path=practice",
        },
    }
}
```

## Git 추적 관련 메모

`git rm --cached <파일>` 명령은 로컬 파일은 그대로 두고 **Git의 추적만 해제**한다. 비밀번호가 담긴 설정 파일을 실수로 커밋했을 때 활용할 수 있다 (다만 이미 푸시된 이력에는 남아있으니 `.gitignore` 선(先) 등록이 우선이다).
