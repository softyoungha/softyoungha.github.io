---
title: Python 기초 - 1. List & Dictionary는 언제 사용해야 할까?
author: Youngha Park
date: 2021-09-04 11:33:00 +0800
categories: [Python, 기초]
tags: [Python, List, Dictionary, Data Handling, List Comprehension]
math: true
mermaid: true
image:
  src: /python.jpg
  width: 680
  height: 440
---

&nbsp;
처음 python을 공부할 때 조금 아리송했던 기억이 있습니다.
학부 시절 저는 컴퓨터 공학 출신이 아니기에 당시에 자료 구조 수업을 들어본 적이 없었고,
python을 공부할 때 구글 검색을 통해서 아무 블로그의 포스트를 참고하거나 Stackoverflow Q&A를 참고하여
직접 실행해보고 체화하는 식으로 공부를 진행했습니다.

&nbsp;
당시 저의 관심사가 딥러닝이다 보니 인터넷에서 도둑질하던 포스트들 중
어떤 이들은 Numpy Array로 무조건 변환해서 사용하고,
또 다른 이들은 Pandas DataFrame 로 표현한 다음 Numpy Array를 섞어쓰는 등 중구난방이더군요.

일단 코드의 시작부터 numpy와 pandas를 부르고 Numpy Array와 Pandas DataFrame만 사용하는 코드도 많았습니다.
```python
import numpy as np
import pandas as pd
```
(무슨 고집이였는지 모르겠는데 저는 `as pd`로 alias 쓰는 것도 싫어했습니다)

그래서 다음과 같은 고민을 늘 해왔던 것 같습니다.

> 언제 list를 사용하지?
>
> dictionary는 언제 쓰지?
>
> pandas DataFrame도 좋고 Numpy Array도 좋지만.. 적재적소에서만 사용하고 싶은데?
>
> 좀 더 간결하게, 좀 더 fancy하게 표현하는 방법이 없을까?

그래서 이 고민과 함께 오늘 포스트에서는 파이썬 기본 자료형 List와 Dict(Dictionary)에 대해서 이야기해보려 합니다.

기본적인 내용 말고 특히 `언제 사용할까`, `어떻게 사용하는 게 잘 쓰는 걸까`에 초점을 맞추겠습니다.

나머지 자료형 set, tuple, OrderedDict 등에 대해서는 list와 dict만 잘 이해하셔도 특징만 잘 기억한다면 충분히 잘 사용할 수 있습니다.

---

# List

python 프로그램을 짤 때 우선적으로 데이터를 python 내에서 표현하는 단계부터 시작합니다.

이 때 데이터 구조를 table 형태로 정의하는 것이 경험상 무조건 좋습니다.

list와 dict로 table을 충분히 표현할 수 있으며
나중에 실제 DB에 적재할 때도 변환 없이 그대로 표현할 수 있습니다.

다음은 python 기초를 다루는 내용에서 자주 보이는 예제입니다.

```python
users = ['영희', '철수', '민수', '범배']
print(users)
# > ['영희', '철수', '민수', '범배']

user_id = users.index('민수')
print(user_id)
# > 2

print(users[user_id])
# > '민수'
```

쉬운 예제이지만 좀 더 어렵게 생각해보겠습니다.

> index, username을 column로 가지는 users라는 테이블이 있는데,
>
> 그 안에 데이터로 '영희', '철수', '민수', '범배' 의 username을 가진 데이터가 4개 있다.

실제 database 라면 테이블을 다음과 같이 생성할 수 있습니다.

```mysql
CREATE TABLE  users (
    index       INT             auto_increment      NOT NULL,
    username    varchar(100)                        NOT NULL,
    CONSTRAINT  users_PK PRIMARY KEY (username)
  )
```

database 가 뭔지 잘 모르는 분들이라면 primary key라는 개념만 잘 이해하시면 될 것 같습니다.

`primary key`란 table에서 각 row마다 `unique`하게 가지는 column입니다.

List에서는 모든 element들이 따로 설정하지 않아도 index라는 primary key를 지니고 있습니다.

위의 예제를 다시 풀어보자면,
유저 리스트(`users`)를 우리가 모르는 상황에서
`username`이 '민수'인 row의 primary key를 찾는 상황으로 해석할 수 있습니다.

다음과 같이 테이블에 다른 속성(feature, column)을 추가할 수도 있습니다.

```python
users = [
  # username, age, gender
  ['영희', 13, 'F'],
  ['철수', 20, 'M'],
  ['민수', 21, 'M'],
  ['범배', 19, 'F']
]

for username, age, gender in users:
    print(f'username: {username}, age: {age}, gender: {gender}')

# > username: 영희, age: 13, gender: F
# > username: 철수, age: 20, gender: M
# > username: 민수, age: 21, gender: M
# > username: 범배, age: 19, gender: F
```

위와 같이 list of list(`List[List]`) 형태로 표현하게 되면 column 순서를 지켜야만 하는 단점이 있습니다.

또한 주석을 필수로 추가해야 하며 Nullable column을 표현할 때도 자리를 무조건 채워주어야 합니다.

```python
users = [
  # username, age, gender, location
  ['영희', 13, 'F', 'Seoul'],
  ['철수', 20, 'M', 'Seoul'],
  ['민수', 21, 'M', 'Busan'],
  ['범배', 19, 'F', None]
]
```

# Dict

위의 상황을 dict로 표현하는 것은 어렵지 않습니다.

```python
# as list
user = ['영희', 13, 'F', 'Seoul']

# as dict
user = {
  'username': '영희',
  'age': 13,
  'gender': 'F',
  'location': 'Seoul'
}
```

키 포인트는

- list는 순서가 있다.
- dict는 순서가 없다.
- list는 index가 primary key로 사용된다.
- dict는 key를 통해 value를 가져온다(get).

Dict를 적용해서 위의 예제를 다음과 같이 수정할 수 있습니다.

```python
users = [
  {'username': '영희', 'age': 13, 'gender': 'F', 'location': 'Seoul'},
  {'username': '철수', 'age': 20, 'gender': 'M', 'location': 'Seoul'},
  {'username': '민수', 'age': 21, 'gender': 'M', 'location': 'Busan'},
  {'username': '범배', 'age': 19, 'gender': 'F'},
]

for user in users:
    username = user.get('username')
    age = user.get('age')
    gender = user.get('user')
    location = user.get('location') or '모름'
    print(f'username: {username}, age: {age}, gender: {gender}, location: {location}')
```

정리하자면 다음과 같은 대응 관계를 생각하면 됩니다.

- users: list <-> table
- user: one dict element of list <-> one row of table


# Filtering

데이터베이스에서 사용되는 주요 기능 중 하나가 `검색`입니다.

나이가 18살 이상인 user의 이름을 출력하는 query는 다음과 같습니다.

```mysql
SELECT  username
FROM    users
WHERE   age > 18
```

python에서는 다음과 같이 여러 방법으로 표현할 수 있습니다.

```python
##### 1. for-loop(처음에는 먼저 이렇게 해봅니다.)

# empty list
filtered_users = []

# start for loop
for user in users:

    # get
    age = user.get('age')
    username = user.get('username')

    # if statement
    if age > 18:
        filtered_users.append(username)

##### 2. list comprehension(Recommended)
filtered_users = [user.get('username')
                  for user in users
                  if user.get('age') > 18]

##### 3. map & filter(이런 게 있다)
def mapping(user):
    return user.get('username')

def filtering(user):
    return user.get('age') > 18

filtered_users = map(mapping, filter(filtering, users))

##### 4. map & filter with lambda(이렇게도 할 수 있다)
filtered_users = map(lambda user: user.get('username'),
                     filter(lambda user: user.get('age') > 18, users))
```

저는 보통 for-loop로 먼저 코딩을 시도해보는데,

조건문이 너무 까다롭거나 select할 element 형태가 복잡할 경우가 아니라면 `list comprehension` 방법으로 수정합니다.

> 하단에 `filter`와 `map`를 사용해서 같은 결과를 얻을 수 있습니다.
>
> `filter & map` vs `list comprehension` 에 대한 논의는 검색해보면 성능면에서 차이가 있다고 합니다만,
>
> 저는 큰 차이가 없다고 생각해서 더 보기 좋은 list comprehension을 주로 사용합니다.

여러 column을 가져올 때에는

```mysql
-- in sql
SELECT  username, location
FROM    users
WHERE   age > 18 AND username LIKE '%수%'
```

```python
# in python
filtered_users = [
  # select
  {
    'username': user.get('username'),
    'location': user.get('location'),
  }

  # from
  for user in users

  # where
  if user.get('age') > 18 and '수' in user.get('username')
]
```

처럼 표현할 수 있습니다.

# Aggregation

database 의 또 다른 주요 기능으로 `Group by`문을 이용한 Aggregation이 있습니다.

지역(location)별 명수(ct)와 나이 평균(max_age)을 구한다면 다음과 같은 쿼리로 표현할 수 있습니다.

```mysql
-- in sql
SELECT      location,
            COUNT(1) AS ct,
            MAX(age) AS max_age
FROM        users
WHERE       username <> '범배'
GROUP BY    1
```

python에서 표현하려면 조금 복잡하게 보일 수 있습니다.

```python
# filtering
filtered_users = [user
                  for user in users
                  if user.get('username') != '범배']

# set -> list(unique)
locations = list({user.get('location') for user in filtered_users})

# aggregation
results = [
  {
    'location': location,
    'ct': len([1
               for user in filtered_users
               if user.get('location') == location]),
    'max_age': max([user.get('age')
                    for user in filtered_users
                    if user.get('location') == location])
  }
  for location in locations
]
```

# 정리

파이썬에서 데이터 핸들링시 중구난방으로 List, Dict, Set 등 여러 형태로 특정 룰 없이 정의할 경우,
나중에 스스로 코드를 볼 때뿐만 아니라 다른 사람과 협업할 때에도 설명할 게 많아집니다.

데이터를 구조화하는 데에 위와 같은 특정 룰을 따른다면 Pandas 나 Numpy 와 같은 다른 라이브러리와 호환도 쉽습니다.

(pandas DataFrame은 애초에 위와 같은 룰을 따르고 있기 때문에 특별한 변환을 해줄 필요도 없습니다.)

```python
import pandas as pd

# data
users = [
  {'username': '영희', 'age': 13, 'gender': 'F', 'location': 'Seoul'},
  {'username': '철수', 'age': 20, 'gender': 'M', 'location': 'Seoul'},
  {'username': '민수', 'age': 21, 'gender': 'M', 'location': 'Busan'},
  {'username': '범배', 'age': 19, 'gender': 'F'},
]

# as pandas DataFrame
df = pd.DataFrame(users)

print(df)
# >         username    age     gender      location
# > 0       영희         13      F           Seoul
# > 1       철수         20      M           Seoul
```

List와 Dict 를 사용해서 대부분의 데이터는 표현할 수 있지만,
그럼에도 Numpy/Pandas 를 사용하는 대표적인 이유는 다음과 같습니다.

1. 성능 이슈
  - Numpy Array는 동적으로 크기가 변하는 Python List와 달리 고정된 크기를 할당하는 형식이고,
    내부적으로 벡터화하여 연산하기 때문에 일반적으로 Python List보다 일반적으로 빠릅니다.

2. 다른 라이브러리와의 연동
  - sqlalchemy, tensorflow, scikit-learn 등 수학 기반 Python 패키지들이
    Numpy array 및 pandas DataFrame일 기본 자료형으로 사용합니다.

3. 자체로 지원하는 함수
  - Numpy Array나 padnas DataFrame 내장함수를 사용하면 매우 간결하게 코딩할 수 있습니다.

정리하자면, 무조건 Numpy나 Pandas 가 좋은 것이 아니고, 무조건 Python Built-in 으로 처리하는 게 좋은 것을 추천하지 않습니다.

데이터 형식을 고르는 보편적인 룰을 따르는 것이 가장 좋고,
코드를 보다 간결하고 구조를 단순하게 만들 수 있는 것이 더욱 좋습니다.




