---
layout: post
title: "파이썬 세 개의 점, Ellipsis 객체는 무엇인가요?"
author: Jason
image: assets/images/thumbnail/python-ellipsis.jpg
featured: true
excerpt: 파이썬에서 사용되는 Ellipsis 객체의 여러가지 용법에 관한 내용을 번역하였습니다.
---

> 아래 글은 [출처](https://www.pakstech.com/blog/python-ellipsis/)의 글을 저자의 허락하에 번역한 내용입니다.

파이썬에서 Ellipsis 객체(…)를 우연히 발견하고 어떻게 사용되는지 궁금했습니다. 원래 [Numeric Python](https://numpy.org/) 패키지에서 매트릭스 슬라이싱을 위해 사용하도록 소개되었지만 다른 목적으로 사용하지 못하는 것은 아닙니다.

Ellipsis 객체의 원래 용법은 다차원 데이터 배열을 보다 쉽게 처리 할 수 있도록 하는 것이었습니다. 또한 FastAPI라는 비교적 새로운 웹 프레임워크에 적용된 용법도 있습니다.

# Numpy indexing
간단한 사례부터 시작하겠습니다. 콜론(:) 문자로 목록을 슬라이스하는데 사용 된다는 것은 이미 알고 있을 것입니다(`start:step:end`). 단독으로 사용하면 기본적으로 원래 목록이 반환됩니다.

```python
>>> a = [1, 2, 3, 4, 5, 6]
>>> a[:]
[1, 2, 3, 4, 5, 6]
```

다차원 파이썬 배열에도 같은 방법으로 적용할 수 있습니다.

```python
>>> b = [[1, 2, 3], [4, 5, 6]]
>>> b[:][:]
[[1, 2, 3], [4, 5, 6]]
>>> b[1][:]
[4, 5, 6]
```

여기서는 일반적인 파이썬 리스트를 사용하므로 각 차원을 별도로 인덱싱해야합니다. numpy 배열을 만들어 봅시다.

```python
>>> import numpy as np
>>> c = np.array(b)
>>> c
array([[1, 2, 3],
       [4, 5, 6]])
```

numpy 배열을 사용하면 각 차원을 고유한 괄호 안에 넣을 필요없이 쉼표로 구분할 수 있습니다. 두 차원을 모두 선택하려면 다음과 같습니다.

```python
>>> c[:,:]
array([[1, 2, 3],
       [4, 5, 6]])
```

결과는 원래 배열과 동일합니다. 이제 Ellipsis 객체를 사용해봅시다.

```python
>>> c[...]
array([[1, 2, 3],
       [4, 5, 6]])
```

결과는 여전히 동일합니다. 여기서는 Ellipsis 객체로 모든 차원을 선택했습니다.

3차원 배열을 만들어 조금 복잡하게 해봅시다.

```python
>>> d = np.array([[[i + 2 * j + 8 * k for i in range(3)] for j in range(3)] for k in range(3)])
>>> d
array([[[ 0,  1,  2],
        [ 2,  3,  4],
        [ 4,  5,  6]],

       [[ 8,  9, 10],
        [10, 11, 12],
        [12, 13, 14]],

       [[16, 17, 18],
        [18, 19, 20],
        [20, 21, 22]]])
```

테스트 데이터 배열을 만들기 위해 리스트 표현식을 사용했습니다. 첫 번째 차원의 두 번째 인덱스의 데이터를 선택해봅시다.

```python
>>> d[1,...]
array([[ 8,  9, 10],
       [10, 11, 12],
       [12, 13, 14]])
```

보시다시피 결과는 원래 배열의 두 번째 요소와 동일한 매트릭스입니다.

마지막 배열을 선택하기 위해 Ellipsis를 사용하여 나머지의 자동 완성으로 사용할 수도 있습니다.

```python
>>> d[...,1]
array([[ 1,  3,  5],
       [ 9, 11, 13],
       [17, 19, 21]])
```

Ellipsis는 배열 사이에 위치 할 수도 있습니다. 이 경우에는 콜론(:)을 사용하는 것과 동일합니다.

```python
>>> d[1,...,1]
array([ 9, 11, 13])
```

그러나 여러 Ellipsis를 사용할 수는 없습니다.

```python
>>> d[...,1,...]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: an index can only have a single ellipsis ('...')
```

이 경우는 콜론(:)을 사용해야 합니다.

# 다른 용도
Ellipsis 객체는 실제로 어떤 동작도 수행하지 않으므로 `작동 없음`을 기술하는데 사용할 수도 있습니다. 따라서 다음처럼 스텁 함수로 유효합니다.

```python
def do_something():
    ...
```

이것은 pass처럼 무언가를 나중에 구현해야할 때 사용하는 것과 비슷하며, 코드에서는 어떤 에러도 발생되지 않습니다.

```python
def do_something():
    pass
```

만약 오류가 발생되기를 바란다면 다음과 같이 NotImplementedError을 사용할 수 있습니다.

```python
def do_something():
    raise NotImplementedError()
```

FastAPI 프레임워크로 개발할 때 Ellipsis 객체를 사용하는 완전히 다른 방법을 발견했습니다(저는 그 방법을 정말 좋아합니다).

FastAPI를 사용하면 Python 3.6에 새로 도입된 Type Annotation으로 query 및 path 파라미터를 정의할 수 있습니다. 예를 들어 다음 코드 조각은 쿼리 파라미터 q를 사용하는 엔드포인트를 정의합니다.

```python
@app.get("/something/)
async def something(q: str):
    return f"Your query was {q}"
```

이 함수는 쿼리 파라미터 없이 엔드포인트가 호출되는 경우 오류 메시지가 리턴됩니다. q의 기본값으로 None을 지정하면 선택적으로 사용할 수 있습니다.

```python
async def something(q: str = None):
```

FastAPI는 파라미터 타입을 자동으로 확인하므로 값을 str 대신 int로 사용하는 경우 정수로 변환합니다. 유효성 검사를 추가하기위해 Query 객체를 사용하여 쿼리 파라미터를 명시적으로 정의할 수 있습니다. 아래 예제에서 q의 최소 길이를 10으로 설정하였습니다.

```python
async def something(q: str = Query(None, min_length=10)):
```

Query 객체의 첫 번째 매개 변수는 q의 기본값이며, 위의 경우는 None으로 설정하였습니다. q를 생략하고 호출할 수는 있지만, q에 값을 설정한 경우 길이를 10자 이상으로하고 호출해야합니다.

그러면 q를 필수 항목으로 설정하고 최소 길이도 설정해야하는 경우는 어떻게해야할까요?

여기서 Ellipsis 객체를 사용해야합니다. Query 객체의 첫 번째 매개 변수(q의 기본값)를 Ellipsis로 설정하면 FastAPI에서 필수 항목으로 설정됩니다. 따라서 메소드 정의는 다음과 같습니다.

```python
async def something(q: str = Query(..., min_length=10)):
```

FastAPI 프레임워크를 처음 접하는 사람에게는 명확하진 않지만, 매우 영리한 해법입니다. None 값은 기본값이 없는 경우에 사용되므로 위와 같은 경우에는 사용할 수 없습니다. 물론 보다 전통적인 방법으로 `required=True` 파라미터와 같은 플래그를 추가하는 방법이 있습니다.

# 결론
이제 Ellipsis 객체가 무엇이고 어디에 사용될 수 있는지 알았습니다. 당신은 다른 사용 사례를 알고 있으신가요?

> 역자주: FastAPI에 적용된 사례는 곰곰히 생각하면 '저렇게도 사용할 수 있겠구나'라고 생각되지만, 보기전까지 저렇게 생각하기란 정말 어려운 거 같습니다.
