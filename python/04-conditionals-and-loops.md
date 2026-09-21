# 조건문과 반복문

## 조건문 흐름

파이썬은 조건을 위에서 아래로 순차 평가하며 **최초로 `True`가 되는 블록만 실행**하고 나머지는 건너뛴다.

```python
score = 85
if score >= 90:
    print("A")
elif score >= 80:
    print("B")   # 여기만 실행되고 즉시 조건문을 탈출
else:
    print("C")
```

입력을 받아 조건으로 바꿔 쓰는 예:

```python
member_answer = input("회원인가요? (y/n): ").strip().lower()
is_member = member_answer == "y"
```

## for문과 range()

```python
for i in range(5):
    print("Python")

for 반복변수 in 반복할값들:
    반복해서 실행할 코드
```

`range()`는 **끝 값을 포함하지 않는다.**

| 호출 | 생성되는 값 |
|---|---|
| `range(5)` | 0, 1, 2, 3, 4 |
| `range(1, 6)` | 1, 2, 3, 4, 5 |
| `range(2, 11, 2)` | 2, 4, 6, 8, 10 |
| `range(2, 10, 3)` | 2, 5, 8 (11은 10 이상이라 제외) |

반복문에서 값을 누적하는 패턴:

```python
total = 0
for number in range(1, 6):
    total += number
```

## while문

```python
while 조건식:
    조건이 참인 동안 반복할 코드

count = 1
while count <= 5:
    print(count)
    count += 1
```

실행 흐름: `count=1` → 조건 확인(True) → 출력 → `count += 1` → 조건 재확인 → ... → `count=6`이 되면 조건이 `False`가 되어 종료.

`break` 조건이 없으면 `while True:`는 무한히 돈다는 점을 항상 의식해야 한다.

## break vs continue

- `break`: 반복문 **전체**를 즉시 끝낸다.
- `continue`: **현재 회차**의 남은 코드만 건너뛰고 다음 반복으로 넘어간다 (반복문 자체는 끝나지 않음).

```python
for number in range(1, 6):
    if number == 3:
        break        # 3에서 반복 전체 종료 → 1, 2 출력
    print(number)

for number in range(1, 6):
    if number == 3:
        continue     # 3만 건너뜀 → 1, 2, 4, 5 출력
    print(number)
```

## 문제를 코드로 옮기기 전에: 수도코드

바로 코드를 작성하지 말고, 한글로 수도코드를 먼저 짜는 습관이 중요하다.

```
문제 → 데이터 → 반복 → 조건 → 결과 → 코드
```

- 데이터는 변수에 담는다.
- 반복문에 들어가는 데이터는 보통 리스트/딕셔너리 형태다.
- 테이블(표) 형태의 데이터를 담을 그릇이 마땅치 않으면 "리스트 안에 딕셔너리"를 떠올린다.

AI에게 수도코드를 짜달라고 요청하고 직접 구현해보는 연습도 도움이 된다.
