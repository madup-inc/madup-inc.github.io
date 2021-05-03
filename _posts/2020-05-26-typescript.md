---
title: Typescript의 단점
excerpt: 아무도 말해주지 않는 Typescript의 각오해야 할 점
image: typescript/typescript.jpg
categories: [tech]
author: willy
---

Typescript를 사용한지 1달도 안 됐지만, 사용이 불편하거나 오히려 고민거리를 만드는 단점이 있어 포스팅합니다.  

장점이야 2012년 이후 너무 많은 사람이 이야기해서 1달도 안 써본 제가 적는 게 의미가 없다고 느꼈습니다.  

인터넷에 검색해봐도 두려워 말고 타입스크립트 쓰라는 트랜드세터들의 글이 수천 개가 넘습니다.  

그런데 왜 Typescript를 도입할 필요가 없는지, 혹은 어떤점에 대해서는 기대하지 말고 단점을 각오하고 사용해야 실망을 덜 하는지 설명해주는 글이 없더군요.  

목차는 아래와 같습니다.  

1. 혼자해서 급한데 타자 수만 늘어난다.
2. 동네북 Any
3. 필요한건 하나인데 어쩌다 보니 모든걸 정의했다.
4. 리액트에서 클래스 쓸 일이 없는데 왜 객체지향이 필요하죠?
5. 장점: 고민할 필요 없지만 다른 블로그에서 언급하는 단점의 반론  

바쁘신 분들을 위해 단점 2가지만 요약하자면 "타자수가 많아지고, 굳이 객체지향을 쓸 필요가 없다" 입니다.  

 > 주의 : 까는 글만 썼지만 저는 타입스크립트 도입에 찬성했고 아직도 타입스크립트를 밀고 있습니다.

---

{% include image.html img="typescript/1.jpg" %}

## 혼자해서 급한데 타자 수만 늘어난다
   > 나는 외롭다... 혼자 개발한다... 코드 리뷰를 언제 해봤는지 모르겠다... 잘 하는 것 보다 빨리 개발이나 하자...


저런 상황이면 props를 미리 정의하거나 인터페이스를 미리 만드는 것이 의미가 없을 수도 있습니다.  

나중에 시간이 흘러 유지보수 할 때 꼼꼼하게 제한한 타입들이 숫자를 문자열로 받거나 컨테이너에서 컴포넌트로 잘못된 props를 넘겨주는 일을 막아줄 수도 있겠지만, 애초에 혼자 개발하는데 많은 코드를 짜는 프로젝트가 아니면 그런 실수 할 일도 적고 괜히 타자수만 늘려서 시간 낭비 할 수도 있겠습니다.  

```typescript
// Typescript  
interface LayoutProps {
   hasSider: Boolean,
   hasHeader: Boolean,
   //etc
}

function Layout(props: LayoutProps): React.Fc {
   //todo
}

//Javascript
const Layout = ({hasSider, hasHeader, ...etc}) => {
   //todo 별다줄
}
```  

물론 타입스크립트도 밑에처럼 짧게 사용하실 수 있지만 그러면 타입스크립트를 사용하는 이유가 좀 퇴색될 것 같습니다.  

```typescript
function Layout({...props}) {
   //todo
}
```  

## 동네북 Any
   > 아무거나 아무거나 하다가 민트초코가 왔다.
   
   
자바스크립트에서 다용도로 쓰는 취급주의 타입은 고작 2개입니다. `Array` 아니면 `Object`이죠. 그래서 새로운 걸 하고 싶으면 저 둘 중에서 하나 골라서 사용하면 됩니다.  

Optional하게 값을 감싸서 쓰고 싶으면 `Array`를 쓰고, KeyValue로 뭔가 저장하고 싶으면 `Object`를 쓰면 됩니다.  

그런데 타입스크립트에서는 별걸 다 직접 만들어 쓰다 보니 `Object`나 뭔지 모르는 걸 쓸 상황에는 `any`를 사용합니다.  

타입스크립트스럽게 쓸 노력이 없으면 나중에 모든 타입이 any로 떡칠돼서, 타입스크립트를 쓰나 마나 아무런 타입 에러도 잡아주지 않을 게 뻔해 보였습니다.  

다 같이 음식 시킬 때 아무거나 달라고 하면 정말 아무거나 줍니다.  

타입스크립트도 정말 아무 데이터나 인자로 넘어갈 수 있습니다.  

이렇게 사용하면 타입스크립트를 쓰는 게 너무 아깝습니다. 귀찮다고 넘기지 말고 2번째 함수처럼 명확하게 인자의 타입을 써주셔야합니다.  

너무 당연한 것 같아도 막상 하다보면 안 지킵니다. 아무거나 달라고 하면 민트초코 먹습니다.  

```typescript 
function number2CommaStringBad(value: any, suffix: any) { // bad
   return value.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",") + suffix 
}

function number2CommaStringGood(value: number | string, suffix: string = '') { // good
  return (typeof value === 'string' ? value : value.toString()).replace(/\B(?=(\d{3})+(?!\d))/g, ",") + suffix
}

const mint = {1: '23000'}

number2CommaStringBad(mint, '원') // [object Object]원
number2CommaStringGood(mint, '원') // syntex error
number2CommaStringGood(100000, '원') // 100,000원
```  

## 필요한건 하나인데 어쩌다 보니 모든걸 정의했다
   > 사막에서 바늘을 찾으려고 사막 전체를 사버렸다. flex
   
   
아래는 그냥 상황 설정입니다. 재미로 봐 주세요;;  

배철강씨는 완벽주의자라 개발 시작할 때 가장 먼저 모든 모델과 클래스가 정의되어야 안심하는 사람입니다. 자 그러면 배철강씨를 위해 Advertise 타입을 미리 만들어 드릴까요?  

데이터 관점에서 보면 `is-a`는 없고 `has-a`가 대부분이 맞다고 생각합니다.  

그러면 이제 데이터베이스와 비슷한 모양새로 만들어 봅시다.  

```typescript
interface Writer {
   createdBy: string,
   updatedBy: string,
}

interface Informaiton {
   email: string,
   managerName: string,
   phone: string
}

interface Manager {
   name: string,
   token: string,
   information: ManagerInformation
} 

interface ManagerInformation {
   signupPath: string,
   infomation: Informaiton
   ...
}

interface Advertiser {
   infomation: Informaiton
   ...
}

interface Capacitie {
   totalCapacite?: number,
   dailyCapacite?: number,
}

interface Publisher {
   cap?: Capacites,
   publisher: string,
   createdBy: string,
   updatedBy: string,
   infomation: Informaiton
}

interface Advertise { // 이거 하나 필요한데요...
   payout: number,
   capatites: Capacitie
   advertisers: Advertiser
   publishers: Publisher[]
   manager: Manager
   writer: Writer
}
```  

다 한 것 같기는 한데, 제가 원한 건 `Advertise` 하나인데 한 번 쓰고 다시는 안 쓸 `Capacities`, `Advertiser`, `Publisher`를 전부 정의해 버렸네요.  

별 생각 없이 모든 인터페이스를 만들다보면 시간 아깝게 이런 상황이 벌어집니다... 바늘 하나 찾겠다고 사막을 산 셈이네요.  

## 리액트에서 클래스 쓸 일이 없는데 왜 객체지향이 필요하죠?
   > 그래! 함수형, 리액티브, 훅 시류에 따라야겠다. 그런데 그 유행에 클래스는 필요 없는 것 같아.
   
   
요즘 리액트 유행에서는 대부분 새로 만드는 컴포넌트는 함수형 컴포넌트로 만들고 이전에 프레젠테이션 컴포넌트(클래스로 만든 컴포넌트)의 역할을 `Hook Api`를 사용해서 만듭니다.  

`Hook`는 2019년 2월에 공식 [릴리즈](https://github.com/facebook/react/releases/tag/v16.8.0){:target="_blank"} 됐는데 1년 조금 넘는 시간 동안 각종 Hook을 지원하는 라이브러리가 등장했습니다.  

시대는 바야흐로 함수형 컴포넌트, Hook의 시대입니다. 이런 상황에서 클래스로 뭘 할 필요성도 못 느끼고 라이브러리를 만들지 않는 한 다형성 구현을 할 필요성을 못 느꼈습니다.  

위에서 살짝 언급했듯이 모델 만드는 데는 `is-a` 대신 `has-a`면 충분하기 때문에 `extends`나 `impliments`를 사용할 필요가 없었습니다.  

`ES6` 이후 추가된 `map`, `flatmap`, `filter`, `reduce` 같은 고차함수, 리액트의 함수형 컴포넌트들이 자바스크립트와 라이브러리가 함수형으로 가는 방향을 보여주는데 간단한 걸 만드는데, 굳이 클래스가 필요한지 의문이 듭니다.  

타입스크립트 [클린코드 가이드](https://github.com/738/clean-code-typescript){:target="_blank"}의 객체지향 관련된 대부분의 내용이 중요하긴 해도 막상 간단한 웹 만들 때는 필요가 없던거죠.  

{% include image.html img="typescript/3.jpg" %}

## 장점: 고민할 필요 없지만 다른 블로그에서 언급하는 단점의 반론

다른 개발자들에게 킬링타입용 글 하나 쓰기 위해 억지억지로 단점을 찾던 중 이건 별로 합리적이지 않다 싶어 제외한 단점들이 많습니다.
그런 단점의 반론으로 생각해 본 내용입니다.  

> 초보 개발자가 많은데 새로운 언어를 배우는 진입장벽 때문에 프로젝트에 악영향이 갈 거다.
 
 
저희가 학생들도 아니고 주니어라도 직업으로 하는 사람인데 타입스크립트 도입 하나 때문에 프로젝트가 망할 거라고 생각하지는 않습니다.  

오히려 잘하는 사람 1명만 있고 나머지 3명의 초보자가 프로젝트를 한다고 가정해봅시다.  

*잘하는 1명이 만든 인터페이스와 제한들*로 인해 나머지 3명이 오히려 타입 관련된 초보적인 오류나 클래스를 잘 못 가져다 쓰는 실수는 피할 것 같습니다.  
> 도입 비용이 너무 많이 들어서


완전히 새로운 프로젝트가 아니라 기존 프로젝트에 도입하는게 고민되시면 전체 코드를 `.ts, .tsx`로 바꾸지 마시고 새로 만드는 부분만 타입스크립트로 개발하세요.  

자바스크립트와 타입스크립트가 호환이 안 되는 것은 아닙니다.  

> 생태계가 과연 괜찮나?


이것도 별 고민할게 없는게 현재 대다수의 라이브러리가 타입스크립트로 만들어지고 타입스크립트를 지원하는 추세입니다.  

타입스크립트는 2019년 스택오버플로우 프로그래밍, 스크립트, 마크업 사용 조사에서 10위를 차지했는데 대략 20퍼센트의 프로그래머가 사용할 수 있습니다. ([링크](https://insights.stackoverflow.com/survey/2019#most-popular-technologies){:target="_blank"})  

같은 조사에서 선호도는 73.1퍼센트로 3위를 차지했습니다. ([링크](https://insights.stackoverflow.com/survey/2019#technology-_-most-loved-dreaded-and-wanted-languages){:target="_blank"})  

사용도 많이하고 좋아하는 사람도 많은 언어입니다.  

> 어차피 평소에 타입 체크 잘 써서 타입스크립트가 필요가 있는지 모르겠네요.


어차피 할거 인자 구문에서 더 쉽게 합시다. 제네릭 하나만 잘 써도 코드가 매력이 넘칩니다.  
 
밑에는 제가 좋아하는 함수인데, 오브젝트 배열 하나를 받고 함수에 따라 [_.groupBy()](https://lodash.com/docs/4.17.15#groupBy){:target="_blank"} 해주는 함수를 만드는 예제입니다.  

```javascript
//javascript
function groupBy(items, func) {
   if (!Array.isArray(items) || typeof func === 'function') throw error() // 이러고도 못잡을 가능성 있음
   const results = {}
   items.forEach((item) => results[func(item)] = (results[func(item)] || []).concat(item))
   
   return results
}
```  

자바스크립트에서는 인자가 배열과 함수인지만 확인하기 위해 코드 안에 if문과 그 조건에 떡칠하기보다는, 그냥 인자 뒤에 `:`로 타입을 명시하거나 제네릭으로 사용하는 사람이 타입을 정하게 해서 에러를 막는 게 더 편합니다.  

```typescript
function groupBy<T>(items: T[], func: (item: T) => any) { // 타입 확인 끝, 엉뚱한 타입의 값을 인자로 주면 syntex error
   const results: any = {}
   items.forEach((item: T) => results[func(item)] = (results[func(item)] || []).concat(item))

   return results
}

interface Something {
   item: string,
   paly: boolean
}

const items: Something[] = [{
   item: 'hello wolrd',
   paly: false
}, {
   item: 'hello',
   paly: false
}, {
   item: 'wolrd',
   paly: true
}]


groupBy<Something>(items, (item) => item.paly) // {true: [...], false: [...]}
groupBy<Something>(items, (item) => item.item) // {'hello wolrd': [...], 'hello': [...], 'world': [...]}
```  

## 후기

이상 굳이 찾아본 타입스크립트 단점과 장점입니다.  

글을 쓰다 보니 장점을 4개나 쓰긴 했지만 장점도, 단점도 모두 다른 사람들이 단점이라 말하는 부분들에 더 집중해서 썼습니다.  

예시로 쓴 간단한 코드들 중에 클래스를 만드는 코드가 없는데 클래스는 딱히 쓸 일이 없었습니다. 그래서 뭔가 단점을 찾기가 어려웠습니다.  

혹시 나중에 기회가 있다면 타입스크립트를 더 사용해보고 이 글의 다음 편으로 타입스크립트 잘 쓰는 법 같은 글을 써보고 싶습니다.

---

[매드업 채용 바로가기](https://www.notion.so/maduphr/fff8c23e3b434fb1abdfb36ad915d3ee){:target="_blank"}  
