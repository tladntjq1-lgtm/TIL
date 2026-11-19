# 청킹(Chunking)과 LangChain 텍스트 분할기, Document

- 학습일: 2026-11-19
- 이어지는 글: [08. RAG 파이프라인](08-rag-pipeline-indexing-and-query.md), [10. 임베딩, 코사인 유사도, ANN, temperature](10-embedding-cosine-similarity-ann-and-temperature.md)
- 08번 글의 인덱싱 단계 ②번(문서 분할)을 자세히 다룬다. 이날 퀘스트는 5문항 중 4문항 정답이었다.

## 1. 청킹이 필요한 이유

**청킹(Chunking)** 은 긴 문서를 일정한 크기의 조각(청크)으로 나누는 전처리 과정이다. 필요한 기술적 배경은 세 가지다.

| 이유 | 설명 |
|---|---|
| **입력 창 크기 한계** | 언어 모델과 임베딩 모델은 한 번에 받을 수 있는 최대 토큰 수가 정해져 있다 |
| **의미 선명도 유지** | 너무 긴 문서를 통째로 임베딩하면 여러 주제가 섞여 벡터의 의미 표현력이 약해진다 |
| **검색 효율성** | 질문과 관련 높은 핵심 문단만 골라 전달하면 응답 속도와 정확도가 올라간다 |

> **청킹은 임베딩을 없애지 않는다.** 청크로 나눠도 각 청크는 임베딩 과정을 그대로 거친다. 청킹은 임베딩이 **잘 수행되도록 돕는** 앞 단계다. "임베딩 과정이 생략된다"는 설명이 청킹의 필요성으로 **적절하지 않은** 선지였다.

## 2. 텍스트 분할 방식 비교

| 방식 | 동작 | 특징 |
|---|---|---|
| `CharacterTextSplitter` (고정 크기 분할) | 글자 수로만 자름 | 문맥을 무시한다 |
| **`RecursiveCharacterTextSplitter`** (재귀적 문자 분할) | **문단 → 문장 → 공백** 순으로 구분자를 단계적으로 적용 | 의미 단위를 최대한 보존하는 **LangChain의 범용 표준 분할기** |
| Semantic Chunking (의미론적 분할) | **임베딩 유사도**를 기준으로 나눔 | 의미가 바뀌는 지점에서 자른다 |
| Metadata Filtering (메타데이터 필터링) | 분할 방식이 **아니라** 검색 범위를 좁히는 기법 | 출처, 연도 같은 정보로 후보를 거른다 |

- 한 줄 기억법: **재귀적 분할 = 큰 구분자부터 차례로 시도해서 의미가 덜 잘리게 자르는 방식.**
- 메타데이터 필터링은 이름에 "분할"이 없다. 분할기가 아니라 **검색 쪽 도구**다.

## 3. `chunk_overlap`: 이름까지 정확히

청크와 청크의 경계에서 문맥이 끊기지 않도록, **앞 청크의 끝부분을 다음 청크의 시작 부분에 중복해서 넣는** 설정값이 `chunk_overlap`이다. 교안 실습에서는 `chunk_overlap=10`으로 설정했을 때 앞 청크의 끝부분이 다음 청크 앞에 중복되어 나타났다.

```python
# 개념 이해용 예시 (LangChain)
from langchain_text_splitters import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=100,      # 청크 하나의 최대 길이
    chunk_overlap=10,    # 앞 청크의 끝 10글자를 다음 청크의 앞에 중복 (문맥 단절 방지)
)
chunks = splitter.split_text(long_text)
```

| 설정값 | 의미 |
|---|---|
| `chunk_size` | 청크 하나의 크기 |
| **`chunk_overlap`** | 이웃한 청크가 겹치는 길이 |

## 4. `Document`: 본문과 부가 정보를 한 객체로

LangChain에서 **본문 텍스트(`page_content`)와 출처·연도 같은 부가 정보(`metadata`)를 하나로 묶는 표준 클래스**가 `Document`다.

```python
# 개념 이해용 예시
from langchain_core.documents import Document

doc = Document(
    page_content="청킹은 긴 문서를 일정한 크기의 조각으로 나누는 전처리 과정이다.",
    metadata={"source": "교안 5장", "page": 3},
)
doc.page_content   # 청킹된 본문 텍스트
doc.metadata       # 출처, 페이지, 카테고리 등 부가 정보 (딕셔너리)
```

- `page_content` = 본문 / `metadata` = 딕셔너리 형태의 부가 정보.
- 본문과 메타데이터를 **일체형**으로 관리할 수 있어서, 검색 결과에 출처를 함께 붙이거나 메타데이터로 검색 범위를 좁힐 수 있다.
- 비유: **책(본문)과 책 뒤에 붙은 도서 카드(출처, 페이지)를 한 묶음으로** 들고 다니는 것.

## 5. `temperature=0` (복습)

RAG에서 LLM이 사실에만 근거해 무작위성이 적은 답을 하도록 할 때는 `temperature=0`이다. 1.5, 2.0, 10처럼 값이 커질수록 답변의 창의성과 무작위성이 커져서 사실 기반 응답에는 맞지 않는다. ([10번 글](10-embedding-cosine-similarity-ann-and-temperature.md)에서 정리한 내용이다.)

## 6. RAG 인덱싱 흐름에서의 위치

```text
문서 수집 → [문서 분할: RecursiveCharacterTextSplitter, chunk_overlap]
          → 각 청크를 Document(page_content, metadata)로 관리
          → 임베딩 → 벡터 저장소
```

## 오답노트 (2026-11-19)

| 문항 | 내가 쓴 답 | 정답 | 틀린 이유 |
|---|---|---|---|
| 3. 경계 문맥 단절 방지 설정값 | `overlap` | **`chunk_overlap`** | 개념은 맞지만 **파라미터 이름**이 정확하지 않았다 |
| 5. page_content와 metadata를 묶는 클래스 | `?` (모름) | **`Document`** | 클래스 이름을 몰랐다 |

- **3번**: 단답형은 코드에 쓰는 **정확한 이름**까지 외워야 한다. `chunk_size`와 짝으로 `chunk_overlap`.
- **5번**: `Document` = **page_content(본문) + metadata(출처 등)**. "본문과 부가 정보를 한 객체로" 라는 문장으로 기억한다.
- **좋은 점**: 1번은 **"가장 적절하지 않은 것"** 이라는 부정 문제였는데 올바르게 읽고 거짓 선지(임베딩 생략)를 골랐다. 지난 4번의 반복 실수가 이번에는 나오지 않았다.
- 맞힌 문제: 1번(청킹의 필요성), 2번(`RecursiveCharacterTextSplitter`), 4번(`temperature=0`).
