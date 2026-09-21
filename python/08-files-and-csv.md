# 파일 입출력과 CSV

## with open()으로 안전하게 파일 열기

```python
with open("memo.txt", "r", encoding="utf-8") as file:
    content = file.read()
```

- `"memo.txt"` → 파일 이름
- `"r"` → 읽기 모드
- `encoding="utf-8"` → 문자 인코딩

`with`를 쓰면 블록이 끝날 때 파일이 자동으로 닫히므로 안전하다.

## 모드: r / w / a

| 모드 | 의미 |
|---|---|
| `r` | 읽기 |
| `w` | 쓰기 (기존 내용을 덮어씀) |
| `a` | 추가 (append, 기존 내용 뒤에 이어 씀) |

```python
with open("result.txt", "w", encoding="utf-8") as file:
    file.write("파이썬 파일 쓰기 실습\n")
    file.write("두 번째 줄입니다.\n")

with open("result.txt", "a", encoding="utf-8") as file:
    file.write("새로운 내용을 추가합니다.\n")
```

## read(), readline(), 파일 반복

```python
# 전체 내용
with open("memo.txt", "r", encoding="utf-8") as file:
    content = file.read()

# 한 줄만
with open("memo.txt", "r", encoding="utf-8") as file:
    first_line = file.readline()

# 한 줄씩 반복 처리
with open("memo.txt", "r", encoding="utf-8") as file:
    for line in file:
        print(line.strip())  # 줄 끝의 \n 제거
```

## 경로 다루기: pathlib.Path

사람마다 파일을 저장해두는 위치가 다르기 때문에, 실행 파일 기준으로 경로를 계산하는 `BASE_DIR` 패턴을 사용한다.

```python
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parents[2]
file_path = BASE_DIR / "data" / "chapter16" / "memo.txt"
```

- `BASE_DIR`처럼 변수 이름을 대문자로 쓰는 것은 **상수처럼 취급**하겠다는 관례적 표시다.
- 파일이 없어도 코드는 작성할 수 있으며, 실행 시점에 `FileNotFoundError`로 확인하면 된다.

```python
file_path = Path("message.txt")
try:
    with open(file_path, "r", encoding="utf-8") as file:
        print(file.read())
except FileNotFoundError:
    print("파일을 찾을 수 없습니다. 경로를 확인해 주세요.")
```

## CSV 파일

CSV는 쉼표로 구분된 텍스트 파일이다. `csv.reader`는 각 행을 리스트로, `csv.DictReader`는 각 행을 딕셔너리(헤더를 key로)로 돌려준다.

```python
import csv

with open("data.csv", "r", encoding="utf-8") as file:
    reader = csv.DictReader(file)
    for row in reader:
        price = float(row["price"])  # CSV에서 읽은 값은 문자열이므로 형변환 필요

with open("output.csv", "w", encoding="utf-8", newline="") as file:
    writer = csv.writer(file)
    writer.writerow(["name", "score"])
```

> CSV를 떠올리면 자연스럽게 **pandas의 DataFrame**을 떠올려야 한다 — 표 형태 데이터를 다룰 때 pandas가 훨씬 편리하다. ([data-analysis/01-pandas-basics-dataframe.md](../data-analysis/01-pandas-basics-dataframe.md) 참고)

## 자주 만나는 예외

- `FileNotFoundError`: 파일 이름·경로 확인
- 인코딩 문제: `encoding="utf-8"` 명시로 대부분 해결
- `KeyError`: `DictReader`에서 없는 컬럼명 접근
- `ValueError`: 문자열을 숫자로 변환할 수 없을 때 (`int()`, `float()`)
