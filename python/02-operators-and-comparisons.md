# 연산자와 비교

## 산술 연산자

| 연산자 | 의미 |
|---|---|
| `+` | 더하기 |
| `-` | 빼기 |
| `*` | 곱하기 |
| `/` | 나누기 (실수 나눗셈) |
| `//` | 몫 (정수 나눗셈) |
| `%` | 나머지 |
| `**` | 거듭제곱 |

`/`, `//`, `%`의 차이를 "125분을 시간과 분으로 바꾸기" 예제로 이해하면 명확하다.

```python
minutes = 125

print(minutes / 60)   # 2.0833333333333335 → 정확한 값(소수 포함)
print(minutes // 60)  # 2  → 60이 125 안에 완전히 몇 번 들어가는지 (시간)
print(minutes % 60)   # 5  → 60씩 나누고 남은 나머지 (분)

hours = minutes // 60
remaining_minutes = minutes % 60
print(f"{minutes}분 = {hours}시간 {remaining_minutes}분")  # 125분 = 2시간 5분
```

`//`와 `%`를 함께 쓰면 "몫 + 나머지"로 딱 나눠지는 값을 만들 수 있다.

## 비교 연산자와 bool

`==`, `!=`, `>`, `<`, `>=`, `<=`의 비교 결과는 항상 `bool`(`True`/`False`) 타입이다.

## 논리 연산자

| 연산자 | 규칙 |
|---|---|
| `and` | 모두 참일 때만 `True` (하나라도 `False`면 진다) |
| `or` | 하나라도 참이면 `True` |
| `not` | 참/거짓을 반대로 바꿈 |

```python
age = 20
has_ticket = True

print(age >= 18 and has_ticket)  # True
print(age < 18 or has_ticket)    # True
print(not has_ticket)            # False
```

## 복합 대입 연산자

```python
score = 80
score = score + 5   # 같은 표현
score += 5           # 복합 대입 연산자
print(score)         # 85
```

## 타입이 다른 값끼리 연산하면 에러가 난다

```python
age = 20
print(age + "살")  # TypeError
```

`int`와 `str`은 서로 다른 타입이라 바로 `+` 연산이 되지 않는다. `str()`로 감싸서 `age`를 문자열로 바꿔주면 해결된다.

```python
print(str(age) + "살")  # "20살"
```

AI에게 질문할 때도 "수정 코드"만 받지 말고, **왜 안 되는지 원리부터 물어본 뒤** 최소한의 수정으로 해결하는 습관을 들이는 것이 좋다.
