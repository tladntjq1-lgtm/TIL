# EC2 키 페어 권한, 스왑 메모리, 미들웨어 실행 순서

- 학습일: 2026-11-06
- 이어지는 글: [04. response_model, N+1 문제, 멱등성](04-response-model-n-plus-1-and-idempotency.md)
- 이날 퀘스트는 5문항 모두 정답이었다. 새로 나온 개념(1~3번)을 정리하고, 어제·그저께 배운 내용(4~5번)이 다시 나온 것을 복습한다.

## 1. 키 페어(.pem) 파일은 왜 `chmod 400`인가

EC2 서버에 SSH로 접속할 때는 비밀번호 대신 다운로드한 **키 페어 파일(.pem)** 을 쓴다.

```bash
chmod 400 my-key.pem                 # 소유자만 읽기 가능, 나머지는 아무 권한 없음
ssh -i my-key.pem ubuntu@<서버주소>
```

- 리눅스(SSH)는 키 파일을 **소유자 외의 다른 사용자도 읽을 수 있게 열려 있으면 보안 위험으로 판단하고 접속을 거부**한다. (`UNPROTECTED PRIVATE KEY FILE` 경고)
- 그래서 소유자만 읽도록 **400**으로 제한한다.
- 파일 전송 속도, 환경 변수 동기화, Nginx 렌더링과는 관계가 없다.

| 숫자 | 대상 | 400의 뜻 |
|---|---|---|
| 첫째 자리 `4` | 소유자 | 읽기(r)만 |
| 둘째 자리 `0` | 그룹 | 권한 없음 |
| 셋째 자리 `0` | 그 외 사용자 | 권한 없음 |

> 키 페어 파일은 비밀번호와 같다. Git 저장소에 올리거나 공유하지 않는다. (이 TIL 저장소에도 실제 키 파일, 서버 IP는 올리지 않는다.)

## 2. 스왑(Swap): 디스크를 메모리처럼 빌려 쓰기

EC2 프리티어 같은 환경은 RAM이 작아서 서버를 올리다 메모리가 부족해질 수 있다. 메모리가 부족하면 리눅스의 **OOM Killer**가 프로세스를 **강제로 종료**한다.

- **스왑(Swap)**: 실제 RAM이 부족할 때 **하드디스크 공간을 메모리처럼 임시로 사용**하는 기술.
- 스왑을 잡아두면 메모리가 순간적으로 부족해도 OOM Killer에 의한 강제 종료를 막을 수 있다.
- 디스크는 RAM보다 훨씬 느리므로 **임시 방편**이다. 근본적으로는 메모리 사용량을 줄이거나 인스턴스 사양을 올려야 한다.

## 3. 미들웨어는 양파 껍질: 나중에 등록한 것이 바깥쪽

미들웨어는 요청이 API 함수에 도착하기 전과 응답이 나가기 전에 끼어들어 공통 처리(로깅, 인증, CORS 등)를 하는 계층이다.

```python
@app.middleware("http")
async def middleware_a(request, call_next):    # 먼저 등록
    print("A 요청 진입")
    response = await call_next(request)
    print("A 응답 나감")
    return response

@app.middleware("http")
async def middleware_b(request, call_next):    # 나중에 등록
    print("B 요청 진입")
    response = await call_next(request)
    print("B 응답 나감")
    return response
```

```text
요청 :  클라이언트 → B → A → 실제 API 함수
응답 :  클라이언트 ← B ← A ← 실제 API 함수
```

- **나중에 등록한 미들웨어가 가장 바깥쪽 껍질**이 되어 요청 시 **가장 먼저** 실행된다.
- 응답은 반대 방향으로, 안쪽부터 바깥쪽으로 빠져나간다.
- 한 줄로: **요청은 "나중에 등록한 것부터", 응답은 "먼저 등록한 것부터".**

> 위 코드는 순서를 보여주려고 만든 예시다. 출력 순서는 B 진입 → A 진입 → (API 함수) → A 나감 → B 나감이다.

## 4. 복습: N+1 문제와 `joinedload()` (→ [04](04-response-model-n-plus-1-and-idempotency.md))

- N+1 문제의 원인은 **지연 로딩(`lazyload`)** 이고, 해결은 **`joinedload()`** 로 JOIN 1회에 같이 가져오는 것.
- `defer()`와 `load_only()`는 연관 테이블이 아니라 **특정 컬럼의 로딩**을 제어하는 옵션이라 N+1과 목적이 다르다.

## 5. 복습: `exclude_unset=True` (→ [03](03-type-hints-and-pydantic-v2.md))

- PUT 수정에서 **클라이언트가 실제로 보낸 필드만** 추출: `payload.model_dump(exclude_unset=True)`.
- 안 보낸 필드가 기본값(`None`)으로 기존 데이터를 덮어쓰는 것을 막는다.

## 오늘의 결과

- 5문항 전부 정답. 4번(`joinedload`)과 5번(`exclude_unset`)은 직전 퀘스트에서 틀렸던 유형인데 이번에 맞혔다.
- 오답노트 없음. 다음에 흔들리기 쉬운 부분은 **미들웨어 순서(요청 vs 응답 방향)** 와 **`chmod` 숫자의 자리별 의미**다.
