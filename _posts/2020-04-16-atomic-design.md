---
title: 리액트와 아토믹 디자인 패턴
excerpt: 아토믹 디자인 시스템을 통한 내부 적용사례입니다.
image: thumbnail/atomic.jpg
categories: [tech]
author: dajyu
---

## 아토믹 디자인이란?
아토믹 디자인은 쉽게 말하면 디자인 요소들을 나누어 파악하고 이 요소들이 조합되는 과정을 통해서 디자인을 구성하는 방식을 의미한다. (즉, 컴포넌트 중심의 디자인 패턴인 것)

- Atoms: 하나의 구성 요소. 본인 자체의 스타일만 가지고 있으며 다른 곳에 영향을 미치는 스타일은 적용되지 않아야 함.
- Molecules: 원자들의 모음
- Organisms: 분자들의 모음
- Templates: 유기체들을 모아 템플릿으로 생성
- Pages: 실제 페이지를 구성

{% include image.html img="atomic-design/1.png" %}

## 왜 아토믹 디자인인가?

리액트는 컴포넌트를 중심으로 만들어지는 프레임워크로, 컴포넌트의 재사용성이 매우 중요하다. 중복이 되는 컴포넌트를 공통으로 사용할 수 있도록 빼는 것이 관건인데, 이것은 개발을 진행하다보면 일일히 생각하며 진행하는 것이 어렵다. 결국에는 중복되는 코드가 많아지고, 이를 관리하는 것이 어려워지게 되는 것이다

그렇기에 개발 단계 처음부터 재사용성이 용이하며, 여러 개의 컴포넌트들을 모아 또 다른 컴포넌트를 만들 수 있도록(조립) 고민하며 개발하는 것이 중요해졌다. 컴포넌트 중심의 아토믹 디자인 패턴은 이러한 리액트의 특성과 잘 맞아 떨어졌다.

어느 프로젝트에서든 버튼과 폼(Input) 등은 있기 마련인데, 이에 대한 디자인만 바뀔 뿐이라면 애초에 모든 프로젝트에서 사용할 수 있는 버튼과 폼 컴포넌트를 만들어 놓으면 효율성이 높아질 것이다.

#### 그래서 다른 부트스트랩, Ant, Material UI랑 다른게 뭔데?
(개인적으로) 다른 점이라면 실제로 우리가 사용할 컴포넌트들만 생성하여 위의 라이브러리들에 비해 가벼울 수 있다는 점과, 회사의 디자인적인 요소(회사 색깔, 테마 등)를 담아서 컴포넌트들을 개발하게 되면 전사 프로젝트에서 사용할 수 있다는 점이 있을 수 있다.

## 실제후기 
현재 회사에서 개발중인 React 프로젝트에서 아토믹 디자인 패턴을 적용해 봤는데, 처음 단계인 atom만 잘 만들어 두면 다음 작업이 순조롭다는 것을 몸소 체험 중이다. 굳이 페이지마다 스타일을 줘서 커스텀 할 필요도 없고(물론 디테일이 필요할 수도 있다), 그 덕분에 더 이상 className을 신경쓰며 개발할 필요가 없다는 점 또한 아주 마음에 들었다.  

{% include image.html img="atomic-design/2.png" %}


실제로 아토믹 패턴을 적용해보면서 molecule에 대한 관점 또한 달라지게 되었다. atom부터 organism 단계까지는 무조건 재사용성이 가능해야 한다는 강박관념(?)이 있었는데, 알고보니 atom 만 재사용성이 강조되며 그 이후의 단계는 재사용성이 가능해질 수도, 또는 한 페이지에서만 사용될 수도 있다는 것이다. 하지만 재사용이 될 수도 있기 때문에 네이밍이 굉장히 중요하다. (회원가입 페이지에서만 쓰일 줄 알고 SignUpFormWithEmailAndPassword라고 만들었던 것이 있는데, 로그인 화면에도 똑같이 쓰여 FormWithEmailAndPassword로 이름을 바꾸게 되었다..)


## 결론 
컴포넌트화, 재사용성과 통일된 스타일, 블록처럼 조합하면 되는 간편함 등 아토믹 디자인 패턴을 리액트 컴포넌트 개발에 도입하게 되면 효율성이 극대화 될 수 있다. 아토믹 디자인 패턴을 적용한다 이는 곧 가장 리액트다운 개발이 가능하게 된다.


#### 적용예제
  
```jsx
// Button 컴포넌트
// src/components/atoms/button/index.js

const Button = ({ type = 'button', children = '' }) => (
  <button type={type}>{children}</button>
)
```
  
```jsx
// Input 컴포넌트
// src/components/atoms/input/index.js

const Input = ({ type = 'text', id = '' }) => (
  <input type={type} id={id} />
)
```
  
```jsx
// Label 컴포넌트
// src/components/atoms/label/index.js

const Label = ({ for = '', children = '' }) => (
  <label htmlFor={for}>{children}</label>
)
```
  
```jsx
// Search Form
// src/components/molecules/searchForm/index.js

import { Button, Input } from 'atoms'

const SearchForm = () => (
  <div>
    <Input />
    <Button type={submit}>Search</Button>
  </div>
)
```
  
```jsx
// src/components/molecules/formLabel/index.js

import { Label, Input } from 'atoms'

const FormLabel = () => (
  <div>
    <Label for="form_test">테스트</Label>
    <Input id="form_test" />
  </div>
)
```


**참고자료**  
- https://patternlab.io/    
- https://bradfrost.com/blog/post/atomic-web-design/ (원문)  
- https://brunch.co.kr/@ultra0034/63 (위 원문의 한글 번역)  
- https://www.smoh.kr/276  
- https://medium.com/@inthewalter/atomic-design-for-react-514660f93ba  
