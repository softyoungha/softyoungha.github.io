---
title: Python 프로젝트 - 1. Python에서 Database Config 관리하기 - (1) File로 관리하기
author: Youngha Park
date: 2021-09-02 11:33:00 +0800
categories: [Python, 프로젝트 구성]
tags: [Python, Configuration, Project Structure]
math: true
mermaid: true
image:
  src: /python.jpg
  width: 680
  height: 440
---

# Configuration?

하나의 python 프로그램에서 Mysql 이나 Postgres DB에 동시에 접속해야 할 때,
혹은 여러 Mysql database에 접속하거나 여러 개의 GCP 계정이나 AWS 계정을 이용해야 할 때,
간결하고 확실하게 프로젝트를 구성하는 법에 대해서 글을 적어보려 합니다.

좀 더 구체적인 상황을 예로 들어보겠습니다.

만약 하나의 mysql 에 접속하는 상황이라면, `pymysql` 라이브러리를 사용해서 다음과 같이 접속할 수 있습니다.

- Simple Method

```python
# Simple, but not recommended
import pymysql

def load_conn():

  # get connection
  conn = pymysql.connect(host='my-host',
                         port=3306,
                         user='youngha',
                         password='my-password',
                         database='my-database')

  return conn
```

위의 Simple Method를 사용하게 되면 다양한 database에 접근해야 할 때 코드를 간결하게 작성하기 어렵습니다.

이때, configuration 정보를 dictionary 형태로 관리하게 되면 좀 더 간결하게 작성할 수 있습니다.

여기에 더해서 `profile_name` 이라는 속성을 추가하여 db config를 구분하겠습니다.

```python
# BETTER
import pymysql

db_config = {
  # profile_name = 'yh-1'
  'yh-1': {
    'host': 'my-host-1',
    'port': 3306,
    'user': 'youngha-1',
    'password': 'my-password-1',
    'database': 'my-database-1'
  },
  # profile_name = 'yh-2'
  'yh-2': {
    'host': 'my-host-2',
    'port': 8306,
    'user': 'youngha-2',
    'password': 'my-password-2',
    'database': 'my-database-2'
  }
}

def load_conn(profile_name: str):

    # get db_config(profile_name이 없을 경우 KeyError)
    _db_config = db_config[profile_name]

    # get connection
    conn = pymysql.connect(**_db_config)

    return conn

conn = load_conn('yh-1')
```

mysql 뿐만 아니라 postgres까지 접속해야 하는 상황이라면 어떻게 할까요?

postgres는 `psycopg2` 라이브러리를 사용하므로 if문을 사용해야 할 것입니다.

좀 더 general하게 코드를 짜기 위해서 `Database Driver`라는 속성을 추가하겠습니다.

```python
# MORE BETTER
db_config = {
  # profile_name = 'mysql-yh-1'
  'mysql-yh-1': {
    'driver': 'mysql',  # Driver 추가
    'host': 'my-host-1',
    'port': 3306,
    'user': 'youngha-1',
    'password': 'my-password-1',
    'database': 'my-database-1'
  },
  # profile_name = 'mysql-yh-2'
  'mysql-yh-2': {
    'driver': 'mysql',  # Driver 추가
    'host': 'my-host-2',
    'port': 8306,
    'user': 'youngha-2',
    'password': 'my-password-2',
    'database': 'my-database-2'
  },
  # profile_name = 'postgres-yh'
  'postgres-yh': {
    'driver': 'postgres',   # Driver 추가
    'host': 'my-host-3',
    'port': 5432,
    'user': 'youngha-3',
    'password': 'my-password-3',
    'database': 'my-database-3',
    'schema': 'public'
  },
}

def load_conn(profile_name: str):

    # get db_config(profile_name이 없을 경우 KeyError)
    _db_config = db_config[profile_name]

    # get driver(driver가 없을 경우 KeyError)
    driver = _db_config['driver']

    # get connection
    if driver == 'mysql':
        import pymysql
        conn = pymysql.connect(**_db_config)

    elif driver == 'postgres':
        import psycopg2
        conn = psycopg2.connect(**_db_config)

    else:
        raise AssertionError("driver는 mysql과 postgres 둘 중 하나만 지원됩니다.")

    return conn
```

이렇게 connection을 불러오는 함수를 세팅해놓으면 후에 profile_name만 입력해서 해당 connection을 불러오는 것이 쉽습니다.

# Config 저장은 어떻게?

두가지 방법이 있습니다.

1. Python 소스 코드에 **Dictionary** 형태로 저장

2. **별도의 파일**로 저장하며 Python 내부에서 로드하여 사용

1번을 따르시려면 위의 `MORE BETTER` case처럼 코드 상에 host, port, user, password 모두 노출하는 것입니다.

개인적으로 진행하는 프로젝트나 짧은 분석 프로젝트(산출물이 소스코드가 아닌 경우)에는 1번과 같이 노출해도 무방할 것입니다.

상황에 따라 둘 중 하나를 사용하면 됩니다만,
다음의 상황들에서는 무조건 2번 방법을 추천합니다.
(웬만하면 2번으로 하시는게..)


- **개발과 운영 환경**을 별도로 구성하는 경우

  - 만약 Python 소스에 Dictionary 형태로 저장하게 되면
    개발 환경에서도 운영 환경에 접근하는 Config 정보를 가지고 있다는 것인데,
    개발 환경과 운영 환경은 격리되는 것이 보통 맞습니다.
    (알고보니 개발에서 운영 DB에 접근해서 데이터를 갈아엎고 있었다거나.. 실수의 스케일이 달라집니다.)

- Flask, Django, FastAPI 등의 **웹서버**에서 사용하는 경우

  - DB 접속정보를 수정해야 할 필요가 있는 경우,
    Python 소스 내에 위치하게 되면 코드를 수정한 후 웹서버를 재실행해주어야 변경된 접속정보가 반영됩니다.
  - 당연히 재실행해야 하는 것이 맞을 수도 있지만, 한창 개발하는 상황에서 리스타트 한번이 매우 번거로운 일이 될 수 있습니다.
    (개발 편의성을 위해)

이 외에도 Python 소스에 위치하면 잘 모르는 사람이 만지기에 무섭다 라는 단점 때문인 것도 있습니다.

# Config 파일 형식

저는 주로 `.json`, `.yaml`, `.cfg` 세가지 중 하나를 사용합니다.

python `list`와 `dict` 데이터 타입과 1:1로 매핑되도록 저장할 수 있어야 하는 것이 중요합니다.

### 1. Json

Python의 Dictionary 타입과 매우 유사하기 때문에 처음엔 많이 사용했습니다만,
은근히 Json 형식이 엄격한 탓에 최근에는 잘 사용하지 않고 있습니다.

콤마(`,`), 그리고 따옴표(`'`)와 쌍따옴표(`"`)의 차이 등을 주의해서 사용해야 합니다.
(찾기 힘들 때도 있습니다.)

```json
// db_config.json
{
  "mysql-yh": {
    "driver": "mysql",
    "host": "my-host-1",
    "port": 3306,
    "user": "youngha-1",
    "password": "my-password-1",
    "database": "my-database-1"
  },
  "postgres-yh": {
    "driver": "postgres",
    "host": "my-host-2",
    "port": 5432,
    "user": "youngha-2",
    "password": "my-password-3",
    "database": "my-database-3",
    "schema": "public"
  }
}
```

파이썬 내부에서는 `json` 라이브러리를 통해 읽을 수 있습니다.

```python
import json

def load_json(file_path: str) -> dict:
    with open(file_path) as f:
        _dict = json.load(f)
    return _dict

# json -> dictionary
db_config: dict = load_json('db_config.json')
```

### 2. Yaml

최근에는 Yaml 파일을 주로 사용합니다.
보기 쉽고 개발자가 아니여도 제대로 작성했는지 확인하기 쉽습니다.
그리고 Json과 다르게 주석도 추가할 수 있습니다.

주의할 점은 숫자와 문자열 처리를 제대로 처리해줘야 한다는 점입니다.
> Yaml 내에서 `list` 타입과 `dict` 타입을 구별하여 표현하는 것은 따로 검색해서 찾아보세요.
> 아래 예시는 dict만을 표현하고 있습니다.

```yaml
# db_config.yaml

mysql-yh:
  driver: mysql
  host: host-1
  port: 3306
  user: youngha-1
  password: my-password-1
  database: my-database-1

postgres-yh:
  driver: postgres
  host: host-2
  port: 5432
  user: youngha-2
  password: my-password-1
  database: my-database-2
  schema: public
```

파이썬 내부에서는 `yaml` 라이브러리를 통해 읽을 수 있습니다.

```python
import yaml

def load_yaml(file_path: str) -> dict:
    with open(file_path) as f:
        _dict = yaml.load(f, Loader=yaml.FullLoader)
    return _dict

# yaml -> dictionary
db_config: dict = load_yaml('db_config.yaml')
```

### 3. Cfg

python에서 cfg 파일을 로드할 때 `configparser` 라는 라이브러리를 사용하는데,
json이나 yaml과 살짝 사용법이 다릅니다.
좀 더 있어보이게 Config 파일을 구성할 수 있습니다.

> 참고로 Airflow 내에서도 `airflow.cfg` config 파일을 사용합니다.

```editorconfig
# db_config.cfg

[mysql-yh]
driver = mysql
host = host-1
port = 3306
user = youngha-1
password = my-password-1
database = my-database-1

[postgres-yh]
driver = postgres
host = host-2
port = 5432
user = youngha-2
password = my-password-1
database = my-database-2
schema = public
```

```python
import configparser

def load_cfg(file_path: str) -> dict:

    config = configparser.ConfigParser()
    config.read(file_path, encoding='utf-8')

    return dict(config)

# yaml -> dictionary
db_config: dict = load_cfg('db_config.cfg')
```


# 개발/운영 환경 분리시 세팅 예시

위의 방법을 적용하여 개발 환경과 운영 환경을 구분해야 하면 다음과 같이 세팅할 수 있습니다.

### 환경 변수 설정

다른 프로그램에서 사용하는 환경변수와 겹칠 수 있으므로 Unique할 수 있도록 Prefix를 하나 정해서 환경변수를 추가하는 것이 좋습니다.

```shell
# vi ~/.bashrc

...
# 개발(DEV)/운영(PROD) 환경 구분
export PROJECT_ENV = 'DEV'
export PROJECT_CONF_DIR = '~/.config/project'
```

### Config 파일

리눅스의 경우 Dot(`.`)을 폴더/파일 앞에 붙히면 해당 파일을 숨길 수 있습니다.
(`ls -al`로 확인할 수 있습니다.)

개발환경과 운영환경이 각각 다른 config 파일을 바라볼 수 있도록 config 파일도 두 개 준비합니다.
(바라보는 db도 다릅니다.)

  - 개발서버: ~/.config/project/db_config.yml
  - 운영서버: ~/.config/project/db_config.prod.yml

```yaml
# ~/.config/project/db_config.yml

mysql-yh:
  driver: mysql
  host: host-1
  port: 3306
  user: youngha-1
  password: my-password-1
  database: my-database-1

postgres-yh:
  driver: postgres
  host: host-2
  port: 5432
  user: youngha-2
  password: my-password-1
  database: my-database-2
  schema: public
```

### Python 코드

```python
# python 코드
import os
import yaml

# 환경 변수 get
PROJECT_ENV = os.getenv('PROJECT_ENV') or 'DEV'
PROJECT_CONF_DIR = os.getenv('PROJECT_CONF_DIR') or '~/.config/project'

if PROJECT_ENV == 'DEV':
    # 개발 서버일 경우 db_config.yml을 바라봅니다.
    db_config_path = os.path.join(PROJECT_CONF_DIR, 'db_config.yml')

elif PROJECT_ENV == 'PROD':
    # 운영 서버일 경우 db_config.prod.yml을 바라봅니다.
    db_config_path = os.path.join(PROJECT_CONF_DIR, 'db_config.prod.yml')

def load_yaml(file_path: str) -> dict:
    with open(file_path) as f:
        _dict = yaml.load(f, Loader=yaml.FullLoader)
    return _dict


def load_conn(profile_name: str):

    # 함수를 실행할 때마다 새롭게 read 하도록 함수 내부에 위치시킵니다.
    # 도중에 config 파일이 수정될 경우 프로그램을 리스타트하지 않아도 수정된 내용으로 반영됩니다.
    db_config = load_yaml(db_config_path)

    # get db_config(profile_name이 없을 경우 KeyError)
    _db_config = db_config[profile_name]

    # get driver(driver가 없을 경우 KeyError)
    driver = _db_config['driver']

    # get connection
    if driver == 'mysql':
        import pymysql
        conn = pymysql.connect(**_db_config)

    elif driver == 'postgres':
        import psycopg2
        conn = psycopg2.connect(**_db_config)

    else:
        raise AssertionError("driver는 mysql과 postgres 둘 중 하나만 지원됩니다.")

    return conn

conn = load_conn('mysql-yh')
```

# 정리

위의 예시는 여러 데이터베이스에 접근해야할 때만을 예시로 들었습니다.
그 외에도 Redis, AWS S3 Bucket, AWS Console, 타 API 등 많은 연동되는 정보를 관리해야할 때
위의 방법을 따르면 충분히 해결하실 수 있을 겁니다.

- 실제 프로젝트 구성시 config 관리
  ![Pycharm Setting Example](/posts/python-project-config-control/img-01.png)

이 포스트에서 다루지 않은 내용으로 `Credential`이 있는데,
Credential 정보는 config 파일 내 혹은 소스 코드 내에서 평문으로 작성되면 문제가 될 여지가 있습니다.
해당 내용은 다음 포스트에서 다루겠습니다.


