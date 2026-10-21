# Flexbox, Grid, 반응형 레이아웃 (Chapter 06~07)

회원관리 페이지(헤더 + 검색 툴바 + 회원 카드 목록)를 만들면서 레이아웃을 단계별로 익혔다. 코드는 `fullstack-web-course`의 `phase06`, `phase07`에 있다.

## Flexbox: 한 줄(1차원) 배치

헤더, 툴바, 카드 안의 버튼처럼 **가로 또는 세로 한 방향**으로 요소를 배치할 때 쓴다.

```css
.site-header {
    display: flex;
    justify-content: space-between; /* 제목은 왼쪽, 메뉴는 오른쪽 끝 */
    align-items: center;            /* 세로 가운데 정렬 */
}

.main-nav {
    display: flex;
    gap: 16px;                      /* 요소 사이 간격 */
}
```

| 속성 | 역할 |
|---|---|
| `display: flex` | 자식 요소들을 한 줄로 배치하는 컨테이너로 만듦 |
| `justify-content` | **주축(기본은 가로)** 방향 정렬 (`space-between` = 양 끝으로 벌림) |
| `align-items` | **교차축(기본은 세로)** 방향 정렬 |
| `gap` | 자식 사이의 간격 |
| `flex: 1` | 남은 공간을 최대한 차지 |

### 검색창이 남은 공간을 차지하게 하기

```css
.toolbar { display: flex; gap: 12px; align-items: center; }
.toolbar input { flex: 1; }   /* 버튼 두 개를 제외한 나머지 폭을 검색창이 다 차지 */
```

### 카드 안에서 세로로 쌓고, 버튼은 가로로 배치

```css
.member-card   { display: flex; flex-direction: column; }  /* 제목·이메일·과정이 세로로 */
.card-actions  { display: flex; gap: 8px; margin-top: 16px; } /* 상세·수정 버튼은 가로로 */
```

## Grid: 격자(2차원) 배치

회원 카드처럼 **행과 열이 있는 목록**은 Grid가 맞다.

```css
.member-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr); /* 같은 너비의 3열 */
    gap: 20px;
}
```

- `1fr`은 "남은 공간을 1비율로 나눠 갖기"라는 뜻이다. `repeat(3, 1fr)`은 3등분.
- **Flexbox는 한 방향 배치, Grid는 행·열 격자 배치**라고 구분하면 헷갈리지 않는다.

## 반응형 웹 (Chapter 07)

### 1. viewport 메타 태그

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

이게 없으면 모바일 브라우저가 페이지를 데스크톱 너비로 가정하고 축소해서 보여준다. 반응형의 출발점이다.

### 2. 유연한 컨테이너: `width` + `max-width`

```css
.page-container {
    width: 90%;        /* 작은 화면에서는 화면의 90% */
    max-width: 1200px; /* 큰 화면에서는 1200px을 넘지 않음 */
    margin: 0 auto;    /* 가운데 정렬 */
}
```

### 3. Mobile First + 미디어 쿼리

**작은 화면(모바일) 스타일을 기본값**으로 먼저 쓰고, 화면이 넓어질 때만 `min-width` 미디어 쿼리로 덮어쓴다.

```css
/* 기본 (모바일): 1열, 헤더는 세로 배치 */
.member-grid  { display: grid; grid-template-columns: 1fr; gap: 16px; }
.page-header  { display: flex; flex-direction: column; }

/* 태블릿 (768px 이상): 2열, 헤더는 가로 배치 */
@media (min-width: 768px) {
    .member-grid { grid-template-columns: repeat(2, 1fr); }
    .page-header { flex-direction: row; justify-content: space-between; align-items: center; }
}

/* 데스크톱 (1024px 이상): 3열 */
@media (min-width: 1024px) {
    .member-grid { grid-template-columns: repeat(3, 1fr); }
}
```

흐름: **1열 → 2열 → 3열**, **세로 헤더 → 가로 헤더**.

### 4. 화면 밖으로 넘치는 것 막기

```css
input, select, textarea { width: 100%; box-sizing: border-box; } /* padding이 너비에 포함되어 가로 스크롤 방지 */
img { max-width: 100%; height: auto; }                           /* 이미지가 컨테이너보다 커지지 않게, 비율 유지 */
```

- `box-sizing: border-box`가 없으면 `width: 100%` + `padding`이 합쳐져 부모보다 커지면서 가로 스크롤이 생긴다.

## 정리

- 배치 방향이 **한 줄**이면 Flexbox, **격자**이면 Grid.
- 반응형은 `viewport` 메타 태그 → 유연한 너비(`%` + `max-width`) → **Mobile First 미디어 쿼리** 순서로 쌓는다.
- 폼 요소와 이미지는 따로 `max-width`/`box-sizing`으로 넘침을 막아줘야 한다.
