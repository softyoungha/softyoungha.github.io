---
title: 2. Typing 은 권장이 아니라 필수!
author: Youngha Park
date: 2021-09-09 11:33:00 +0800
categories: [Python, 기초]
tags: [Python, Typing]
math: true
mermaid: true
image:
  src: https://cdn.jsdelivr.net/gh/softyoungha/softyoungha/blog/img/python.png
  width: 680
  height: 440
---

다음은 제가 실제 사용하는 함수 중에 Memory 사용량을 추적하기 위해서 사용하는 함수 하나를 가져온 것입니다.

```python
def get_memory_usage(as_msg):
    import psutil
    mem = psutil.virtual_memory()
    divide = 1024 ** 3

    total = f'{mem.total / divide:.1f}G'
    available = f'{mem.available / divide:.1f}G'
    percent = f'{mem.percent:.1f}%'
    used = f'{mem.used / divide:.1f}G'

    if as_msg:
        return f'Total: {total}, Available: {available}, percent: {percent}, used: {used}'

    else:
        return {
            'total': mem.total,
            'available': mem.available,
            'percent': mem.percent,
            'used': mem.used,
        }
```

간단히 설명드리면 `as_msg = True`일 경우 현재 메모리 상태를 String 형태로 반환하고,
`as_msg = False`일 경우 Dictionary 형태로 반환합니다.

다행히도 함수명 `get_memory_usage`이나 parameter 명칭 `as_msg`이 알아볼만 하고
함수의 구조도 단순하여 이 함수를 어떻게 쓸지 다른 사람도 쉽게 파악할 수 있을 듯 합니다.

하지만, 나만 쓰는 함수가 아니라면 이 함수를 설명해주는 documentation을 상세히 해주어야 합니다.

(사실 나만 쓰는 함수라도 이렇게 하면 안됩니다)

주석 및 문서화를 하는 것은 습관적으로 하면 좋습니다.

다만, 이를 구구절절히 다 적는 것은 함수 내용이 바뀔 때마다 수정해주어야 하기 때문에 상당히 피곤한 일입니다.

```python
def get_memory_usage(as_msg):
    """
    현재 메모리를 출력합니다.

    :keyword total: 전체 메모리 사용량
    :keyword available: 사용 가능한 메모리(G)
    :keyword percent: 사용 중인 메모리(%)
    :keyword used: 사용 중인 메모리(G)
    :parameter as_msg: [bool] True일 경우 string으로, False일 경우 dict로 반환합니다.
    :return [str or dict] 현재 메모리
    """
    ...

# __doc__로 함수의 documentation을 불러올 수 있다.
print(get_memory_usage.__doc__)
```

여기서 다른 사용자들(+ 기억이 안나는 미래의 나)이 파악할 수 있도록
함수의 input과 output의 데이터 타입을 `typing`을 사용하여 알려줄 수 있습니다.

```python
# simple typing
def get_memory_usage(as_msg: bool):
    ...
```

위와 같이 `as_msg` input parameter가 `bool` type임을 알려줄 수 있습니다.

여기에 더해, return의 데이터형식도 알려줄 수 있습니다.

```python
def get_memory_usage(as_msg: bool) -> str:
    ...
```

그런데, 위의 예제에서는 as_msg의 조건문을 통해 return 값이 `str`이 될수도 있고 `dict`가 될 수도 있습니다.

이 경우 `typing` 패키지로 처리할 수 있습니다.

```python
from typing import Union

def get_memory_usage(as_msg: bool) -> Union[str, dict]:
    ...
```

다른 사용자들은 위의 함수를 보고 `as_msg` argument는 `bool` type이구나, return 값의 type은 `str`이나 `dict` 구나! 하고 알 수 있습니다.

---

더 자세히 하고 싶다면 개별 변수마다 데이터 형태를 적어놓을 수 있습니다.

```python
divide: int = 1024 ** 3

total: str = f'{mem.total / divide:.1f}G'
available: str = f'{mem.available / divide:.1f}G'
percent: str = f'{mem.percent:.1f}%'
used: str = f'{mem.used / divide:.1f}G'
```

물론 누가봐도 int고 누가봐도 str인 경우 굳이 적을 필요는 없습니다만 다음과 같이 int나 str같은 단순한 데이터타입이 아닌 경우에 사용하면 좋습니다.

```python
# 커스텀 Class
class MyClass(object):
    def __init__(self):
      pass

# 실행하면 MyClass instance를 반환하는 함수
def my_func() -> MyClass:
    a = MyClass()

    return a

# my_func를 실행해서 받는 데이터 형이 MyClass 타입일 거야!
my_var: MyClass = my_func()
```

위의 예제는 간단하지만, Class를 정의하는 코드와 함수를 정의하는 코드, 그리고 함수를 사용하는 코드가 모두 다른 부분에 있는 경우,
타고타고 가서 내용을 파악하는 것이 매우 어렵습니다.

간단하지만 이렇게 typing만 해주는 것만으로도 코드를 파악하기 매우 쉽습니다.

> 주의하실 점은 typing은 타입을 명시해주는 것뿐이지 에러를 일으키진 않습니다.
>
> typing의 목적은 type annotation입니다.
>
> 별도의 검증을 추가하기 위해서는 `assert`, `isinstance`를 사용해서 `AssertionError`를 일으키도록 구성해야 합니다.

```python
def my_func(number: int):

    # raise AssertionError
    assert isinstance(number, int), f"number input은 반드시 `int` 타입이여야 합니다.: {number}"

    ...
```

---

좀 더 다양한 케이스를 다뤄보겠습니다.

`typing` 패키지를 사용해서 더 다양한 경우를 다룰 수 있는 데요, 대표적으로 List, Dict, Any, Callable 등이 있습니다.

## typing: List

간단한 함수라면 Python built-in 데이터타입인 `list`를 사용하면 될 겁니다.

```python
my_list: list = [3, 5, 6, 7]
```

여기서 list 내의 element들의 type이 모두 같다면 `typing`을 사용해서 다음과 같이 적을 수 있습니다.

```python
from typing import List

my_list: List[int] = [3, 5, 6, 7]
```

## typing: Any, Union

List 내의 element들이 모두 int가 아니라면 어떻게 할까요?

element들의 타입을 특정할 수 없다면 Any를 사용해도 괜찮습니다.

```python
from typing import List, Any

# is it alright?
my_list: List[Any] = [3, 5, 6, 7, '하이룽', [3, 4, 5]]
```

그나마 element 타입을 알 수 있다면 Union을 사용합니다.

```python
from typing import List, Union

# is it alright?
my_list: List[Union[int, str]] = [3, 5, 6, 7, '하이룽']
```

> 이렇게 documentation 할 수 있다 정도이지만 이렇게까지 할 필요는 없습니다.
>
> 만약 이렇게까지 적어야하는 상황이 왔다면 데이터 설계가 잘못된 게 아닌지 의심해볼 필요가 있습니다.


## typing: Dict

dictionary는 Key와 Value로 나누어지기 때문에 두가지 모두 타입을 지정해주어야 합니다.

```python
from typing import Dict

my_dict: Dict[str, int] = {
  'hi': 1,
  'my': 2,
  'name': 3,
  'is': 4
}
```


## typing: Callable

함수가 반환되는 경우에는 Callable을 사용합니다.

```python
from typing import List, Callable

# 함수 생성
def my_func_1st():
    print('hi')

# lambda 함수 input, output 데이터타입 설명
my_func_2nd: Callable[[str], str] = lambda msg: f'hello {msg}'

# 함수 리턴
def my_func_3rd() -> List[Callable]:
    return [my_func_1st, my_func_2nd]
```


## typing: Optional

None이 허용되는 값일 경우 사용합니다.

`Optional[str]` 는 `Union[str, None]`과 같습니다.

```python
from typing import Optional

def give_message(as_msg: bool, msg: Optional[str] = None) -> Optional[str]:
    if as_msg:
        return msg
    else:
        return None
```

---

# 정리

코드를 적으실 때 나만 지금 잠시 돌리고 지울 것이 아니라면 반드시 주석이나 별도의 문서화를 진행하셔야 합니다.

이 때 `typing` 패키지를 사용해서 최소한의 설명을 추가하는 것이 무조건 좋기 때문에,
권장이 아니라 필수로 코드에 반영하도록 합시다.

> 만약 Pycharm 유저라면 typing을 코드에 잘 반영해놓았다면 엄청난 이점이 있습니다.
>
> 어떤 변수의 class를 추적하기 어렵게 코드가 복잡한 경우,
> typing을 해놓는다면 Pycharm에서 해당 변수의 class를 연결해주어서 class 내부 함수들도 자동완성으로 사용할 수 있습니다.
