---
layout: post
title: Docker Compose and Pytest
description: 
date: '2021-08-30'
tags: programming
---

A note on running pytest through docker compose. 


## Scenario

Suppose we have a local development setup with docker compose. It contains different services some built from sources others created from images like a database that's used for running the integration tests locally. 


Can the same setup be used on the CI server ? That is to run tests, code checkers formatters or other tools from within docker.

If one does not want to mess with the CI server this approach provides a good isolation from whatever environment is installed on the server. Then the CI server only needs the ability to run docker and it can build anything.

## Setup

`docker-compose run ...` can help accomplish this.


Source code

```python
# test_integrations.py 

def test_integrations():
    # an integration test accessing the database 
    postgres_url = os.getenv("POSTGRES_URL")    
    # do some work

# app.py

if __name__ == "__main__":
    # the main entry point, a web app or whatever
    pass
```

Docker

```bash
# Dockerfile

FROM python:3.8.8

WORKDIR /usr/src/app

COPY requirements.txt /usr/src/app/

RUN pip install pip --upgrade &&  pip install -r requirements.txt

COPY *.py /usr/src/app/

CMD [ "python" "app.py" ]
```

```bash
# docker-compose.yml

version: '2'
services: 
  app:
    build: .
    environment:
        POSTGRES_URL: postgresql://dbuser:dbpass@db:5432/postgres
    links:
      - db

  db:
    image: postgres:latest
    environment:
        POSTGRES_USER: dbuser
        POSTGRES_PASSWORD: dbpass
```

Usage. Locally and on CI

```
# spin up the database (optionally)

docker-compose up db 

# run pytest 

docker-compose run app pytest .

# run black 

 docker-compose run app black . --check

# run mypy 

docker-compose run app mypy .
```
