# Next.js App Router 기초와 TypeScript 선택 속성

쇼핑몰 프로젝트(상품 목록 → 상품 상세)를 만들면서 배운 내용 정리.

## App Router: 폴더 이름이 곧 URL

App Router에서는 `app/` 아래의 **폴더 구조가 그대로 URL 경로**가 되고, 각 폴더 안의 **`page.tsx`** 파일이 그 주소의 화면이 된다.

| 파일 위치 | 접속 주소 |
|---|---|
| `app/products/page.tsx` | `/products` (상품 목록) |
| `app/products/[id]/page.tsx` | `/products/1`, `/products/2` ... (상품 상세) |

## 동적 라우팅: `[id]` 폴더

- 대괄호로 감싼 **폴더명(`[id]`)** 이 동적 세그먼트가 된다. 1이든 2든 어떤 값이 들어와도 이 폴더가 받는다.
- 그 안에 반드시 **`page.tsx`** 를 두고, URL 파라미터로 `id`를 전달받는다.

```
app/
└── products/
    ├── page.tsx          → /products
    └── [id]/
        └── page.tsx      → /products/:id
```

### 헷갈렸던 오답 구조: `app/products/page/[id].tsx`

- `page`는 폴더 이름이 아니라 **파일 이름**(`page.tsx`)이다.
- `[id]`가 **폴더가 아니라 파일**로 되어 있다. `[id].tsx` 같은 파일 방식은 예전 **Pages Router** 스타일이라 App Router의 폴더 기반 라우팅 규칙과 맞지 않는다.

## 목록 렌더링: `.map()`

배열 형태의 가상 상품 데이터를 순회하며 각 상품을 `ProductCard` 컴포넌트로 반복 렌더링할 때는 자바스크립트 **`.map()`** 을 쓴다. 배열의 각 요소를 변환해 **새로운 배열(여기서는 JSX 요소들)** 을 반환한다.

```tsx
{products.map((product) => (
  <ProductCard key={product.id} product={product} />
))}
```

## 페이지 이동: `<Link>`

일반 `<a>` 태그는 클릭할 때마다 전체 페이지를 새로고침하지만, Next.js의 **`<Link>`** 컴포넌트는 **화면 깜빡임 없이** 페이지를 전환한다. SPA 방식의 클라이언트 사이드 네비게이션이다.

```tsx
import Link from "next/link";

<Link href={`/products/${product.id}`}>상세 보기</Link>
```

## TypeScript: 선택 속성(optional property)

`interface`에서 속성 이름 뒤에 **`?`** 가 붙으면 "없어도 되는" 선택 속성이다. 신청서에서 이름·전화번호는 필수, 별명은 선택인 것과 같다.

```ts
interface Product {
  id: number;        // 필수 - 상품마다 꼭 있어야 하는 식별자
  price: number;     // 필수
  imageUrl: string;  // 필수
  isNew?: boolean;   // 선택 - 신상품이 아닌 상품은 생략 가능
}
```

## 프로젝트 범위: 필수 구현 vs 제외 범위

이번 쇼핑몰 프로젝트는 **프론트엔드 화면 구현**에 집중하는 범위였고, 아래 항목들은 **제외 범위**였다.

- 회원가입/로그인 폼
- 실제 데이터베이스 연결
- 장바구니 서버 저장
- PG사 결제 모듈

반대로 가상 데이터 기반의 상품 목록/상세 페이지, 동적 라우팅, 컴포넌트 렌더링 등은 필수 구현 범위다. "무엇을 만들지"뿐 아니라 "**무엇을 만들지 않을지**"를 먼저 정하는 것이 프로젝트 범위 관리의 시작이다.

## 오답노트 (2026-10-14)

- **동적 라우팅 구조를 `[id].tsx` 파일 형태로 착각함** → App Router는 "`[id]` **폴더** + 그 안의 `page.tsx`"라고 외우기. 폴더 이름이 주소, `page.tsx`가 화면.
- **TypeScript 선택 속성을 `id`로 고름** → `?`가 붙은 것만 선택 속성이다. `id`처럼 식별자 역할을 하는 값은 필수인 경우가 대부분이고, 있어도 되고 없어도 되는 `isNew?` 같은 부가 정보가 선택 속성이 된다.
