## Cypress란?

**Cypress는 JavaScript 애플리케이션을 테스트하는 툴(도구)이다**

> 실제로 [공식사이트](https://www.cypress.io/) title에 이렇게 잘 적혀있다.
> testing for javascript | write, run, debug | Cypress

최근 **체쿠리(사이드 프로젝트 멤버)**팀 분들이랑 프로젝트를 진행하면서, QA 과정 중에 주요 사용자 흐름을 수동으로 검증해야 했다. 전체 페이지를 계속 테스트하는 건 단조롭고 반복적인 작업이라 큰 부담으로 느껴졌고, 놓치는 부분도 있을 수 있었다.

그래서 배포하는 과정마다 일일히 수동으로 QA 할 수 없었기에 테스트 툴을 도입하게 되었다.
(체쿠리는 소규모 음악학원을 위한 출석부 웹 서비스입니다.🙇‍♂️)

## 왜 선택했을까?

cypress이외에도 다른 테스트 툴 도구들이 있다. jest, playwright 등...
이러한 과정에서 목표는 크게 세가지가 있었다.

> 1. E2E 테스트
> 2. 커뮤니티 크게 활성화 되어있는 (예를들어 유명한것들? ㅎㅎ)
> 3. 크로스 브라우저 테스트 지원

좁혀보니 Cypress와 Playwright로 고민하게 되었다.

그래서 직접 둘다 가볍게 사용해보면서 느꼈던 점

- Cypress는 명령 로그를 통해 애플리케이션 상태를 쉽게 추적
- Cypress 브라우저 화면에서는 테스트 진행 상황을 실시간으로 확인할 수 있어 직관적
- 설정이 간편함

위의 느껸던 점들이 Cypress를 선택하게 했다!

```
npx cypress open
```

cypress를 설치 후, 위 해당 명령어로 테스트 실행 및 디버깅을 편리하게 도와주는 기능들을 제공하는 GUI chrome과 electron 데스크톱을 지원해준다.
![](https://velog.velcdn.com/images/qjatn0955/post/c5e86b16-3219-4a00-bf65-088273d12c7e/image.png)

직접 스펙을 추가할 수 있고 위에서 언급한 2번 3번의 내용인 명령 로그로 애플리케이션 상태 추적, 브라우저 화면에서 테스트 진행상황 실시간으로 확인 가능! 👍🏻
![](https://velog.velcdn.com/images/qjatn0955/post/bd1b4d82-b145-4f2b-b76a-bb17a450c51e/image.gif)

## 학생 등록 테스트 코드 작성기

먼저, 테스트 할 동작들을 묶어주는 그룹 **describe** 작성하고 테스트 케이스가 실행 되기 전, 실행 되어야 할 로그인 후 메인 페이지로 이동하였다. 페이지에 접속 하기위해서는 로그인 후 토큰이 담겨야 입장이 가능하기 때문이다.

(해피패쓰 테스트를 작성해둔 케이스이기에 다른 엣지 케이스도 작성할 예정!)

```
describe('학생 등록 페이지', () => {

 before(() => {
    cy.Login()
    cy.visit('/book')
  })

}
```

그럼 이제 페이지에 접속할 권한이 생겼으니 **it**을 통해 실제 테스트 케이스를 정의 해보았다.

> 1. 첫 번째 출석부 선택
> 2. 학생 관리 페이지로 이동
> 3. 학생 등록 시작
> 4. Step 1: 기본 정보 입력
> 5. Step 1 완료 후 다음 단계 이동
> 6. Step 2: 추가 정보 입력
> 7. 최종 제출 및 성공 확인

```
describe('학생 등록 페이지', () => {
  before(() => {
    cy.Login()
    cy.visit('/book')
  })

  it('학생 등록 시나리오', () => {

    cy.get('[data-cy="book-0"]').should('be.visible').click()

    cy.get('[data-cy="navigate-attendee"]').click()

    cy.get('[data-cy="icon-attendee-create"]').click()

    cy.StudentFormStep1({
      name: new Date().toISOString().slice(-5),
      birth: '20230404',
      gender: 'male',
    })

    // "다음으로" 버튼 클릭하여 Step2로 이동
    cy.get('[data-cy="stepSubmit"]').click()

    cy.StudentFormStep2()

    cy.get('[data-cy="submit-attendee-create"]').click()

    // Then
    cy.get('[data-cy="toast-success"]').should('exist')
  })
})

```

## Cypress Commands & env

> Cypress Commands

**Cypress 커스텀 커맨드**는 자주 사용하는 명령어들을 재사용 가능한 함수로 만들어, 테스트 코드 내에서 cy.login()이나 cy.StudentFormStep1 처럼 호출할 수 있게 해준다.

이렇게 하면 **중복 코드를 제거하고, 테스트의 가독성과 유지보수성**을 높일 수 있다.

아래 코드는 cypress/support/commands.ts 파일에 작성할 수 있는 로그인 커스텀 커맨드 예제이다.

```
Cypress.Commands.add('Login', () => {
  return cy
    .request('POST', `${Cypress.env('api_server')}/auth/signin`, {
      username: 'dkandkdlel',
      password: 'test123123!!',
    })
    .then((response) => {
      const accessToken = response.body.data.accessToken
      return cy.setCookie('accessToken', accessToken).then(() => {
        return response
      })
    })
})
```

> Cypress 환경 변수 설정 옵션

Cypress에서는 여러 가지 방법으로 환경 변수를 설정할 수 있다. 이 중, **cypress.config.ts**과 **cypress.env.json**을 사용하고 있다.

예를 들어, cypress.config.ts 파일에서는 기본값을 아래와 같이 설정한다.

```
// cypress.config.ts

import { defineConfig } from 'cypress'

export default defineConfig({
  e2e: {
    baseUrl:"http://localhost:3000",
  },
})

```

api_server URL 정보를 깃허브에 노출시키고 싶지 않기 때문에, 이를 cypress.env.json 파일에서 관리하고 있다

```
// cypress.env.json
{
  "api_server": "https://api.server.url/api/v1"
}

```

민감한 정보를 cypress.env.json에서만 관리하여 깃허브에 노출되지 않도록 할 수 있다. 개발 환경에서는 cypress.env.json에 정의된 값이 cypress.config.ts에서 설정한 값을 덮어쓰게 되어, 민감한 정보를 보호할 수 있다. 👍🏻

## 끝으로..

E2E 테스트이외에 유닛 테스트(Unit Test)도 해야하는거 아닌가? 라고 생각했었다. **하지만 빈번한 기능 변경과 디자인 수정으로 인해 테스트 코드도 지속적으로 수정**해야 하고 앞으로 있을 변경에 업데이트 하기가 쉽지 않을것 같다는 부담감과 시간적 여유가 없었다.

다른 테스트 방식을 도입하면서 작은 단위의 안정성도 중요하지만 사용자 흐름에서 안정성이 더 가치있다고 생각되었다. 그래서 고민 끝에, E2E 테스트만 적용하기로 결정했다.
상황상 최선의 선택이었다.😭

항상 모든 프로젝트에 좋은 걸 다 적용하면 제일 베스트겠지만, 전부 다 적용할 수 없으니 주어진 상황에서 최선의 선택을 하는 게 제일 베스트이니까!☺️
