---
title: 프론트엔드 테스트 팁
excerpt: 프론트 테스트 종류와 개발 경험담 그리고 유닛테스트 가이드까지
image: front-test-tips/2.png
categories: [tech]
author: willy
---

# 프론트엔드 테스트 팁

## 개요

안녕하세요 벌써 병역특례 3년차! 곧 민간인이 되는 Integration Engineering 팀 윌리입니다! 본격적으로 글을 읽으시기 전에 어떤 내용인지 이 글의 성격을 알려드리려 합니다.

### 대상

비즈레버 프론트개발에 대해 궁금하신 분, 왜 다양한 프론트 테스트가 필요한지 궁금하신 분, 리액트 컴포넌트를 유닛테스트할 때 swr 때문에 고생하시는 분이 이 글을 읽으실 때 더 좋은 정보를 얻으실 것 같습니다.

`유닛, 통합 테스트 팁` 챕터는 개발자를 대상으로 기본적인 react, javascript, typescript에 대해 알고 있는 사람을 가정하고 작성했습니다.

목차는 아래와 같습니다.

### 목차

1. 테스트의 종류
2. 비즈레버 프론트는 어떤 테스트를 하나?
3. 유닛, 통합 테스트 팁
    1. 테스트 라이브러리와 기초
    2. 컴포넌트 테스트
    3. React Hook, SWR Hook 함수 테스트
    4. 타이머가 있는 경우
    5. Hook을 쓰는 페이지 테스트 (통합 테스트)
4. 후기: 완벽한 테스트를 위해

## 테스트의 종류

### 테스트의 종류

타입이나 컨벤션을 잡는 정적 테스트를 제외하고 일반적으로 소프트웨어 개발에서 이야기하는 테스트는 보통 3가지를 말합니다.

`unit test`, `integration test`, `e2e(end to end) test`입니다.

만약에 테스트에 대한 지식이 없으신 분은 차례대로 더 포괄적이고 큰 범위의 테스트를 한다 생각하시면 이해에 도움이 됩니다.

저는 3가지 테스트를 게임 스타크래프트의 저그 종족에 비유하고 싶습니다. 초보인 상대를 이기기 위해서는 저글링, 히드라, 뮤탈 유닛 중 하나만 잘 사용해도 승리할 수 있지만, 중수 이상의 실력자는 3가지 유닛을 한 게임에 다 사용해야 게임에서 이길 수 있습니다.

테스트도 다양한 유닛을 활용하는 저그처럼 필요에 따라 다양하게 방법을 채용하면 소프트웨어 품질을 더 잘 보증 할 수 있다고 생각합니다.

### unit

unit은 유닛테스트, 단위테스트라고 많이들 말씀하시는데 코드에서 가장 작은 한 묶음인 함수, 클래스같은 모듈을 테스트하고 프론트엔드에서는 파일구조를 컨테이너/컴포넌트로 나눌 때 컴포넌트에 해당되는 파일에 테스트를 작성하면 유닛테스트로 봅니다. 저그 종족에서 안 뽑을 수 없는 저글링처럼 가장 기본적인 단위 테스트입니다.

### integration

integration, 통합테스트는 여러 모듈끼리 연결이 있는 코드를 테스트합니다. 객체끼리 데이터를 주고받고 여러 모듈을 핸들링하는 코드의 테스트 통합테스트로 보면 됩니다. 프론트엔드에서는 파일구조를 컨테이너/컴포넌트로 나눌 때 컨테이너에 해당하는, 즉 페이지 자체를 테스트하는 코드를 짜면 여기에 해당된다 볼 수 있습니다.

예를 들면 아래 같은 테스트입니다.

```html
api를 패칭 값에 따라 원하는 컴포넌트를 뿌려주고 상호작용이 잘 되는지 확인한다.
```

### e2e

e2e 테스트는 end to end의 줄임말로 모듈, 코드 입장에서 품질을 보증하던 위 2가지 방법과 다르게 코드를 통해 사용자 입장에서 테스트하는 방법을 말합니다. 예시로 들면 Cypress, Selenium 같은 툴로 실제 사용자처럼 개발 환경으로 구성된 사이트에 들어가서 테스트를 하는 코드를 작성하면 e2e라고 말할 수 있습니다. 비즈 레버의 예를 들면 "광고 생성을 테스트하기 위해서 광고 상품 정보 페이지에 입력하고 광고 자산(매체)을 연동 페이지로 이동해 페이지와 페이스북, 구글 정보를 입력했을 때 정상 동작한다"가 있습니다.

### 테스트의 영역

unit과 integration은 코드 레벨에서 테스트를 작성하기 때문에 같은 라이브러리를 쓰고 같은 프로젝트 내에서 작업해서 실제 테스트 코드를 작성할 때 영역이 겹치기도 합니다. 하지만 e2e는 사용자 입장에서 실제 랜더링을 해서 앞의 두 가지보다 휠씬 테스트 시간이 길고 별도의 프로젝트 파일에서 테스트하는 코드를 작성하기도 합니다.

### 역할이 달라서 다 필요하다

이처럼 각 테스트의 영역과 목적이 다르기 때문에 더 좋은 코드, 더 좋은 제품을 만들기 위해서는 3가지를 다 해야됩니다. 하지만 **현실은 이렇게 방대한 테스트 코드를 만들면 막상 기능 개발보다 테스트코드에 신경을 쓸 수 밖에** 없습니다.

현실적인 점을 고려 할 때 굳이 필요가 없다고 생각되는 중요치 않은 코드나 기능, 페이지는 테스트 코드를 작성하지 않기도 합니다.

무조건 모든 종류의 테스트를 커버리지 100% 맞추기보다 테스트가 무의미한 코드는 커버리지에서 제외하는 것도 현실을 고려한 좋은 선택이죠.

## 비즈레버는 어떤 테스트를 하나?

이 챕터는 왜 3가지 테스트를 다 사용하고 IE팀 프론트 개발자들은 요즘 어떤 방식으로 테스트를 만드는지에 대한 내용입니다. 개발 히스토리에 대한 내용이기 때문에 리액트 테스트 코드를 만드는 방법 이외에 관심이 없으시다면 건너띄고 읽으셔도 무방하십니다.

### 좋은 건 다 쓰고 있다.

결론부터 말하면 비즈레버 프론트는 unit, integraiton, e2e를 다 하고 있고 최근에는 테크스펙을 작성하면서 어떤 코드를 테스트할지 간단하게 먼저 정해두고 개발에 들어갑니다. 일종의 TDD 같지만, 정석에 맞게 테스트 먼저 작성하고 코드를 만들고 다시 돌아와서 테스트를 작성하는 그런 방식은 또 아닙니다.

### 태초마을의 유닛테스트와 e2e 테스트

비즈레버 초기에는 컴포넌트를 다 랜더링해서 테스트하는 것이 상당히 어렵다 판단해서 함수 단위의 테스트를 작성했습니다. 이유는 react hook을 사용하는 컴포넌트의 원하는 **랜더링 횟수**가 정확하지 않을 수 있고 목업해야되는, 페이즈를 테스트 할 때는 **목업해야되는 SWR이 너무 많아** 유닛테스트 작성에 시간을 많이 사용했고 나중에는 함수만 별도 파일로 분리해서 작성했습니다.

함수만 따로 테스트 하다 보니 함수는 다 테스트가 되는데 실제 렌더링이 잘 되는지나 통합테스트가 제대로 확인되지는 않았습니다. 그래서 사용자 입장에서 테스트가 필요하다는 이야기가 나와 그때 (아직 IE 팀이 있기 전에) 지금은 정직원이 되셔서 CT팀에서 잘 개발해주고 계시는 제이크가 Selenium으로 돌아가는 e2e 테스트 코드를 작성했었습니다. Selenium에 녹화 기능을 통해 실제로 사용자처럼 행동하고 js로 남기는 기능을 쓰셨던걸로 기억합니다.

Selenium으로 기록된 js파일의 쿼리셀렉터(css className, id) 쓸 때 없이 복잡하고 프로젝트 파일도 비즈레버와 별도로 분리해서 관리하다 보니 테스트를 자주 돌리지도 않고 관리의 불편함이 있었습니다. 그래서 지금은 이직하신 다쥬가 비즈레버 프론트 레포지터리에 cypress를 설치하고 전부 이관하셨던 스토리가 있습니다.

이런 히스토리로 인해 현재 비즈레버에는 기본적인 함수 유닛테스트와 cypress를 활용한 e2e 테스트가 존재합니다.

### 애증의 대상 e2e, cypress

[cypress](https://www.cypress.io/)는 e2e 테스팅 라이브러리입니다. 예제도 좋고 다양한 함수를 제공하며 [git-action](https://github.com/features/actions)에서도 돌릴 수 있는 아주 좋은 라이브리러입니다.

Cypress에 대해서는 이미 많은 블로그에서 좋다는 글이 이미 작성되어 있어서 자료가 많습니다. e2e가 필요하다면 도입을 추천해 드립니다.

Cypress를 활용한 e2e 테스트는 디자인과 실제 동작, 통합테스트까지 포괄적으로 모든 걸 할 수 있고 좀 극단적으로는 페이지가 아니라 컴포넌트만 개별로 랜더링해서 테스트하는 코드를 짤 수도 있습니다.

이런 범용성 좋은 Cypress와 초반에 고생하면서 Cypress 테스트 코드를 만들어주신 여러 프론트엔드 개발자의 노력으로 e2e 테스트를 유용하게 사용했고 많은 버그를 사전에 잡았습니다.

하지만

단점 없이 완벽하면 제가 이 글을 안 썼겠죠? 큰 단점이 있습니다. 그건 바로 너무 느리다는 점입니다. ㅠㅠ

{% include image.html img="front-test-tips/1.png" caption="Cypress 실행 결과, 길어서 한 화면에 다 보이지도 않는다…" %}

유닛테스트는 실제로 랜더링을 하지는 않기 때문에 속도가 충분히 나와서 eslint같은 정적 테스트와 함께 수시로 돌릴 수 있습니다. 예시로 비즈레버 프론트에서는 [husky](https://typicode.github.io/husky/#/)를 활용하여 원격저장소에 push 직전에 유닛테스트를 돌리고 유닛테스트가 실패하면 push를 막습니다. 이렇게 유닛테스트는 항상 잘 관리되고 해당 기능에 수정 후 테스트를 확인하지 않으면 푸쉬에 실패하게 되어 있어 유용하지만 e2e 테스트를 똑같이 세팅하면 e2e는 속도가 너무 느려서 push 직전에 몇십분 개발자를 기다리게 할 수 없었습니다.

e2e 시간이 최근에는 테스트가 계속 늘어나다 보니 어느 순간부터 모든 테스트를 한 번 컴퓨터로 돌리는 시간이 30분 가까이 걸리기 시작했습니다.

이런 문제 때문에 나중에는 배포직전이 아니면 Cypress를 관리하지 않게 되었습니다.

단점을 해결해보고자 `git-action`에 올려서 해결을 보려 했으나 이미지처럼 제가 사고를 치고 말았습니다...

{% include image.html img="front-test-tips/2.png" caption="나중에 자수하고 광명을 찾았다" %}

갑자기 나름의 자랑을 해보면 저는 프론트를 주력으로 다루는 개발자치고 [devops](https://aws.amazon.com/ko/devops/what-is-devops/)에 상당히 관심이 많은 개발자입니다. 지금도 매드업 개발 블로그에는 제가 4년전 인턴 때 git-aciton도 없던 시절에 jenkin ci, circle ci, travis ci의 예제를 구축해보고 하나를 고르는 글을 썼을 정도로 관심이 많습니다. ([링크](https://tech.madup.com/travis1/)1)([링크](https://tech.madup.com/travis2/)2)

덕분에 이런 시간과 관련된 이슈를 보면 CI로 해결해야겠다는 생각부터 합니다.

```
아! CI에서일정 주기로 한 번씩 테스트를 돌리게 하자, 특히 병렬로 돌아가면 아주 펀쿨섹시한 코드가 나오겠지?
```

그래서 아래처럼 action에서 여러 컨테이너를 만들고 테스트를 대분류로 나줘서 실행시키고 결과를 취합하는 ci를 작성했습니다.

```yaml
//캐싱, 인증쪽은 생략
name: cypress-ci

on:
  pull_request:
    branches:
      - main
      - develop
      - 'feature/**'
jobs:
  e2e_test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        containers: [ ] # 디렉터리 구조에 따라여기서 필요한 만큼 컨테이너 추가 작성
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v1
        with:
          node-version: 12
      - name: yarn install
        run: yarn
      - name: Cypress.io
        uses: cypress-io/github-action@v2
        with:
          start: yarn start
          wait-on: 'http://localhost:3000'
          wait-on-timeout: 120
          browser: chrome
          spec: cypress/integration/lever/${{ matrix.containers }}/*
```

초기에 상정했던 git-action에 e2e를 넘기는 것에는 문제가 없었습니다.

{% include image.html img="front-test-tips/3.png" %}

하지만! git-action 사용 시간을 고려못했습니다. 그래서 문제가 발생했습니다.

{% include image.html img="front-test-tips/4.png" caption="나중에 자수하고 광명을 찾았다" %}

문제의 원인은 git-action 환경은 실제 컴퓨터 환경보다 스펙이 낮아서 잘못된 로직으로 짜여진 일부, 특히 그 중에서도 Cypress의 랜더링 시간을 코드에서 수동으로 대기하는 로직에 있는 함수들이 action 환경에서는 시간을 초과해서 에러가 생기고 그것을 **다시 재실행하면서 테스트 전체 시간도 너무 많이 늘어나는** 최악의 상황이 되어버렸습니다.

Cypress는 브라우저를 띄우지 않고 cli 환경에서 실행시켜도 **백그라운드에서는 브라우저를 띄우고** **실제 랜더링** 되는 작업을 거칩니다. 그러다보니 스펙이 낮은 환경에서는 일반 컴퓨터에서는 충분했던 wait 함수 시간이 ci 환경에서는 대기시간이 지나도 다 랜더링 되지 않아서 테스트에 실패하는 경우가 발생했습니다.

결국 Cypress 위주로 사용하면서 좋은 점도 많았지만 나쁜점도 많아서 Cypress와 e2e를 활용해 대부분의 테스트를 커버하겠다는 큰 야망은 이뤄지지 못하고 다시 통합테스트, 유닛테스트에 더 신경 쓰게 된 것이 최근에 일입니다.

## 유닛, 통합 테스트 팁

글 앞부분에서는 3가지 테스트 종류와 각각의 장점이 있지만 결국 모든 테스트를 다 세팅해야 된다는 이야기를 매우 길게 했었습니다. 투머치토커 기질이 있어 앞부분에 이미 다른 블로그 글 1개 수준의 이야기를 했었는데 이제부터는 [react-testing-library](https://testing-library.com/docs/react-testing-library/intro/)를 써보면서 배운 팁에 대해 공유하려고 합니다.

### 테스트 라이브러리와 기초

기본적인 유닛테스트 라이브러리 소개와 기초적인 예시를 다룹니다.

### 사전지식1: 테스트 환경

모든 가이드와 예시는 react testing library, jest를 쓸겁니다.

유닛테스트 라이브러리도 여러 종류가 있지만 react testing library를 굳이 사용하면서 글을 쓰는 이유는 [create-react-app](https://create-react-app.dev/)에 기본으로 들어있기 때문입니다. react testing library 외 많이 비교되는 라이브러리로는 에어비엔비에서 만든 [enzyme](https://enzymejs.github.io/enzyme/)가 있습니다. 저는 둘 다 같이 써도 상관이 없다 생각하지만 아무래도 테스트 코드의 스타일 통일성을 생각하면 하나만 써도 무방합니다. 제가 쓰면서 느낀 특징은 아래와 같습니다.

#### enzyme vs react-testing-library

enzyme

```html
에어비엔비에서 만들었고 깊은/얕은 테스트 구분이 있음
장점: props, setState 같은 함수 덕분에 컴포넌트에 프롭스를 바꾸는게 편리함
단점: 클릭 액션 같은 실제 랜더링 될 때에 동작은 테스트는 불편
결론: 상태관리에 따라 mock한 특정 함수가 호출 되는지 확인하고 싶으면 편리하다.
```

react-testing-library

```html
깊은/얕은 구분 없이 테스트를 위해 실제 dom을 랜더링하고 wrapper에 감싸여 있음
장점: 우리가 익숙한 dom API 사용 가능 그리고 실제로 어떤 텍스트나 html, css가 랜더링 되었는지 함수를 제공해서 용이하고 dom에 있는 이벤트를 발생시키기 편리함
단점: enzyme에 비해 컴포넌트 상태 변경이 불편 그리고 컴포넌트 내부 상태 변경 로직에 따라 랜더링이 여러 번 바뀌기 때문에 waitFor를 사용해 원하는 화면이 랜더링 될 때까지 기다려야함 이런 점 때문에 Swr이랑 같이 사용하면 실수할 여지가 있음
결론: 랜더링 된 결과를 테스트하고 싶을 때 유용하다.
```

#### react-testing-library 직접 설치하고 관리 할 때

기본적으로 create react app에 react testing library가 있긴 하지만 막상 create react app 이하 cra의 package.json을 보면 이름이 조금 다른 모습을 볼 수 있습니다. 이름은 react testing library가 맞지만 **패키지명은 @testing-library/~** 같은 이름을 하고 있어서 수동으로 설치하는 분들은 이점에 주의해주셔야합니다.

```shell
//cra를 쓴다면 불필요
npm install -d @testing-library/[원하는 패키지]
```

#### 사전지식2: jest

react-testing-library는 [jest](https://jestjs.io/)를 기본세팅으로 합니다.

라이프사이클의 경우 거의 모든 테스트 라이브러라가 비슷하기 때문에 [링크](https://jestjs.io/docs/api)로 대체하고 jest에 대해 더 소개하겠습니다! jest는 핵심기능은 expect와 mock 2가지를 제공합니다.

expect는 파이썬이나 자바같은 언어에서 테스트 코드를 만들어보신 분들은 한 번에 이해하실겁니다. junit에서 assertEqual, assertTrue, assert~ 으로 시작하는 함수를 보신적 있으신가요?

jest는 다른 언어에서는 assert~ 으로 객체를 확인하는 함수 역활을 expect 라는 함수가 그 역활을 해줍니다. 자바나 파이썬과 다르게 javascript 특징에 맞게 react-testing-library (이름과 다르게 패키지 이름은 @testing-library) 안에 있는 jest-dom을 이용하면 dom 객체를 확인하는 함수들이 추가됩니다. 예시로는 아래처럼 dom 객체를 다루면서 사용이 가능합니다.

```html
//@testing-library/jest-dom

const dom = getByText('someting')
expect(dom).toBeInTheDocument() //dom이 있으면 통과

const button = getByTestId('mad-button`)
expect(button).toHaveClass('.ok') // .ok 클래스가 있으면 통과
expect(button).not.toHaveAttribute('disalbed') // disabled 상태의 버튼이 아니면 통과

const validVorm = getByTestId('valid-form')
expect(form).toBeValid() // form 이 valid면 통과
```

모든 함수를 소개할수는 없고 리액트에서 테스트를 익숙치 않으신 분들은 탭 하나 더 띄우고 항상 참조하기를 추천드립니다.

[https://github.com/testing-library/jest-dom](https://github.com/testing-library/jest-dom)

mock은 테스트에 유용하게 쓸 수 있게 특정함수, 패키지를 목킹합니다.

`jest.fn`, `jest.mockImplementation`, `jest.mockReturnValue`로 목업을 불러오고 `expect`로 몇번 호출 됐는지 무엇이 인자로 들어갔는지 마지막 결과가 어떤지 다 확인이 가능합니다.

아래는 많이 쓰는 예시 몇 가지를 뽑아보았습니다.

```js
const mockFn1 = jest.fn()
mockFn()
expect(mockFn1).toHaveBeenCalledTimes(1) // mock 함수가 1번 호출되면 통과, 함수의 호출 횟수를 확인

const mockFn2 = jest.fn()
const fn2Arg = { name: 'willy', age: 24}
mockFn(fn2Arg)
expect(mockFn2).toHaveBeenLastCalledWith(fn2Arg) // fn2Arg와 동일한 인자가 마지막으로 호출 되었을 때 함수의 인자로 들어왔으면 통과

const mockFn3 = jest.fn().mockReturnValue('hello') //hello를 리턴하는 목업함수
expect(mockFn3()).toBe('hello')

const mockFn4 = jest.fn().mockImplementation((number => number * number)) //특정 로직으로 작동하는 목업함수를 만듬
expect(mockFn4(2)).toBe(4) // 2*2 = 4
```

jest에 가장 기본적인 함수일부를 소개했는데 hook이나 패키지 목업이 가능합니다.

드디어 테스트를 위한 사전 지식 준비는 다 끝났습니다.

### 컴포넌트 테스트

컴포넌트 테스트라고 하는 것들이 사실 디자인과, 로직 2가지 종류의 테스트입니다.

1번) 인자에 따른 디자인의 변화를 검증한다.

2번) 컴포넌트 내부에 있는 함수를 검증한다.

저는 컴포넌트 내부의 함수들은 최대한 분리해서 테스트하는 것을 추천합니다.

하지만 함수만 테스트하면 프론트앤드 테스트라고 할 수 없기 때문에 디자인을 테스트하기 위해 랜더링을 해서 테스트하는 예시를 소개하겠습니다.

css에 신경을 벌 써서 버튼처럼 안 생겼지만… 버튼 예시입니다.

**create-react-app, typescript 세팅에서 아래 코드만 바로 추가하면 동일하게 해보실 수 있습니다.**

기본적으로 리액트 지식이 있는 분들을 대상으로 하기 때문에 세팅은 생략하고 바로 코드로 들어가겠습니다.

```tsx
// 불필요한 코드는 다 생략
// src/component/Button.tsx
export default ({ styleType = 'default ', ...props}) => {
  return <button className={styleType} {...props}/>
}
```

```css
// src/component/Button.css
.mad-button {
    font-size: 13px;
    line-height: 16px;
    letter-spacing: -0.3px;
    text-align: center;
    margin: 0;
    outline: none;
    cursor: pointer;
    border: 1px solid;
    padding: 4px 20px;
    height: 58px;
    border-radius: 10px;
}
.mad-button.default {
    background: #fff;
    border-color: black;
    color: black;
}
.mad-button:not(.default) {
    color: #fff;
}
.mad-button.danger {
    background: red;
    border-color: #ffccc7;
}
.mad-button.primary {
    background: blue;
    border-color: #bae7ff;
}
```

```html
//App.tsx 내부 display: 'flex', gap: '30px'
<div>
    <Button styleType="default">
      default
    </Button>
    <Button styleType="danger">
      danger
    </Button>
    <Button styleType="primary">
      primary
    </Button>
</div>
```

버튼 3개를 나열한다고 하면 아래처럼 나올겁니다.

{% include image.html img="front-test-tips/5.png" caption="좀 이상하게 생겼지만 버튼이다" %}

styleType 인자를 className으로 주는 버튼 컴포넌트가 있습니다. 아래처럼 테스트 작성이 가능합니다.

```jsx
// src/component/button.test.js
import { render } from '@testing-library/react'
import Button from './Button'
describe('버튼 테스트', () => {
  test('styleType이 className으로 사용됨', () => {
    const component = render(<Button styleType="danger">버튼</Button>)
    expect(component.container.querySelector('.mad-button.danger')).toBeInTheDocument()
  })
})
```

cmd에서 `yarn test`, `npm test`로 결과를 확인하면 아래와 같이 동작합니다.

{% include image.html img="front-test-tips/6.png" caption="결과" %}

컴포넌트를 랜더링 할 때는 [render](https://testing-library.com/docs/react-testing-library/api#render)함수를 사용합니다. `render`는 container와 각종 쿼리 함수들을 제공합니다.

container는 실제 html body에 해당되는 그러니깐 실제 랜더링된 html dom객체(ReactElement 타입)입니다. [https://developer.mozilla.org/ko/docs/Web/API/Document](https://developer.mozilla.org/ko/docs/Web/API/Document) 이거로 생각하시면 됩니다.

element를 검색할 때는 **screen이 더 권장**되는 방법이긴 합니다. 하지만 **각종 쿼리 함수들이 있음에도 container를 사용한 이유**는 검색함수들 중에서 **className으로 dom 객체를 찾는 함수는 없기 때문**입니다.

`screen`에서 제공하는 쿼리 함수들은 `getByText`, `getByRole`,  `getByTestId` 처럼 텍스트나 미리 설정한 element attribute로 element를 찾는데 `screen`을 사용해서 테스트 하는 방법해도 괜찮습니다. 하지만 우리가 만든 코드가 아니라 외부 디자인 컴포넌트를 사용하는 경우 디자인 컴포넌트의 디자인을 커스텀하면서 className은 알고 있지만 어떤 attribute가 들어있는지 정확하게는 모르는 경우가 많기 때문에 저는 container.`querySelector()`를 사용해서 검색하기를 추천드립니다.

screen과 render에 제공하는 검색함수와 거의 비슷한 함수들을 제공하고 있습니다. 어떤 함수들이 있는지는 아래 링크를 확인해주세요

[https://testing-library.com/docs/queries/about/#screen](https://testing-library.com/docs/queries/about/#screen)

[https://stackoverflow.com/questions/61482418/react-testing-library-screen-vs-render-queries](https://stackoverflow.com/questions/61482418/react-testing-library-screen-vs-render-queries)

### React Hook, SWR Hook 함수 테스트

SWR과 Hook 함수를 테스트 하는 방법에 대해서 알려드리려 합니다. 예시는 SWR를 사용하지만 React hook이면 모두 비슷한 방법으로 테스트가 가능합니다. 요점은 [renderHook](https://react-hooks-testing-library.com/)을 사용하는 것이기 때문입니다.

[SWR](https://swr.vercel.app/ko)은 [next.js](https://nextjs.org/)를 만든 팀에서 만든 패칭, 캐싱 라이브러리로 각광받고 있습니다. 비즈레버에서는 상태관리 필요한 복잡한 프로젝트는 아니고 캐싱으로 충분히 대체된다 판단하고 redux와 관련 라이브러리를 걷어내고 SWR 데이터 패칭과 리액트 hook 위주로 개발하고 있는데 SWR과 캐싱 라이브러가 궁금하시면 CT팀의 첼시가 작성한 글을 읽어보세요!

[https://tech.madup.com/react-query-vs-swr/](https://tech.madup.com/react-query-vs-swr/)

SWR을 사용하는 예시를 위해 session storage 키값에 따라 값을 저장하고 캐싱해서 읽어오는 hook을 만들고 테스트 코드를 만들어보겠습니다.

일단 hook 입니다. `src/hook/useSession.ts` 를 추가해주세요

```tsx
import useSWR from "swr"

export default (sessionKey: string) => {
  const set = (key, value) =>  {
    sessionStorage[key] = JSON.stringify(value)
  }
  const get = key => {
    if (!sessionStorage[key]) {
      return null
    }
    const value = sessionStorage[key]

    try {
      return JSON.parse(value)
    } catch (e) {
      return value
    }
  }
  const { data, mutate, isValidating } = useSWR(
    `session-${sessionKey}`,
    () => get(sessionKey),
    {
      refreshInterval: 10000,
      //설정 생략
    }
  )
  return {
    data: data === null ? undefined : data,
    isValidating,
    mutate: (sessionValue?: string) => {
      if (sessionValue !== undefined) {
        set(sessionKey, sessionValue)
      }
      return mutate()
    },
  }
}
```

실제 사용을 위해 일단 app.js에서 사용해보겠습니다.

```jsx
import React, {useState} from 'react';
import './App.css';
import useSession from './hook/useSession';

//create-react-app 으로 생성 본문만 지우고 useSession 사용
function App() {
  const [key, setKey] = useState('test-session-key1')
  const [value, setValue] = useState<any>()
  const { data, mutate } = useSession(key)
  return (
    <div className="App">
      <header className="App-header">
        <div>
          {key}: {data}
        </div>
        <div>
          session Key
          <input
            onChange={async e => {
              setKey(e.target.value)
              await mutate()
            }}
            defaultValue={key}
          />
        </div>
        <div>
          change session Value to
          <input
          onChange={e => setValue(e.target.value)}
          defaultValue={data}
          />
          <button onClick={() => mutate(value)}>
            submit
          </button>

        </div>
      </header>
    </div>
  );
}

export default App;
```

원하는 동작으로는 sessionKey에 따라 세션스토리지에서 데이터를 가져오고 만약에 key가 변경되지 않는다면 캐싱된 데이터를 받아오는 것입니다. key가 변경된다면 화면을 새로 랜더링 해서 화면에 보이는 값이 바뀝니다. 아래 동영상처럼 동작하면 정상입니다.

{% include video.html src="front-test-tips/useSessionExample.mp4" controls="true" caption="예시 영상" %}

이렇게 직접 개발자가 화면에서 확인해봤지만 우리는 이 SWR로 만든 hook에 유닛테스트를 추가하기로 했죠?

일단 테스트 케이스를 정의하겠습니다.

```jsx
// src/hook/useSession.test.js

describe('useSession 테스트', () => {
  test('mutate함수의 인자 값이 session storage에 저장된다.', () => {
  })
  test('인자가 같으면 같은 data가 리턴된다.', () => {
  })
  test('인자가 다르면 다른 data가 리턴된다.', () => {
  })
})
```

#### renderHook

hook의 결과를 받고 기다리는 테스트 용도로 [renderHook](https://react-hooks-testing-library.com/reference/api#renderhook) 이라는 함수가 있습니다.

renderHook은 hook만 별도로 작업할 수 있게 hook의 랜더링을 대기하는 함수와, 결과를 확인하는 객체로 이루어져있습니다. (cleanup, act 함수 포함)

renderHook은 cra로 리액트를 설치했을 때 **별도 설치가 필요**합니다. 만약에 세팅이 안 되었다면 아래 따라서 설치해주세요

```shell
# if you're using npm
npm install --save-dev @testing-library/react-hooks
# if you're using yarn
yarn add --dev @testing-library/react-hooks
```

출처: [https://react-hooks-testing-library.com/installation](https://react-hooks-testing-library.com/installation)

renderHook 안에 있는 객체중 여러개를 다 기억할 필요없이 저는 2,3가지 위주로 사용하길 추천드립니다. `renderHook().waitFor`, `rednerHook().waitForValueToChange`, `renderHook().result.current` 입니다.

hook을 대기하는 함수는 [waitFor](https://react-hooks-testing-library.com/reference/api#waitfor), [waitForNextUpdate](https://react-hooks-testing-library.com/reference/api#waitfornextupdate), [waitForValueToChange](https://react-hooks-testing-library.com/reference/api#waitforvaluetochange) 3가지 종류가 있는데 NextUpdate를 추천하지 않는 이유는 개발자가 예상한 것 보다 hook이 더 여러 번 랜더링 되는 경우가 있어서 NextUpdate 보다는 **waitFor를 사용해 원하는 결과가 나올 때 까지 기다려주는 것이 유리**합니다.

예를 들면 SWR을 쓰면 isValidating 옵션이 변경 때문에 실재로 결과가 1번에 나온거 같아도 SWR을 사용하는 hook은 2번씩 호출됩니다.

“나는 SWR을 안 쓰고 hook에 관한 테스트만 작성 할건데?” 라고 하셔도 hook이 조금 복잡하면 굳이 테스트 코드에서 NextUpdate를 여러번 반복 호출해야되는데 굳이 그럴 필요는 없겠죠?

같은 이유로 화면에 보이는 결과는 정해져있지만 `renderHook().result.all`을 사용하면 배열 안에 여러 데이터가 쌓여 있는 것을 종종 볼 수 있습니다. 그래서 항상 마지막 결과를 확인하는 `renderHook().result.current`를 추천드립니다.

아래는 `'mutate함수의 인자 값이 session storage에 저장된다.'` 에 대한 테스트 케이스 입니다. 한 번 위에서 아래대로 주석과 함께 읽으시면 어렵지 않습니다.

```jsx
import useSession from './useSession'
import {renderHook, act} from '@testing-library/react-hooks'

describe('useSession 테스트', () => {
  test('mutate함수의 인자 값이 session storage에 저장된다.', async () => {
    const hook = renderHook(() => useSession('testKey1'))
    // session storage에 세팅이 안 된 초기값은 undefined
    expect(hook.result.current.data).toBe(undefined)
    
    //값을 바꿔준 mutate 함수는 hook의 state를 바꾸는 비동기 함수이기 때문에 act를 사용한다.
    await act(() => hook.result.current.mutate('입력한 테이터1'))
    
    //const {data} = useSession('testKey1')의 data가 undefined가 아닐 때까지 대기
    await hook.waitFor(() => hook.result.current.data !== undefined) 
    
    //마지막 data가 '입력한 테이터1' 인지 확인
    expect(hook.result.current.data).toBe('입력한 테이터1')
  })
  test('인자가 같으면 같은 data가 리턴된다.', () => {
  })
  test('인자가 다르면 다른 data가 리턴된다.', () => {
  })
})
```

`값을 바꿔준 mutate 함수는 hook의 state를 바꾸는 비동기 함수이기 때문에 act를 사용한다.` 여기는 조금 궁금하실 것 같아서 더 첨언하면 hook이 아니고 컴포넌트를 테스트 할 때도 state가 변경되는 비동기 작업을 외부에서 하는 경우 act를 사용하셔야됩니다. 안 그러면 아래 같은 테스트 에러가 발생합니다.

{% include image.html img="front-test-tips/7.png" caption="결과" %}

3개의 테스트 케이스 중 1개를 만들어봤는데 이제 나머지 2개는 그냥 똑같은 패턴으로 테스트 작성하시면 됩니다.

```jsx
import useSession from './useSession'
import {renderHook, act} from '@testing-library/react-hooks'

describe('useSession 테스트', () => {
  test('mutate함수의 인자 값이 session storage에 저장된다.', async () => {
    const hook = renderHook(() => useSession('testKey1'))
    expect(hook.result.current.data).toBe(undefined)
    await act(() => hook.result.current.mutate('입력한 테이터1'))
    await hook.waitFor(() => hook.result.current.data !== undefined)
    expect(hook.result.current.data).toBe('입력한 테이터1')
  })
  test('인자가 같으면 같은 data가 리턴된다.', async () => {
    const hook1 = renderHook(() => useSession('testKey2'))
    await act(() => hook1.result.current.mutate('입력한 테이터2'))
    await hook1.waitFor(() => hook1.result.current.data !== undefined)
    expect(hook1.result.current.data).toBe('입력한 테이터2')
    const hook2 = renderHook(() => useSession('testKey2'))
    expect(hook2.result.current.data).toBe('입력한 테이터2')
  })
  test('인자가 다르면 다른 data가 리턴된다.', async () => {
    const hook1 = renderHook(() => useSession( 'testKey3'))
    await act(() => hook1.result.current.mutate('입력한 테이터3'))
    await hook1.waitFor(() => hook1.result.current.data !== undefined)
    expect(hook1.result.current.data).toBe('입력한 테이터3')
    // 키 변경
    const hook2 = renderHook(() => useSession('testKey4'))
    await hook2.waitFor(() => !hook2.result.current.isValidating)
    expect(hook2.result.current.data).not.toBe('입력한 테이터3')
  })
})
```

### 타이머가 있는 경우

프론트엔드 테스트 코드를 작성하다보면 타이머가 필요한 경우들이 있습니다.

이런 경우는 [jest.useFakeTimer](https://jestjs.io/docs/timer-mocks) 를 사용하면 쉽게 테스트할 수 있습니다. 위에 맨처음 예시에서 만든 버튼을 사용한 카운터를 만들어서 설명 드리겠습니다.

```jsx
// src/components/Counter.tsx
import Button from "./Button";

export default ({}) => {
  const [count, setCount] = useState(0)
  const countUP = () => setTimeout(() => setCount(count + 1), 1000)
  const countDown = () => setTimeout(() => setCount(count - 1), 1000)
  return <>
    <div>숫자: <span data-testid="counter-value">{count}</span></div>
    <div style={{ display: 'flex', gap: '20px'}}>
      <Button styleType="danger" onClick={countUP}>up</Button>
      <Button styleType="primary" onClick={countDown}>down</Button>
    </div>
  </>
}
```

위 코드를 사용해보면 아래 영상처럼 버튼을 눌렀을 때 천천히 숫자가 변동하는 것을 볼 수 있습니다.

{% include video.html src="front-test-tips/slowCounter.mp4" controls="true" caption="예시 영상" %}

테스트 코드를 만들어보면 아래와 같습니다.

```jsx
import Counter from './Counter'
import { render, screen, fireEvent, act} from '@testing-library/react'
describe('counter', () => {
  beforeEach(() => {
    jest.useFakeTimers()
  })
  test('버튼을 누르고 1초 뒤에 카운트가 올라간다.', () => {
    render(<Counter/>)
    fireEvent.click(screen.getByText('up'))
    expect(screen.getByTestId('counter-value').textContent).toBe('0')
    jest.advanceTimersByTime(1200) // 1초 이상 지나감
    expect(screen.getByTestId('counter-value').textContent).toBe('1')
  })
  test('버튼을 누르고 1초 뒤에 카운트가 내려간다.', () => {
  //생략
  })
})
```

`advanceTimersByTime`는 시간을 흘려보내는 함수입니다. 테스트 beforeEach 단계에서 `useFakeTimers` 를 세팅하고 실제 테스트가 필요할 때 advanceTimersByTime를 호출한 뒤에 다시 screen을 확인하면 시간을 확인할 수 있습니다.

cmd 결과가 성공으로 나왔다면 잘 따라 오셨습니다!

### Hook을 쓰는 페이지 테스트 (통합 테스트)

위에서는 계속 컴포넌트를 테스트 했었습니다. 그러면 이제는 여러 컴포넌트를 조합한 페이지 테스트를 만든 방법을 알려드리려 합니다. 유닛과 통합테스트의 개념 차이는 있지만 **실제로 테스트를 만들어보면 서로 다른점이 없습니다.**

리엑트에서 **페이지는 조금 복잡한 컴포넌트일 뿐**입니다. 그래서 컴포넌트를 테스트 할 때와 별로 다른점은 없고 똑같이 랜더링하고 스크린을 확인하는 방식으로 테스트가 가능합니다.

`useSession` 예시에 있던 App.js의 테스트 코드를 만들어보겠습니다.

```jsx
import React, {useState} from 'react';
import './App.css';
import useSession from './hook/useSession';

//create-react-app 으로 생성 본문만 지우고 useSession 사용
function App() {
  const [key, setKey] = useState('test-session-key1')
  const [value, setValue] = useState<any>()
  const { data, mutate } = useSession(key)
  // 테스트 편의를 위해 data-test-id 세팅
  return (<div className="App">
      <header className="App-header">
        <div data-test-id="session-value">
          {key}: {data}
        </div>
  //생략
  )
  
}

export default App;
```

저는 App.js에서 `useSession.data에 따른 다른 데이터가 화면에 보인다`를 테스트 하고 싶습니다.

그러면 `useSession` 에 data를 내가 원하는 값으로 설정해야되는데 `useSession.test.js`서는 직접 세션 스토리지를 건드렸었습니다. 하지만 이번에는 `useSession`을 가져다 쓰는 입장이기 때문에 스토리지를 직접 건드리기 보다는 `jest.mock`을 사용하기를 추천드립니다.

jest.mock을 활용하면 모듈을 통째로 mock할 수 있습니다.

```jsx
import React from 'react';
import { render, screen } from '@testing-library/react';
import App from './App';
import useSession from './hook/useSession'

// 해당 모듈전체를 mock, useSession.mockImplementation도 똑같음
jest.mock('./hook/useSession')

describe('통합 테스트', () => {
  test('useSession.data에 따른 다른 데이터가 화면에 보인다', () => {
    useSession.mockImplementation( () => {
      return {
        data: 'mock 데이터',
        mutate: jest.fn()
      }
    })
    render(<App/>) // 컴포넌트와 다를 것이 없다.
    //초기값 확인
    expect(screen.getByTestId('session-value').textContent).toContain('mock 데이터')
  })
})
```

예시에서는 `jest.mock`을 사용했는데 만약에 원래 로직은 유지하고 호출횟수나 어떤 인자가 들어왔는지만 확인하고 싶다면

`jest.SpyOn`을 사용하기를 추천드립니다.

## 후기: 완벽한 테스트를 위해

휴… 여기까지 부실하면서 긴 글을 읽어주신분들에게 감사의 인사를 전합니다.

글의 분량이 상당하고 제가 알고 있는 팁들을 많이 서술했는데 아쉽게도 이 정도 정보로는 테스트를 만드는데 어려움을 겪으실 수 있습니다. ㅠㅠ

이번 글이나 [jest](https://jestjs.io/docs/getting-started)가 아니라 어떤 라이브러리든 블로그 글만 읽기 보다는 **공식 사이트와 같이** 읽으면서 기본적인 API를 다 알고 계셔야 실증 위주로 써진 블로그 글이 유용한 정보를 제공할 수 있습니다.

글의 마지막으로 제 근황을 조금 공유하려고 합니다.

유닛테스트를 만드는 방법에 대해서 할 말 거의 다 했지만. 테스트는 배포 자동화와 같이 세팅이 되어야 비로소 제대로 활용이 가능합니다. 비즈레버 프론트개발 근황은 올해 1월에는 [codeCov](https://about.codecov.io/) 라는 서비스를 세팅했습니다.

PR이 올라갈 때마다 테스트 커버리지를 수집하고 그래프를 제공해주고 있습니다.

{% include image.html img="front-test-tips/8.png" caption="디자인 리뉴얼 작업하면서 테스트가 퍼센트 커버리지가 겨우 18% ㅠㅠ 이걸 올리는 것이 앞으로의 과제" %}

저도 그러고 있고 매드업 개발자들은 시간이 있을 때 마다 코드의 품질을 보증하기 위해 여러가지 방법들을 고민하고 저연차 개발자들도 직접 여러가지를 해볼 수 있습니다.

제가 매드업에 입사하기 이전에는 “광고 회사에서 개발자가 할 일이 있나?“ 라고 생각했는데 막상 광고회사에 와보니 일반적인 소프트웨어 서비스를 주력으로 만드는 회사들 만큼 어쩌면 더 할 일이 많습니다.

예를 들면 페이스북, 구글, 카카오의 API를 연동하고 방대한 광고 데이터를 정제하고 그걸 시각화해서 고객에게 제공하고 광고 관리 툴을 AE들에게 제공한다… 이런 작업을 하다 보니 서비스 규모에 비해서 정말 할 일이 많습니다. 매드업에 개발자가 많기 때문에 저걸 다 혼자서 하지는 않지만 그래도 할 일이 많은 만큼 코드 품질, 리뷰가 더 중요합니다.

그런 배경 때문에 앞으로 더 테스트를 많이 만들고 코드 품질을 올릴 여러 가지 방법을 고민해야겠다고 생각하고 있습니다.

다시 한 번 긴 글 읽어주셔서 감사의 인사를 전합니다
