---
title: React Query vs SWR
excerpt: React Query와 SWR의 차이점
image: react-query-vs-swr/thumb.jpg
categories: [tech]
author: chelsea
---

리액트 개발자라면 누구나 Redux, MobX 등의 상태 관리 라이브러리를 사용해 본 경험이 있을 것이다. 이 라이브러리들은 클라이언트 상태 관리에는 유용하지만, 서버 상태와 동기화되지 않기 때문에 프론트엔드 개발자들은 직접 상태를 업데이트해 줘야 하는 불편함이 있다.

이러한 불편함을 해소하기 위해 등장한 새로운 방식의 라이브러리가 바로 React Query와 SWR이다. 우리 회사의 프로덕트 중 하나인 ‘[레버](http://biz.lever.me/)’에서는 상태 관리를 위해 SWR을 사용하고 있는데, 개인적으로 공부하던 중 요즘 트렌드는 React Query라는 것을 알게 되었다.

{% include image.html img="react-query-vs-swr/graph.png" caption="React Query와 SWR의 npm 다운로드 수 비교" %}

2021년 1월까지만 해도 SWR이 미세한 차이로 앞서나가고 있었으나, 현재는 React Query의 다운로드 수가 우세한 상황이다. 같은 동기를 가지고 개발된 두 라이브러리는 어떤 차이가 있을까?

## Overview

### **React Query**

> React Query는 리액트 애플리케이션에 서버 상태를 가져오고, 캐싱하고, 동기화하고, 업데이트하는 것을 쉽게 해 준다.
> 

### **SWR**

> SWR은 먼저 캐시에서 데이터를 반환한 다음, 서버에 데이터를 가져오는 요청을 보내고, 마지막으로 최신 데이터를 제공하는 전략이다.
> 

SWR과 React Query 공식 문서에 나와 있는 짤막한 소개다. 둘의 개념 자체는 비슷해 보인다. 둘을 직접 사용해 보며 차이점을 알아보자. *(SWR v1.1.1, React Query v3.34.2 기준)*

## Basic Usage

아래 예제를 통해 기본적인 사용법을 비교해 보자.

### **SWR**

```jsx
import useSWR from "swr";

const App = () => (
  <div>
    <SWRProfile />
  </div>
);

const SWRProfile = () => {
  const {data, error} = useSWR("https://61b88c9d64e4a10017d19053.mockapi.io/user", url =>
    fetch(url).then(res => res.json())
  );

  if (error) return <div>failed to load</div>;
  if (!data) return <div>loading...</div>;

  return <Profile library="SWR" data={data} />;
}

const Profile = ({library, data}) => (
  <div>
    <h1>Users from {library}</h1>
    {data.map(user => <p>{user.level} developer <strong>{user.name}</strong></p>)}
  </div>
)

export default App;
```

### **React Query**

```jsx
import { QueryClient, QueryClientProvider, useQuery } from "react-query";

const queryClient = new QueryClient();
const url = "https://61b88c9d64e4a10017d19053.mockapi.io/user";

const App = () => (
  <div>
    <QueryClientProvider client={queryClient}>
      <ReactQueryProfile />
    </QueryClientProvider>
  </div>
);

const ReactQueryProfile = () => {
  const {isLoading, error, data, isFetching} = useQuery("users", () =>
    fetch("https://61b88c9d64e4a10017d19053.mockapi.io/user").then(res => res.json())
  );

  if (error) return <div>failed to load</div>;
  if (isLoading) return <div>loading...</div>;

  return <Profile library="React Query" data={data} />;
}

const Profile = ({library, data}) => (
  <div>
    <h1>Users from {library}</h1>
    {data.map(user => <p>{user.level} developer <strong>{user.name}</strong></p>)}
  </div>
)

export default App;
```

### **Provider**

SWR은 별도의 Provider 없이 컴포넌트에서 바로 사용할 수 있지만, React Query는 Provider로 컴포넌트를 감싸지 않을 경우 에러가 발생한다.

### **Fetcher**

`useSWR`, `useQuery` 모두 두 번째 인자로 fetcher를 받는다. 차이점이 있다면 SWR은 fetcher의 인자로 `useSWR`의 첫 번째 인자를 넘겨 주고, `useQuery`는 fetcher에 url을 직접 전달해야 한다는 점이다. 또한 SWR은 전역 설정을 통해 fetcher를 정해 둘 수 있지만, React Query는 항상 두 번째 인자에 fetcher를 넘겨 줘야 한다.

### **Status**

SWR은 `isValidating`을 이용해 상태를 표현하는 데 반해, React Query는 `isLoading`, `isFetching`을 통해 데이터의 상태를 보여 준다. 특히 `isFetching`은 첫 번째 로드를 제외한 데이터 업데이트 시의 상태를 나타내는 값으로 모든 데이터 로드 상태를 나타내는 `isValidating`과 구별된다.

## Mutation

SWR과 React Query는 모두 뮤테이션이라는 개념을 가지고 있다. 하지만 두 라이브러리에게 있어 이 개념은 다르게 작용한다. 둘 다 변형시킨다는 의미에서는 같지만, React Query의 뮤테이션은 post/patch/put/delete를 통해 서버의 상태를 변형시키는 것이고, SWR의 뮤테이션은 `useSWR()`을 통해 받아온 데이터를 클라이언트 사이드에서 변형시켜 업데이트해 주는 개념이다.

## Devtools

React Query는 `react-query/devtools`를 통해 devtools를 제공한다. 사용 방법은 아래와 같다.

```jsx
import { ReactQueryDevtools } from 'react-query/devtools'

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  )
}
```

`process.env.NODE_ENV`가 `development`일 때만 보이는 컴포넌트이므로 걱정할 것은 없다. `ReactQueryDevtools`는 왼쪽 하단에 토글 버튼이 위치하는 방식이며, embed 방식으로 사용하고 싶다면 `ReactQueryDevtoolsPanel`을 사용하면 된다.

SWR 또한 devtools를 사용할 수 있으나, 공식적으로 제공되고 있지는 않아 서드 파티 라이브러리를 사용해야 한다.

## Bi-directional Infinite Queries

페이지네이션과 무한 스크롤 같은 UI를 사용할 때 SWR과 React Query 모두 도움이 된다. 하지만 SWR은 이전 페이지의 데이터를 불러 오려면 유저가 부가적인 코드를 작성해야 한다. 그에 반해 React Query는 `getPreviousPageParam`, `fetchPreviousPage`, `hasPreviousPage`, `isFetchingPreviousPage` 프로퍼티들을 통해 이전 페이지 데이터를 쉽게 핸들링할 수 있다.

## Lagged Query Data

React Query는 다음 데이터를 불러오기까지 현재 데이터를 표시해 준다. 예를 들면 페이지네이션에서 다음 페이지를 불러오는 동안 보여 줄 데이터가 없을 때 현재 캐싱되어 있는 데이터를 자동으로 렌더링하는 것이다. 이는 페이지네이션에서 꽤 유용하게 사용된다. SWR이 기본적으로 지원하고 있는 기능은 아니지만 부가적인 코드 작성을 통해 구현 가능하다.

## Offline Mutation

위에서 말했듯 SWR과 React Query의 뮤테이션은 다르게 작동한다. React Query에서는 오프라인 상태에서 뮤테이션을 시도했을 때 해당 요청을 잠시 멈췄다가 온라인 상태가 되면 요청을 재시도한다. 이러한 API 요청을 멈췄다가 다시 시도해 서버의 데이터를 변경하는 것은 SWR에는 없는 기능이다.

## Selectors

```jsx
import { useQuery } from 'react-query'

function User() {
  const { data } = useQuery('user', fetchUser, {
    select: user => user.username,
  })
  return <div>Username: {data}</div>
}
```

위의 코드는 React Query에서 selector를 이용해 쿼리 결과의 부분을 추출하는 코드다. React Query에만 있고 SWR에는 없는 기능이다.

## Data Optimization

React Query는 렌더링 퍼포먼스 측면에서도 뛰어난 라이브러리다. 쿼리가 업데이트될 때만 컴포넌트를 업데이트한다. 또한 여러 컴포넌트가 같은 쿼리를 사용하고 있을 때는 한꺼번에 묶어 업데이트해 준다. 이 항목은 SWR에는 해당되지 않는다.

## Auto Garbage Collection

React Query는 쿼리가 지정된 시간(설정하지 않을 경우 기본 5분) 동안 쿼리가 사용되지 않을 경우 자동으로 Garbage Collection을 지원한다. 이 또한 SWR에는 해당되지 않는 항목이다.

## Prefetching APIs

React Query는 `prefetchQuery`를 통해 데이터 프리패칭을 다룰 수 있다. SWR에서 불가능한 부분은 아니며, 다른 방식으로 코드를 작성해 같은 효과를 볼 수 있다. React Query에서는 별도의 API를 지원한다는 부분이 차이점이다.

## Query Cancellation

기본적으로 프로미스가 처리되기 전에 마운트가 해제되거나 사용되지 않는 쿼리는 취소되지 않는다. 그러나 `AbortSignal`을 사용하거나 프로미스에 `cancel` 함수를 적용하면 프로미스가 취소됨에 따라 쿼리도 함께 취소된다. 쿼리가 취소되면 해당 상태가 이전으로 돌아간다. 이 기능은 React Query에서만 지원되고 있다.

이외에도 React Query에서만 지원하는 많은 기능들이 더 있다. 자세히 알아보고 싶다면 [링크](https://react-query.tanstack.com/comparison)를 눌러 확인해 보자.

## 결론

React Query를 공부하기 전, SWR과의 차이점을 알아보기 위해 이것저것 검색했지만 마땅한 자료를 찾기 힘들었다. 당시 참고했던 글 중 하나는 2020년 1월에 작성된 글이었는데, 그때까지만 해도 React Query의 버전이 상당히 낮아 SWR과 비교 자체가 안 되는 수준이었다. 하지만 React Query는 점차 발전해 왔고, 현재는 프론트엔드 개발자들의 높은 선호도뿐만 아니라 지원하는 기능들의 수준도 탁월하다. 개인적으로 렌더링 퍼포먼스를 높이기 위해 데이터를 최적화한 점과 자동으로 가비지 컬렉션을 지원한다는 점에서 크게 감명받았다. 이제는 React Query의 시대를 기대해 봐도 괜찮지 않을까?

## Reference

- [React Query](https://react-query.tanstack.com/)
- [React Hooks for Data Fetching – SWR](https://swr.vercel.app/)
