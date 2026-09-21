# 문자열

## 연결과 반복

문자열에서는 숫자 계산과 같은 기호(`+`, `*`)를 쓰지만 의미가 달라진다.

```python
first = "Hello"
second = "Python"

print(first + second)        # HelloPython (연결)
print(first + " " + second)  # Hello Python
print("Python! " * 3)        # Python! Python! Python!  (반복)
print("-" * 20)              # 구분선 만들기에 자주 사용
```

- 문자열 `+` 문자열 → 연결
- 문자열 `*` 정수 → 반복

## 인덱싱과 슬라이싱

- 인덱싱으로 특정 위치의 문자 하나를 가져올 수 있다. 음수 인덱스는 뒤에서부터 센다.
- 슬라이싱으로 문자열 일부를 잘라낼 수 있다.

```python
text = "Hello Python"
print(text[0])     # H
print(text[-1])    # n (맨 끝 글자)
print(text[0:5])   # Hello
print(text[::-1])  # 전체를 역순으로 뒤집기
```

## 자주 쓰는 문자열 메서드

| 메서드 | 역할 |
|---|---|
| `upper()` / `lower()` | 대문자 / 소문자 변환 |
| `strip()` | 양 끝 공백(또는 지정 문자) 제거 |
| `replace(old, new)` | 문자열 치환 |
| `find()` | 문자열 위치 검색 |
| `count()` | 등장 횟수 세기 |
| `len()` | 길이 확인 |

### `strip()` 주의사항 — 가운데 문자는 지우지 않는다

```python
text = "   hello world   "
print(text.strip())  # "hello world" ← 앞뒤 공백만 제거, 가운데는 그대로

line = "hello\n"
print(line.strip())  # "hello" ← 줄바꿈 문자도 제거됨 (readline() 결과에서 자주 사용)

text = "***hello***"
print(text.strip("*"))  # "hello" ← 지정한 문자만 양 끝에서 제거

text = "  hi there  "
print(text.lstrip())  # "hi there   " ← 왼쪽 공백만 제거
print(text.rstrip())  # "   hi there" ← 오른쪽 공백만 제거
```

`strip()`은 양 끝에서부터만 검사해서 제거하므로, 가운데 있는 공백/문자는 절대 지우지 않는다. `input().strip()`처럼 사용자 입력값을 다룰 때 자주 쓴다.

## f-string

f-string은 문자열 안에 변수와 계산식을 직접 넣을 수 있어 가독성이 가장 좋은 문자열 포매팅 방법이다(1세대 `%` 연산자, 2세대 `str.format()`보다 발전된 형태).

```python
price = 12000
print(f"{price:>10,}")  # 오른쪽 정렬(>) + 폭 10 + 천단위 콤마(,)
```

디버깅용 `=` 연산자: 변수 이름과 값을 동시에 출력해준다.

```python
score = 95
print(f"{score=}")  # score=95
```
