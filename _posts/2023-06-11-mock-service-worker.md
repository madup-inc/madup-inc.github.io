---
title: "MSW - 더 나이스한 목킹을 위한 고민"
excerpt: "MSW 를 이용한 브라우저 및 Node 환경에서의 목업코드 활용방법"
image: /mock-service-worker/cover-image.jpg
categories: [tech]
use_math: true
author: jake
---

## 🤔 고민
웹사이트를 개발하다 보면 백엔드와 프론트 간의 개발 속도 차이로 인한 문제와, API 응답 데이터에 의존하는 로직에 대한 테스트 코드 작성이 어려운 문제 등이 자주 발생합니다. 이러한 문제들은 현재 진행 중인 개발에 집중하기 어렵게 만들어줄 뿐 아니라 중복 코드와 같은 불필요한 작업을 초래합니다.

따라서 이번 글에서는 각각의 문제들이 발생하는 원인과 해결책에 대해 자세히 살펴보겠습니다. 함께 읽어보세요!

**< 정리 >**
1. **백엔드와 프론트 간의 개발속도 차이로 인한 문제** 
   - 백엔드와 프론트가 동시에 개발하면 발생되는 문제점

2. **API 응답 데이터에 의존을 갖는 로직에 대한 테스트코드 작성**
    - API를 호출하는 Custom hooks 혹은 Component 코드에 대한 테스트 코드 작성

<br/>
<br/>

### 1️⃣ 첫번째 고민
**( 백엔드와 프론트 간의 개발속도 차이로 인한 문제 )**

{% include image.html img="/mock-service-worker/1.png" caption="프로적트 개발 계획의 이상과 현실" %}

회사에서는 일반적으로 `기획 -> 백엔드 개발 -> 프론트 개발` 의 순서로 제품을 개발합니다. 하지만 실제로는 `기획 -> 백엔드, 프론트엔드 개발` 처럼 백엔드, 프론트엔드 개발이 동시에 진행되는 경우가 많습니다.

이러한 상황에서 발생하는 문제는, 프론트엔드에서 백엔드로부터 제공되는 API 응답 데이터를 기다리면서 화면을 구성해야 한다는 것입니다. 이로 인해 프론트엔드는 API가 완성될 때까지 다음 작업으로 넘어갈 수 없으며, 백엔드는 더 빠른 API 개발에 대한 압박을 받게 됩니다.

다행히도, 개발 커뮤니티에서는 이러한 문제를 해결하기 위해 다양한 방법들을 시도하고 있습니다.

{% include image.html img="/mock-service-worker/2.png" caption="백엔드에서 목업데이터 전달" %}

예를 들어 백엔드에서는 비즈니스 로직 구현 전에 Mockup 데이터를 반환하는 API를 먼저 제공하거나, Postman과 같은 외부 서비스를 이용하여 Mockup 용 API 서버를 제공하는 방법을 사용합니다.

```javascript
/* MockupData 선언 */
const mockupData = [
  {
    name: "홍길동",
    age: 28
  },
  {
    name: "고길동",
    age: 48
  },
  ...
]

/* Api 호출 대신에 MockupData를 사용 */
const response = mockupData// fetch(...)
```

또한, 프론트엔드에서는 Mockup API를 요청하는 대신에 자체적으로 Mockup 데이터를 만들어 사용하는 방법을 사용하기도 합니다.

이와 관련하여 검색을 해보면 다양한 API Mockup 처리 방법들이 나오는 것을 확인할 수 있습니다.

> * Mockup 처리 : 가짜 데이터를 만들어서 특정 로직의 응답결과로 대신 출력하도록 하는 작업
PostMan mock server, Beeceptor… 등등 많습니다.
참고자료: [Top 7 Free & Paid mock API tools (2022 Review)](https://testfully.io/blog/mock-api/#benefits-of-mock-apis)

<br/>

### 2️⃣ 두번째 고민
**( API 응답 데이터에 의존을 갖는 내부 로직에 대한 테스트코드 작성 )**

두번째 고민은 API호출 로직이 포함된 코드에 대하여 검증 로직이 필요할때 발생하게 됩니다.

> API 를 호출 과 동시에 해당 응답데이터를 가공하는 함수를 작성하거나 혹은 UI컴포넌트 의 결과가 의도한 대로 잘 나오는지 확인해야할 때

프론트엔드에서는 주로 백엔드에서 받아온 데이터로 UI를 그리는 행위가 빈번하기 때문에, 응답 데이터에 의존성을 갖는 컴포넌트 및 함수들이 많습니다.

중요한 로직은 순수 함수 형태로 분리해 의존성을 제거하는 것이 좋지만, 서비스가 복잡해지면 API 호출 관련 의존성을 분리하는 것이 어려워집니다. 따라서, API 호출 관련 의존성이 있는 코드를 테스트할 때는 Mockup 처리를 해야하는 불편함이 생깁니다.

간단한 컴포넌트의 경우 테스트 라이브러리에서 지원하는 Mockup 처리 관련 메서드를 이용하여 충분히 해결이 가능하지만, 프로덕트가 고도화 됨에 따라 비즈니스 로직과 UI 컴포넌트들이 더욱 복잡해지면 Mockup 처리해야 할 대상이 많아지기 때문에 처리가 어려워질 수 있습니다.

```jsx
/* 
  모든 Users 정보를 가져오는 부모 컴포넌트 
  특정 유저정보를 찾아 자식 컴포넌트에게 전달하는 역할
*/
const Parent = () => {
  const targetId = 1
  const [users, setUsers] = useState([])
  const targetUser = users.filter(({id})=> id === targetId)
  
  useEffect(()=> {
    (async ()=>{
      const users = await fetch('http://dev.api.me/users')
      setUsers(users)
    })()
  },[])
  
  return <Child user={targetUser} />
}

/* 
  특정 유저가 남긴 모든 댓글정보를 가져오는 자식 컴포넌트 
  특정 유저가 남긴 댓글을 모두 보여주는 역할
*/
const Child = ({user}) => {
  const [userComments, setUserComments] = useState([])
  useEffect(()=> {
    (async ()=>{
      const comments = await fetch(`http://dev.api.me/comments/user/${user.id}`)
      setUserComments(comments)
    })()
  },[])
  
  return (
    <ul>
      {
        userComments.map(({content}) => (
          <li>{content}</li>
        ))
      }
    </ul>
  )
}
```

> ⚠ 위의 예제코드는 자식 컴포넌트로 내려갈수록 호출되는 API 또한 늘어나는 경우를 이해시켜 드리기위해 제공되었습니다.

위의 코드는 부모 컴포넌트에서 API를 호출하여 자식 컴포넌트에게 데이터를 전달하고, 자식 컴포넌트에서는 해당 데이터를 가공하여 또 다른 API를 호출하는 상황을 보여줍니다. 이 경우 JEST를 사용하여 Mockup 처리를 하려면, 모든 의존하는 API 호출 함수를 상위 컴포넌트부터 하위 컴포넌트까지 파악하여 각각 별도의 Mockup 처리를 해주어야 합니다.

사실 테스트 코드 작성만 해도 상당히 신경써야 할일이 많은 일인데, Mockup 처리 작업으로 인해 온전히 테스트 코드에만 집중하기 힘든 상황에 놓여지게 된 상황인 것이죠… 😅

뿐만 아니라 팀내 여러 프론트엔드 개발자들이 각자 테스트 코드를 작성할 때, 동일한 API에 대해 각자 Mockup 처리하는 중복코드가 발생할 수 있으므로 이러한 중복을 최소화하기 위한 방법 또한 고민해야 합니다.

<br/>
<br/>

## 💡 해결
저희 팀은 두 가지 문제를 해결하기 위해  [Mock Service Worker(이하 MSW)](https://mswjs.io/docs/) 를 도입했습니다. 다양한 대안들이 있음에도 MSW를 선택한 이유는, 앞서 언급한 두 가지 문제를 MSW를 사용하면 모두 해결할 수 있기 때문입니다.

<br/>

### ▶️ MSW 란?

이야기에 들어가기 앞서 MSW란 무엇인지 간단하게 살펴보겠습니다.

MSW는 Mock Service Worker 의 약자로 이름에서 아실 수 있듯이 **Service Worker 라는 기술을 이용**해서 Mockup 작업을 돕는 라이브러리 입니다.

여기서 **서비스 워커(Service Worker)** 란, 최신 브라우저에서 지원되고 있는 기술로 웹 응용 프로그램, 브라우저, 그리고 (사용 가능한 경우) 네트워크 사이의 프록시 서버 역할을 합니다.

**서비스 워커를 이용하면 네트워크 요청이나 응답을 가로채서 조작하는것이 가능**한데요, MSW는 이러한 특성을 이용해서 실제 API 요청이 발생했을시 미리 준비해둔 목업 데이터로 대신 응답을 보내는 방식을 사용하고 있습니다.

<br/>

### ▶️ 기존 Mockup 처리방식

기존에는 네트워크 요청을 가로채기 위해 각각의 개발자가 네이티브 http, https, XMLHttpRequest 모듈을 다른 함수로 대체하여 Mockup 처리를 하거나, PostMan Mockup Server와 같은 목업서버를 직접 구축하여 테스트할 때 해당 서버에서 응답데이터를 받아 활용하는 방식을 사용했습니다.

<br/>

### ▶️ MSW 채택이유

MSW는 기존 방식과는 다르게 네트워크 요청이 발생하면 데이터만 교체하는 방식을 사용하여, 복잡한 처리 없이 서비스워커를 이용해 보다 간단하게 처리할 수 있습니다. 또한, MSW에서는 별도의 프로덕트 내부로직의 수정 없이 브라우저 환경과 테스트 환경(Node)에서 각기 다른 모킹 상태를 만들어 줄 수 있어, 앞서 언급한 두 가지 문제 상황을 해결할 수 있습니다.

또한, API를 만드는 것과 유사하게 개발할 수 있어 사용성 면에서도 큰 비용 없이 활용이 가능합니다. 이러한 이유로 저희 팀은 MSW를 채택하여 사용하게 되었습니다.

**[ 채택이유 정리 ]**

* Mockup 처리가 간단하다 
* 브라우저 및 Node 각 환경별로 목업데이터를 활용할 수 있다. 
* 학습에 큰 어려움이 없다.

<br/>
<br/>

## 📄 MSW 간단문서

<br/>

### ▶️ MSW 설치

```shell
npm install msw --save-dev
# or
yarn add msw --dev
```

<br/>

### ▶️ 서비스 워커 생성
```shell
npx msw init public/ --save
```
> Browser 환경에서 API 요청을 가로채기 위해 반드시 필요한 파일

<br/>

### ▶️ Handler (aka. Router) 생성

특정 API 경로로 요청이 시작되었을때 우리가 의도한 Mockup Data가 사용되게 하기 위해서는 **Mockup 처리를 원하는 API 경로와 이에 따른 Mockup Data를 맵핑하는 과정**이 필요합니다.

바로 해당 작업을 처리하는 곳이 Handler 인데요, 해당 코드를 살펴보면 Express 에서 Router를 작성하는 형태와 비슷하게 생긴 것을 확인할 수 있습니다.

코드는 아래와 같습니다.

```typescript
// src/mocks/handlers.ts
import { rest } from 'msw'

export const handlers = [
  /* user id를 이용해서 User 정보를 가져오는 API */
  rest.get('https://dev.api.me/user/:userId', (req, res, ctx) => {
    const { userId } = req.params
    return res(
      ctx.json({
        id: userId,
        firstName: 'John',
        age: 38,
      }),
    )
  }),
]
```

<br/>

### ▶️ Resolver (aka. Services) 생성

**Resolver는 위의 Handler에서** `(req, res, ctx) => {...}`  **형태의 코드를 일컬으며 백엔드에서 API 개발시 작성되는 서비스로직 과 유사**합니다. 각 API경로로 들어왔을때 보내진 정보를 갖고 데이터를 가공하여 반환하는 역할을 합니다.

코드는 아래와 같이 작성하며, express 와 동일하게 req, res, ctx 객체 활용이 가능하며, 이곳에서는 API호출시 같이 기입된 parameter, body, header 값에 대한 활용이 가능합니다. 해당 값들은 req객체에 들어가 있으며 자세한 사용법은 아래 코드 혹은 공식문서를 참고 바랍니다.

API의 `(req, res, ctx) => {...}`  을 **resolver.ts** 로 분리

```typescript
// src/mocks/handlers.ts
import { rest } from 'msw'
import { mockUser } from './resolvers'

export const handlers = [
  /* user id를 이용해서 User 정보를 가져오는 API */
  rest.get('https://dev.api.me/user/:userId', mockUser),
]
```

```typescript
// src/mocks/resolvers.ts
export const mockUser: ResponseResolver<RestRequest, RestContext>  = (req, res, ctx): any => {
  const { userId } = req.params
  return res(
    ctx.json({
      id: userId,
      firstName: 'John',
      age: 38,
    }),
  )
}
```

<br/>


### ▶️ Browser 에서 MSW 활성화

```typescript
// src/mocks/browser.ts
import { setupWorker } from 'msw'
import { BrowserHandlers } from './handlers'

const worker = setupWorker(...BrowserHandlers)

export default worker
```

브라우저 환경에서 미리 준비해둔 handler 를 사용하기 위해서는 MSW 의 setupWorker 메서드를 통해 worker 를 먼저 만들어 주셔야 합니다.

```jsx 
import worker from 'mocks/browser'

// App.ts
const App = () => {
  const isDevEnv = 'development' === process.env.ENV_VAR
  
  if (isDevEnv) {
    worker.start({
      onUnhandledRequest: 'bypass',
    })
  }
  
  return (
    <div>
    ...
    </div>
  )
}
```

그리고 React 프로젝트 가장 최상단 파일(App.ts)에서 worker.start 를 통해 활성화를 시켜주셔야 합니다.

{% include image.html img="/mock-service-worker/3.png" caption="msw 활성화" %}

만약 활성화가 되었다면, 개발자도구에서 console 창에 위와 같은 로그가 나타나게 됩니다.

{% include image.html img="/mock-service-worker/4.png" caption="msw 목업 api 동작" %}

그리고 미리 준비해둔 목업 API가 호출되었고, 서비스워커가 정상적으로 목업데이터를 반환했다면, 위와 같은 로그가 나타나게 됩니다.

<br/>

### ▶️ JEST 에서 MSW 활성화

```typescript
// src/mocks/server.ts
import { setupServer } from 'msw/node'
import handlers from './handlers'

const server = setupServer(...handlers)

export default server
```

Node 환경에서 미리 준비해둔 handler 를 사용하기 위해서는 MSW 의 setupServer 메서드를 통해 server 를 먼저 만들어 주셔야 합니다.

```typescript
import { rest } from 'msw'
import server from 'mock/server'

test('test code', () => {
  beforeAll(() => server.listen())
  afterEach(() => server.resetHandlers())
  afterAll(() => server.close())
  ...
})
```

그 다음에는 테스트 코드안에서 미리 만들어둔 server 를 통한 목업데이터를 사용하기 위해서는 위처럼 3가지의 작업이 필요합니다.

* 각 테스트코드를 돌기전에 서버를 활성화 시키기 위한 `server.listen()`
* 각 테스트코드가 돌고난 이후에 서버를 초기화 시키기 위한 `server.resetHandlers()`
* 모든 테스트코드가 종료된 이후에 서버를 종료하기 위한 `server.close()`

하지만  매 테스트 코드 작성시 위에서 언급한 코드를 넣어줘야 한다는 규칙은 다소 귀찮은 작업이 될 수 있습니다.

```javascript
// src/setupTests.js
import server from './mocks/server'

/* MSW: start */
beforeAll(() => server.listen())
afterEach(() => server.resetHandlers())
afterAll(() => server.close())
/* MSW: end */
```

그래서 **src/setupTests.js** 파일을 생성한 후 위와 같은 코드로 채워주세요

```javascript
// src/jest.config.js
module.exports = {
  setupFilesAfterEnv: ['./src/setupTests.js'],
  ...
}
```

그리고 **src/jest.config.js** 파일을 생성한 후 `setupFilesAfterEnv: ['./src/setupTests.js']`, 옵션을 넣어주세요.

이렇게 셋팅해두면, 매 테스트 코드가 돌기전에 **src/setupTests.js** 파일에 기입해둔 코드를 미리 실행해둘 수 있습니다.

<br/>

### ▶️ Util 함수들

해당 섹션에서 설명되는 코드는 공식문서에서 언급된 내용은 아니며, 필자가 셋팅 과정에서 편의를 위해 만든 함수이니 각 상황에서 잘 판단하셔서 활용하시기 바랍니다.

<br/>

(1) createUrl
* 매번 api등록시 도메인을 같이 기입해주는것이 번거로워서 만든 함수

```typescript
/*
 * const setPath = createUrl('https://exampleServer.com')
 * setPath('/users') --> https://exampleServer.com/users
 */
export const createUrl = (domain: string) => (path: string) => domain + path
```

(2) returnResolver
* api 호출시 Mockup data를 그대로 반환하는 상황에서 사용하기 위한 숏컷함수

```typescript
import { curry } from 'lodash'
import { ResponseResolver, RestContext, RestRequest } from 'msw'
/*
 * const FakeUser = {id:1, name: 'user1'}
 * const GetUserResolver = returnResolver(FakeUser)
 */
export const returnResolver =
  <T>(data: T): ResponseResolver<RestRequest, RestContext> =>
  (req, res, ctx): any =>
    res(ctx.status(200), ctx.json(data))
```

<br>
<br>

## 👍 좋은 경험

<br>

### ▶️ 빠른 개발 & 빠른 피드백 반영 가능!
Mock Service Worker(MSW)를 도입하면서 가장 좋았던 경험 중 하나는 초기에 빠른 개발이 가능했다는 것입니다. 이를 통해 개발 초기 스프린트에서 빠르게 실제 서비스가 동작하는 것처럼 구현이 가능했고, 결과물을 바탕으로 미리 디자이너와 기획자의 피드백을 받을 수 있었습니다.

{% include image.html img="/mock-service-worker/5.jpg" caption="출처: https://unsplash.com/ko/%EC%82%AC%EC%A7%84/5QgIuuBxKwM" %}

프론트엔드 개발의 가장 큰 고민 중 하나는, 개발자와 디자이너 간의 소통입니다. 종종 개발자와 디자이너가 서로 다른 생각으로 결과물을 만들어내기도 하죠. 이러한 이유 때문에 디자이너 검수 및 피드백을 반영하는 데에도 상당한 시간을 투자해야 하는 경우가 많습니다.

저의경우 이러한 상황에서 MSW도입이 큰 도움이 되었습니다. MSW를 이용하면 API가 나오기 전에 미리 디자인 검수를 받고, 문제가 발생한 부분을 수정할 시간을 가질 수 있습니다. 이렇게 실제 API에 의존하지 않고 화면을 구현하여 빠른 피드백을 받고 이를 토대로 빠른 수정을 통해 디자이너와 개발자 간 생각 차이를 좁혀나갈 수 있었습니다.

<br>

### ▶️ 보다 높아진 테스트 코드에 대한 집중력

프론트엔드 개발 과정에서 가장 큰 문제 중 하나는, 다양한 API 호출이 이뤄지는 복잡한 화면을 테스트하는 것입니다. 이러한 경우에는 UI 테스트를 위해 해당 화면과 관련된 모든 API를 목업처리해야 하는데, 이는 상당히 번거롭고 시간이 많이 소요됩니다.

또한, 목업데이터를 만드는 업무 자체도 중복으로 이뤄지는 경우가 있습니다. 개발자가 복잡한 데이터 구조를 이해하고, 각 API 목업데이터를 만들어야 하는데, 같은 로직을 구현하는 또 다른 개발자가 중복으로 목업데이터를 만들어내는 경우도 많았습니다.

하지만, MSW(Mock Service Worker)를 도입하면 API 자체에 대한 목업처리가 가능해지기 때문에, 이를 통해 팀원 모두가 공통된 목업 데이터를 사용할 수 있게 되었습니다. 또한, 목업 데이터 개발과 테스트 개발을 분리하여 개발할 수 있게 되었고, 각 테스트 코드마다 목업 데이터를 만들며 쏟아야 했던 시간들을 테스트 코드 자체에 더 쏟을 수 있는 환경이 조성되었습니다.

물론 MSW 도입이 목업데이터를 만드는 업무 자체를 없애주진 않았지만, 팀 내 모든 프론트엔드 개발자들이 매 API가 추가될 때마다 목업데이터를 하나씩 서로 추가해 주면서 각 개발자가 부담해야 하는 목업 데이터 개발양이 크게 줄어들 수 있었습니다. 따라서 MSW의 도입으로 더욱 효율적인 개발 환경이 조성되었다는 것을 알 수 있습니다.

<br>
<br>


## 참고자료
- [https://swagger.io/about/](https://swagger.io/about/)
- [https://openapi-generator.tech/](https://swagger.io/about/)


<br/>
<br/>
<br/>

👉 [매드업 채용 바로가기](https://recruit.madup.com)
