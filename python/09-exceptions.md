# 예외 처리

## 오류의 세 종류

- **문법 오류(SyntaxError)**: 코드를 실행하기도 전에 걸러짐
- **실행 중 예외(Exception)**: 실행되다가 특정 조건에서 발생
- **논리 오류**: 오류는 안 나지만 결과가 잘못됨 (가장 찾기 어려움)

트레이스백(Traceback)의 **마지막 줄**에서 예외 이름과 설명을 먼저 확인하고, 파일명과 줄 번호를 따라 실제 오류 위치를 찾아간다.

## try - except - else - finally

```python
try:
    number = int(input("정수: "))
except ValueError:
    print("정수가 아닙니다.")
else:
    print(f"입력 성공: {number}")   # try에서 예외가 없었을 때만 실행
finally:
    print("입력 처리를 마쳤습니다.")  # 성공/실패 관계없이 항상 실행
```

| 키워드 | 실행 조건 |
|---|---|
| `try` | 예외가 발생할 수 있는 코드를 시도 |
| `except` | 지정한 예외가 발생했을 때 대응 |
| `else` | `try`에서 예외가 없었을 때 실행 |
| `finally` | 성공/실패와 관계없이 마지막에 항상 실행 |

## 대표 예외 5가지

| 예외 | 대표 상황 | 먼저 확인할 것 |
|---|---|---|
| `ValueError` | `int("abc")` | 값의 형식 |
| `ZeroDivisionError` | `10 / 0` | 나누는 값 |
| `FileNotFoundError` | 없는 파일 열기 | 파일 이름·경로 |
| `KeyError` | 없는 딕셔너리 키 | 키 이름·존재 여부 |
| `IndexError` | 범위를 벗어난 인덱스 | 리스트 길이·인덱스 |

## while + try로 올바른 입력을 받을 때까지 반복

```python
while True:  # break 조건이 없으면 무한히 돈다
    try:
        score = int(input("점수(0~100): "))
    except ValueError:
        print("숫자로 입력해 주세요.")
        continue

    if 0 <= score <= 100:
        break
    print("0부터 100 사이의 값을 입력해 주세요.")

print(f"입력된 점수: {score}")
```

## 피해야 할 형태: 이름 없는 except

```python
# 나쁜 예
try:
    result = 100 / number
except:
    print("오류")
```

문제점:
- 어떤 오류가 발생했는지 알 수 없다.
- 예상하지 못한 개발 버그까지 조용히 숨겨버린다.
- 디버깅할 정보가 줄어든다.

```python
# 좋은 예
try:
    result = 100 / number
except ZeroDivisionError:
    print("0으로 나눌 수 없습니다.")
```

## 예외 종류별로 나눠서 처리하는 계산기 예제

```python
def calculate(a, b, operator):
    if operator == "+":
        return a + b
    elif operator == "-":
        return a - b
    elif operator == "*":
        return a * b
    elif operator == "/":
        return a / b
```

이 함수를 테스트할 때는 **정상 입력만으로는 예외 처리 품질을 확인할 수 없다.** 아래처럼 경계값과 비정상 입력을 같이 테스트해야 한다.

```
10, 2, /    → 정상 계산
10, 0, /    → ZeroDivisionError
abc, 2, +   → ValueError
10, 2, %    → 지원하지 않는 연산자
```

> 예외 처리와 "값의 범위를 검증하는 것"은 서로 다른 문제다. 예외 처리는 프로그램이 죽지 않게 막는 것이고, 범위 검증은 값 자체가 요구사항에 맞는지 확인하는 것이다.
