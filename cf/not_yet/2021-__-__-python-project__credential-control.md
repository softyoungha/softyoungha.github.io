---
title: Python에서 Credential 저장하기
author: Youngha Park
date: 2021-09-02 11:33:00 +0800
categories: [Python, 프로젝트 구성]
tags: [Python]
math: true
mermaid: true
image:
  src: /python.png
  width: 680
  height: 440
---

# Credential?

프로그램에 접근 권한을 부여하는 자격 증명이라 생각하면 쉽습니다.

Credential 정보, 특히 Admin에 대한 정보는 민감하게 다뤄지는 것이 맞습니다.

간단하게 생각하면 `비밀번호` 라고 생각하시면 됩니다.

- 예를 들자면 Github에 로그인할 때 ID/Password,
  AWS S3 에 접근하기 위한 ACCESS_KEY/SECRET_KEY,
  Jupyter Server에 접근하기 위한 Password 등이 있습니다.

- 장고 웹페이지를 AWS 서버에 띄우는 경우

  - 서버로의 SSH 접근 Key 혹은 Id/Password
  - 백엔드로 사용하는 Mysql Database의 host/port/id/password
  - Django FERNET_KEY
  - 미디어 파일 및 static 파일을 저장하는 S3의 ACCESS_KEY/SECRET_KEY
  - Django Admin Id/password

  이외에도 여러가지 민감하게 저장되어야 하는 정보들이 많습니다.

- Credential은 평문으로 저장하면 안 됩니다.

  - 대기업들과 하는 분석/개발 프로젝트들에서는 보통 마지막에 보안성 체크를 진행하게 되는데,
    이 항목이 대부분 단골로 등장합니다.

# 그럼 어떻게?

### Simple Credential

하나의 값만 저장해야 할 때는 다음과 같이 사용합니다.

1. 먼저 python을 실행하는 서버에 `환경 변수`를 설정합니다.

- Linux라면,
  ```bash
  # vi ~/.bashrc

  ...
  export DJANGO_FERNET_KEY="abcd"
  ```

- Pycharm을 사용한다면 `environment variables`에 입력하시면 됩니다.

  - Pycharm 세팅에서 terminal, python console, run configuration 등등 여러 곳에서 설정할 수 있습니다.

  ![Pycharm Setting Example](posts/2021-09-02-python-01-credential-control/img-01.png)

2. python 프로그램 내에서 환경 변수를 가져와 사용합니다.

> 주의: 새롭게 설정한 환경 변수를 사용하려면 이미 실행되고 있던 python은 재부팅해야 합니다.
> Jupyter notebook이라면 kernel restart, Fastapi나 Django webserver라면 다시 실행해야 합니다.

```python
import os

# os.getenv 로 get 해올 수 있습니다.
DJANGO_FERNET_KEY = os.getenv('DJANGO_FERNET_KEY')

# os.environ dictionary 에서 직접 get으로 가져오는 것과 동일합니다.
DJANGO_FERNET_KEY = os.environ.get('DJANGO_FERNET_KEY')

# 값이 없으면 assertion Error를 일으키도록 설정하는 것도 좋습니다.
assert DJANGO_FERNET_KEY is not None, f"'DJANGO_FERNET_KEY'의 값이 있어야 합니다."
```

### Credential Set

위의 방법으로 여러 Credential을 설정할 수 있습니다만, Database 접근할 때처럼 'Credential 뭉치'를 저장하는 것이 편할 때가 있습니다.

  - 하나의 mysql db에만 접근하는 상황이라면 host, port, user, password 정보를 저장해야 합니다.
  - 두개의 mysql db에 접근해야 하는 상황이라면, 거기다 다른 postgres db까지 접근해야 하는 상황이라면 더욱 복잡해질 수 있습니다.
  - 이를 구별하기 위해 configu

이때 파일 위치를 하나로 특정해놓고 해당 파일에 정보를 저장해놓는 것이 좋습니다.

보통 config 정보를 저장하는 파일 형식으로는 json, yaml, cfg 가 있습니다.



1.
