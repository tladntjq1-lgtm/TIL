# 함수

## 함수 정의와 호출

반복되는 코드를 함수로 묶으면 재사용할 수 있다.

```python
print("안녕하세요!")
print("파이썬 공부를 시작합니다.")

print("안녕하세요!")
print("파이썬 공부를 시작합니다.")
```

위 코드를 함수로 묶으면:

```python
def greet():
    print("안녕하세요!")
    print("파이썬 공부를 시작합니다.")

greet()
greet()
```

## 매개변수(parameter) vs 인수(argument)

```python
def add(a, b):        # a, b → 매개변수(parameter)
    return a + b

total = add(10, 20)   # 10, 20 → 인수(argument)
print(total)          # 30
```

- `10`과 `20`을 인수로 전달 → `a`, `b`가 값을 받음 → `a + b` 계산 → `return 30` → `total`에 30 저장.
- 함수 안에서는 `print`를 잘 쓰지 않고, **필요한 값을 `return`하는 것이 일반적**이다. (`print`는 화면에 보여줄 뿐 값을 반환하지 않는다. `split()`처럼 값을 반환하는 함수와는 역할이 다르다.)

## 가변 인수 `*args`, `**kwargs`

매개변수 앞에 별표 하나(`*`)를 붙이면, 전달된 여러 인수를 순서대로 묶어 **튜플**로 처리한다. 별표 두 개(`**`)는 **딕셔너리** 구조로 처리된다.

```python
def show_args(*args, **kwargs):
    print(args)    # 튜플
    print(kwargs)  # 딕셔너리
```

> `*하나는 튜플`, `**딕셔너리` — 헷갈릴 때마다 이 문장으로 기억하기.

같은 원리로 언패킹에도 별표를 쓸 수 있다.

```python
x, *rest = (1, 2, 3, 4)
print(x)     # 1
print(rest)  # [2, 3, 4]  ← 별표가 붙은 변수는 원본 자료형과 관계없이 항상 리스트로 담긴다
```

## 여러 값 반환 (튜플 언패킹)

함수는 여러 값을 반환하고, 호출하는 쪽에서 튜플 언패킹으로 나눠 받을 수 있다.

```python
def divide(a, b):
    quotient = a // b
    remainder = a % b
    return quotient, remainder

q, r = divide(17, 4)
```

## 기본값 매개변수 / 키워드 인수

```python
def greet(name, greeting="안녕하세요"):
    print(f"{greeting}, {name}님!")

greet("민수")                     # 기본값 사용
greet("민수", greeting="반갑습니다")  # 키워드 인수로 덮어쓰기
```

## generator와 yield

`return`은 함수를 완전히 종료시키지만, `yield`는 **값을 반환하면서도 함수의 실행 상태(지역 변수, 실행 위치)를 메모리에 보존한 채 일시 정지**한다. 다음 `next()` 호출 시 그 지점부터 다시 실행된다.

리스트 컴프리헨션(`[]`)과 문법이 비슷하지만 소괄호(`()`)를 쓰는 **제너레이터 표현식**은 결과를 한 번에 메모리에 올리지 않고 필요한 시점에 하나씩 생성해 메모리를 절약한다.

```python
squares = (x * x for x in range(1000000))  # 제너레이터: 메모리 절약
```
