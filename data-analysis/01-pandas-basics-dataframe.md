# Pandas 기초와 DataFrame

## Series vs DataFrame

- `pandas`를 설치 후 `import pandas as pd`로 불러온다.
- **Series**: 1차원 데이터 (한 열)
- **DataFrame**: 2차원 데이터 (표 전체, 여러 열의 모음)

한 열만 선택하면 `Series`, 여러 열(리스트로)을 선택하면 `DataFrame`이 되는 차이를 실제로 확인해봐야 한다.

```python
import pandas as pd

df = pd.read_csv("data/orders.csv")

df["price"]        # Series (한 열)
df[["price", "qty"]]  # DataFrame (여러 열)
```

## 데이터 구조 점검하기

```python
df.head()      # 앞부분 미리보기 (기본 5행)
df.head(10)    # 괄호 안 숫자로 상위 데이터 개수 조절
df.tail()      # 뒷부분 미리보기
df.shape       # (행 수, 열 수)
df.columns     # 열 이름 목록
df.dtypes      # 각 열의 데이터 타입
df.info()      # 구조 전체를 한 번에 확인 (열, 타입, 결측치 개수 등)
```

## 열 선택과 Boolean 마스크로 행 필터링

```python
customers["age"] >= 30              # 비교식 → Boolean Series 생성
customers_over_30 = customers[customers["age"] >= 30]  # True인 행만 남음
```

이렇게 비교식으로 만든 `True`/`False` 값의 Series를 **불리언 마스크**라고 부른다. 여러 조건을 조합할 때는 `&`(and), `|`(or)와 **괄호**를 함께 써야 한다.

```python
mask = (customers["age"] >= 30) & (customers["city"] == "서울")
result = customers[mask]
```

## 숫자 데이터 요약

```python
df["price"].mean()      # 평균
df["price"].min()       # 최솟값
df["price"].max()       # 최댓값
df["price"].sum()       # 합계
df["price"].describe()  # 개수/평균/표준편차/사분위수 등을 한 번에 요약
```

## 결측값 개수 확인

```python
df.isna().sum()   # 열별로 결측값(NaN) 개수를 확인
```

## 자주 만나는 오류 체크 순서

1. `ModuleNotFoundError` — pandas가 현재 가상환경에 설치되어 있는지
2. `FileNotFoundError` — CSV 경로가 맞는지
3. `KeyError` — 존재하지 않는 컬럼명을 조회했는지
4. 자료형 문제 — 숫자처럼 보이는 값이 실제로는 문자열(Object)일 수 있음

AI에게 새로운 분석 코드를 바로 요구하기보다, **결과가 왜 그렇게 나왔는지 구조와 해석을 먼저 검토받고, 다음 질문을 이어가는 방식**이 더 남는 학습이었다.
