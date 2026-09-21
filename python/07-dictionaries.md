# 딕셔너리

## key: value 구조

딕셔너리는 중괄호와 `키: 값` 형식으로 만든다. 리스트가 순서(인덱스)로 값을 찾는 것과 달리, 딕셔너리는 **키(key)** 로 값을 찾는다.

```python
student = {"name": "민수", "score": 85}
print(student["name"])   # 민수
```

- `key`는 데이터베이스의 **PK(주키)** 와 비슷한 역할로, 값을 식별하는 이름표다.
- key-value 구조를 쓰는 이유: 순서가 아니라 **의미 있는 이름**으로 값을 조회할 수 있어서다.

## 조회, 추가, 수정, 삭제

```python
student["age"] = 20        # 새 키 추가
student["score"] = 90      # 기존 키 값 수정

del student["age"]         # 삭제
student.pop("score")       # 삭제 (삭제된 값을 반환)
```

## 안전하게 조회하기: get(), in

없는 키를 그냥 `student["없는키"]`로 조회하면 `KeyError`가 발생한다. 안전하게 다루려면:

```python
student.get("height", "정보 없음")   # 키가 없으면 기본값 반환
"name" in student                    # 키 존재 여부 확인 (bool)
```

## keys(), values(), items()

```python
for key in student.keys():
    print(key)

for value in student.values():
    print(value)

for key, value in student.items():
    print(key, value)
```

## 리스트 + 딕셔너리 조합 (표 형태 데이터)

여러 건의 데이터를 표현할 때는 **리스트 안에 딕셔너리**를 담는 구조를 가장 많이 쓴다. 엑셀 시트처럼 한 행 = 딕셔너리 하나로 보면 된다.

```python
students = [
    {"name": "민수", "score": 85, "attendance": 92},
    {"name": "지영", "score": 72, "attendance": 95},
    {"name": "현우", "score": 91, "attendance": 78},
    {"name": "수빈", "score": 88, "attendance": 90},
]

selected_students = []
for student in students:
    name = student["name"]
    score = student["score"]
    attendance = student["attendance"]
    if score >= 80 and attendance >= 80:
        selected_students.append(student)

for student in selected_students:
    print(
        f"{student['name']}: "
        f"점수 {student['score']}점, "
        f"출석률 {student['attendance']}%"
    )
```

## 중첩 딕셔너리

바깥 키부터 한 단계씩 읽어 내려가면 헷갈리지 않는다.

```python
member = {
    "id": 1,
    "profile": {"email": "test@example.com", "unique": True},
}
print(member["profile"]["email"])
```

`email` 처럼 **UNIQUE 제약**이 필요한 값과, 학번처럼 **UNIQUE 또는 NOT NULL** 제약이 필요한 값을 구분해서 설계하는 감각은 이후 SQL 테이블 설계와도 이어진다.
