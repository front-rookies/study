# 페이지 구성 방식 - SPA, MPA

## 🔹 **SPA (Single Page Application)**

- 하나의 페이지 안에서 어플리케이션이 구동되는 방식
- 전체 페이지를 다시 불러오지 않고 **필요한 부분만 동적으로 갱신**
- ex) `React`, `Vue`, `Angular`

## 🔹 **MPA (Multi Page Application)**

- 사용자 상호작용에 따라 **여러 개의 페이지가 제공**되는 구조
- 페이지 전환 시 **서버로부터 새 HTML을 매번 로드**
- ex) `PHP`, `JSP` 등

---

# 렌더링 방식

| 방식                             | 설명                                              |
| -------------------------------- | ------------------------------------------------- |
| **CSR (Client Side Rendering)**  | 클라이언트 측에서 렌더링 처리                     |
| **SSR (Server Side Rendering)**  | 서버에서 HTML을 렌더링하여 클라이언트에 전달      |
| **SSG (Static Site Generation)** | 빌드 타임에 미리 정적인 페이지를 생성해 놓는 방식 |

> 📌주로 SPA + CSR/ MPA+SSR로 결합시켰지만 최근 들어 다양한 기술과 프레임워크 등장으로 **SPA에서도 SSR이 가능**하다.

## 착각할 수 있는 오해

### SSR == RSC(React Server Components)?

SSR은 렌더링된 HTML을, RSC는 RSC payload라는 JSON 포맷으로 온다.

SSR은 초기 렌더링을 서버에서 처리하고 JS 코드를 추가해 인터랙션을 추가한다.
반면에, RSC는 React에서 생성된 React element를 직렬화(RSC payload)하여 클라이언트로 전송하고, 클라이언트에서 초기 렌더링 및 확장을 수행하는 형태이다.

개념적으로 비슷하지만, 매커니즘과 전송 결과물에 차이가 있다.

### React에서 ServerSideComponent 사용하면 페이지 구성 방식은 무조건 MPA인가?

아니다. RSC는 SSR과 같이 전체 페이지를 렌더링하는 것이 아니라, 서버에서 렌더링되는 `컴포넌트`이다.

React에서 RSC를 사용하면 SSR+CSR이 혼합된 SPA라고 할 수 있다.

하나의 HTML에서 route 처리가 된다. -> SPA
RSC 컴포넌트는 서버에서 렌더링을 한다 -> SSR
클라이언트 측 컴포넌트 -> CSR

즉, 전체 페이지가 교체되지 않고 일부 컴포넌트만 서버에서 렌더링되어 클라이언트로 전달되기 때문에, 이 구조는 MPA로 볼 수 없다.

또 다른 예로, a 태그를 사용하면 MPA 방식으로 동작할 수 있다.

```
<a href="">
```

a 태그는 브라우저가 전체 페이지를 새로 요청하기 때문이다.

---

# CSR (Client Side Rendering)

## 🔸 동작 방식

- 서버에서 **빈 HTML 뼈대**와 JS 파일을 전달
- 클라이언트가 JS를 다운로드해 동적으로 DOM을 구성

## ✅ 장점

- 빠른 반응 속도 (라우팅 등 클라이언트에서 처리)
- 서버 부하 적음
- 화면 깜빡임 없음
- TTV(Time to View)와 TTI(Time to Interactive)의 간격이 없음

## ❌ 단점

- 초기 로딩 속도 느림
- 빈 HTML 구조라 SEO(검색 엔진 최적화)에 불리함

---

# SSR (Server Side Rendering)

![](https://velog.velcdn.com/images/goodbyebada/post/ba3f52d7-94ba-4e8b-b5b1-3d30139b5189/image.png)
렌더링 된 HTML을 볼 수 있다.

## 🔸 동작 방식

- 서버에서 렌더링된 HTML과 JS를 클라이언트에 전달
- 브라우저가 HTML을 먼저 렌더링 → JS로 로직 연결

## ✅ 장점

- 초기 로딩 속도가 빠름
- 검색 엔진 최적화(SEO)에 유리

## ❌ 단점

- 화면 깜빡임 존재
- TTV와 TTI 간격 발생
- 서버 부하 발생

---

# SSG (Static Site Generation)

## 🔸 동작 방식

- 모든 웹 페이지를 **빌드 시점에 정적으로 생성**
- 요청 시 미리 생성된 페이지를 전달

## ✅ 장점

- 로딩 속도 매우 빠름
- SEO에 매우 유리 (이미 렌더링된 상태)
- 서버, 클라이언트가 할 일이 적음

## ❌ 단점

- 콘텐츠가 자주 바뀌는 사이트에는 부적합
- 사이트 변경 시 **전체 재빌드 필요**

---

# SSR vs SSG 비교

| 기준            | SSR                   | SSG                |
| --------------- | --------------------- | ------------------ |
| 생성 시점       | 요청 시 서버에서 생성 | 빌드 시 미리 생성  |
| 데이터 업데이트 | 실시간 반영 가능      | 실시간 반영 어려움 |

---

# CSR 단점 개선 방법 🛠️

## 느린 초기 로딩 속도 개선

- **Code Splitting**  
  → SPA는 초기 실행 시에 필요한 웹 리소스를 다운 받음
  → `lazy loading` 을 적용해 한 번에 다운 받지 않도록 함
  → React의 `React.lazy`, `Suspense`

- **Tree Shaking**  
  → 사용하지 않는 모듈 제거로 번들 크기 감소
  ```js
  // 좋지 않은 예
  import * as util from "./util";
  ```
  → 위와 같이 import하면 사용하지 않는 함수도 포함됨

## SEO 개선

- **Pre-rendering (사전 렌더링)**  
  → CSR이더라도 특정 페이지를 미리 렌더링해 검색 엔진이 인식할 수 있도록 처리

## SSR/SSG 부분 도입

- 초기 로딩 속도 개선 + SEO 최적화
- 전체를 SSR로 하지 않고 **특정 페이지만 SSR/SSG 적용 가능**

---

# SSR/SSG 도입 방법

### 1. 프레임워크 없이 도입

- Express, Nest 등으로 별도 서버 구성  
  → 백엔드 지식 필요, 복잡도 증가

### 2. 프레임워크 활용

| 프론트엔드 | 프레임워크          |
| ---------- | ------------------- |
| React      | `Next.js`, `Gatsby` |
| Vue        | `Nuxt.js`           |
| Angular    | `Universal`         |

**Next.js SSR 예시:**

```jsx
export async function getServerSideProps() {
  const res = await fetch("https://.../data");
  const data = await res.json();
  return { props: { data } };
}

function Page({ data }) {
  return <div>{data.title}</div>;
}

export default Page;
```

서버에서 미리 데이터를 fetch한 뒤 페이지 컴포넌트의 props로 넘기는 방식(page Router)

---

# CSR / SSR / SSG / Universal 선택 기준

| 상황                                | 권장 렌더링 방식          |
| ----------------------------------- | ------------------------- |
| 유저 상호작용 많음                  | **CSR**                   |
| 개인정보 포함, 검색엔진 노출 불필요 | **CSR**                   |
| 검색 노출 필요(SEO), 정보 고정      | **SSG**                   |
| 검색 노출 필요(SEO), 정보 자주 변경 | **SSR**                   |
| 초기 속도 빠름 + SEO + 반응성 중요  | **Universal (CSR + SSR)** |

# CSR / SSR / SSG 활용 예시

- CSR
  - 소셜 미디어 플랫폼(예 : 페이스북)
    - 로그인한 유저를 위한 서비스 (개인 정보 포함)
    - 검색 엔진 노출 불필요
    - 유저 상호작용 많음 (좋아요, 댓글 달기) -> 실시간 빠른 화면 변동
    - 실시간으로 정보 업데이트
- SSR
  - 예) 쿠팡, 아마존등
    - 검색 엔진 노출 필요함
    - 가격이나 할인율와 같은 정보가 자주 변경됨
    - 메인 스크립트가 무거운 경우, 서버에서 렌더링함으로써 초기 로딩 시간을 줄일 수 있다.
- SSG
  - 예) 마케팅 웹사이트, 기술 문서
    - 기술 문서는 정보가 버전 업데이트 외에는 고정
    - 마케팅 웹사이트 또한 정보가 자주 변경되지 않음

---

벨로그에서 보기
https://velog.io/@goodbyebada/CSRSSRSSG

---

출처 및 참고 자료

https://www.youtube.com/watch?v=YuqB8D6eCKE

https://www.daleseo.com/spa-ssg-ssr/

[https://velog.io/@seungchan\_\_y/CSR-SSR#-ssr의-동작-과정과-특징](https://velog.io/@seungchan__y/CSR-SSR#-ssr%EC%9D%98-%EB%8F%99%EC%9E%91-%EA%B3%BC%EC%A0%95%EA%B3%BC-%ED%8A%B9%EC%A7%95)
https://teawon.github.io/cs/csr-ssr/
https://www.youtube.com/watch?v=jEJEFAc8tSI

---

## ✍️ 추가로 더 알아볼 개념

- [ ] RSC
- [ ] ISR
