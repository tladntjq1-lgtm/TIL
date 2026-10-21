# JavaScript 기초: DOM 이벤트, 입력 검증, 함수·배열·객체 (Chapter 08~09)

사칙연산 계산기(Chapter 08)와 회원 데이터 관리(Chapter 09)를 만들며 정리했다. 코드는 `fullstack-web-course`의 `phase08`, `phase09`에 있다.

## 1. DOM 요소 가져오기와 이벤트 등록

```js
const num1Input = document.getElementById('num1');
const calcBtn   = document.getElementById('calcBtn');

calcBtn.addEventListener('click', function () {
    // 버튼이 클릭될 때 실행할 코드
});
```

- `document.getElementById('id')`: HTML의 `id`로 요소를 찾는다.
- `addEventListener('click', 함수)`: 클릭 같은 이벤트가 일어났을 때 실행할 함수를 등록한다.
- `<script src="app.js">`는 요소들이 만들어진 **`</body>` 직전**에 두어야, 스크립트가 실행될 때 요소를 찾을 수 있다.

## 2. 입력값 검증은 순서가 중요하다

```js
const rawNum1 = num1Input.value.trim();   // 입력값은 항상 "문자열"

if (rawNum1 === '' || rawNum2 === '') { alert('두 숫자를 모두 입력해 주세요.'); return; }   // 1. 비었는지
if (isNaN(rawNum1) || isNaN(rawNum2)) { alert('올바른 숫자를 입력해 주세요.'); return; }    // 2. 숫자인지
const num1 = Number(rawNum1);                                                              // 3. 변환
if (operator === '/' && num2 === 0)   { alert('0으로 나눌 수 없습니다.'); return; }         // 4. 0으로 나누기
```

- `input.value`는 숫자를 입력해도 **문자열**이라, 계산 전에 `Number()`로 바꿔야 한다. (`"1" + "2"`는 `"12"`)
- **빈 값 검사를 `isNaN`보다 먼저** 해야 한다. `isNaN('')`은 `false`다. 빈 문자열이 `Number('')`에서 `0`으로 바뀌기 때문이다. 그래서 `isNaN`만으로는 빈 입력을 걸러내지 못한다.
- `.trim()`으로 앞뒤 공백을 제거해야 `"   "` 같은 입력도 빈 값으로 처리된다.
- `return`으로 검증에 실패하면 함수를 즉시 끝낸다 (Python의 [예외 처리](../python/09-exceptions.md)에서 "정상 입력만 테스트하면 안 된다"고 했던 것과 같은 생각이다).

## 3. 연산자 분기: `switch`

```js
switch (operator) {
    case '+': result = num1 + num2; break;
    case '-': result = num1 - num2; break;
    case '*': result = num1 * num2; break;
    case '/': result = num1 / num2; break;
}
alert(`결과: ${num1} ${operator} ${num2} = ${result}`);
```

- `break`가 없으면 아래 `case`까지 계속 실행된다.
- 백틱(`` ` ``)과 `${}`를 쓰는 **템플릿 리터럴**은 Python의 f-string과 같은 역할이다.

## 4. 함수, 배열, 객체

```js
function greet(name, age) {
    return `안녕하세요, ${age}세 ${name}님! 환영합니다.`;   // 값을 반환
}

let users = [
    { id: 1, name: '김철수', age: 25, isActive: true },
    { id: 2, name: '이영희', age: 30, isActive: false },
];
```

**배열 안에 객체**를 담는 구조는 Python의 [리스트 안의 딕셔너리](../python/07-dictionaries.md)와 같다. 한 행이 객체 하나인 표 형태 데이터다.

## 5. `map()`과 `filter()`

| 메서드 | 하는 일 | 결과 |
|---|---|---|
| `map(fn)` | 각 요소를 **변환**해서 | 같은 길이의 새 배열 |
| `filter(fn)` | 조건이 `true`인 요소만 **골라서** | 길이가 줄어들 수 있는 새 배열 |

```js
// map: 모든 회원을 <li> 문자열로 변환
const userHtmls = users.map(user => `<li>이름: ${user.name} / 나이: ${user.age}세</li>`);
userListEl.innerHTML = userHtmls.join('');   // 배열을 하나의 문자열로 합침

// filter: isActive가 true인 회원만
const activeUsers = users.filter(user => user.isActive);
```

- 둘 다 **원본 배열을 바꾸지 않고** 새 배열을 반환한다.
- `map` 결과는 배열이라 `innerHTML`에 넣기 전에 `.join('')`으로 문자열로 합쳐야 한다. 안 하면 쉼표가 같이 찍힌다.
- Next.js 퀴즈에서 상품 목록을 `.map()`으로 카드 컴포넌트로 바꿨던 것이 같은 개념이다. ([nextjs/01](../nextjs/01-app-router-and-typescript-basics.md))

## 6. 배열에 추가하고 화면 다시 그리기

```js
const lastId = users.length > 0 ? users[users.length - 1].id : 0;   // 삼항 연산자: 비었으면 0부터
const newUser = { id: lastId + 1, name: userName, age: userAge, isActive: userAge < 50 };
users.push(newUser);   // 배열 끝에 추가
renderUsers();         // 화면을 그리는 코드를 함수로 만들어 두었기 때문에 다시 호출만 하면 된다
```

핵심 패턴: **데이터(배열)를 바꾸고 → 그리는 함수를 다시 호출**한다. 화면을 직접 고치는 대신 데이터와 렌더링을 분리하면 추가/삭제가 쉬워진다.

## 짚고 넘어간 점

- **`!userAge`로 나이를 검증하면 `0`도 "입력 없음"이 된다.** `0`은 falsy라서다. 빈 값 여부와 숫자 여부는 따로 검사하는 편이 정확하다. (Python의 [Falsy 값](../python/01-variables-and-datatypes.md)과 같은 원리)
- **`innerHTML`에 사용자 입력을 그대로 넣으면 위험하다.** 이름 칸에 `<script>`나 HTML 태그를 넣으면 그대로 해석될 수 있다(XSS). 사용자 입력은 `textContent`로 넣는 것이 안전하다. 이번 실습은 학습용이라 `innerHTML`을 썼지만 실제 서비스에서는 피해야 한다.
- **HTML 오타는 화면에 그대로 보인다.** Chapter 08 `index.html`의 `</select>s`처럼 태그 뒤에 글자 하나가 붙으면 에러 없이 화면에 `s`가 찍힌다. HTML은 문법 오류를 알려주지 않고 조용히 렌더링하므로 브라우저로 직접 확인해야 한다.
