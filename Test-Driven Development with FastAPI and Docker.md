# TDD with FastAPI & Docker

## Getting Started

### Setup

- Create a new project and install FastAPI along with [[Uvicorn]], an [[ASGI]] server used to serve up FastAPI: (You may need [[pyenv]] for more details on python environment setup)

```bash
$ mkdir fastapi-tdd-docker && cd fastapi-tdd-docker
$ mkdir project && cd project
$ mkdir app
$ python3.9 -m venv env
$ source env/bin/activate

(env)$ pip install fastapi==0.65.3
(env)$ pip install uvicorn==0.14.0
```

> Feel free to swap out vitualenv and Pip for [Poetry](https://python-poetry.org/) and [Pipenv](https://pipenv.pypa.io/). For more, review [Modern Python Environments](https://testdriven.io/blog/python-environments/)

Add an `__init__.py` file to the "app" directory along with a `main.py` file. Within `main.py`, create a new instance of FastAPI and set up a synchronous sanity check route:

```python
# project/app/main.py


from fastapi import FastAPI

app = FastAPI()


@app.get("/ping")
def pong():
  return {"ping": "pong!"}
```

- That's all you need to get a basic route up and running!
- You should now have:
	- project
		- app
			- `__init__.py`
			- `main.py`

Run the server from the "project" directory:

```bash
uvicorn app.main:app
```

- `app.main:app` tells Uvicorn where it can find the FastAPI application --e.g., "within the 'app' module, you'll find the app, `app = FastAPI()`, in the 'main.py' file.

Navigate to http://localhost:8000/ping in your browser. You should see:

```json
{
"ping": "pong!"
}
```

> Why did we use Uvicorn to serve up FastAPI rather than a development server?
> Unlike Django or Flask, FastAPI does not have a built-in development server. 
> New to ASGI? Read through the excellent [Introduction to ASGI: Emergence of an Async Python Web Ecosystem](https://florimond.dev/blog/articles/2019/08/introduction-to-asgi-async-python-web/)

- FastAPI automatically generates a schema based on the [OpenAPI](https://swagger.io/docs/specification/about/) standard. You can view the raw JSON at http://localhost:8000/openapi.json. This can be used to automatically generate client-side code for a front-end or mobile application. FastAPI uses it along with [Swagger UI](https://github.com/swagger-api/swagger-ui) to create interactive API documentation, which can be viewed at http://localhost:8000/docs:
- kill the server

### Auto-reload

- Let's run the app again. This time, we'll enable auto-reload mode so that the server will restart after changes are made to the code base:

```bash
uvicorn app.main:app --reload
```

- Now when you make changes to the code, the app will automatically reload. Try this out.

### Config

- Add a new file called `config.py` to the "app" directory, where we'll define environment-specific [configuration](https://fastapi.tiangolo.com/advanced/settings/) variables:
```python
# project/app/config.py


import logging
import os

from pydantic import BaseSettings


log = logging.getLogger("uvicorn")


class Settings(BaseSettings):
  environment: str = os.getenv("ENVIRONMENT", "dev")
  testing: bool = os.getenv("TESTING", 0)


def get_settings() -> BaseSettings:
  log.info("Loading config settings from the environment...")
  return Settings()
```

- Here we defined a `Settings` class with two attributes:
1. `environment` - defines the environment (i.e., dev, stage, prod)
2. `testing` - defines whether or not we're in test mode

- [BaseSettings](https://pydantic-docs.helpmanual.io/usage/settings/), from Pydantic, validates the data so that when we create an instance of `Settings`, `environment` and `testing` will have types of `str` and `bool`, respectively.

Update `main.py` like so:

```python
# project/app/main.py


from fastapi import FastAPI, Depends

from app.config import get_settings, Settings


app = FastAPI()


@app.get("/ping")
def pong(settings: Settings = Depends(get_settings)):
  return {
	  "ping": "pong!",
	  "environment": settings.environment,
	  "testing": settings.testing
  }
```

- Take note of `settings: Settings = Depends(get_settings)`. Here, the `Depends` function is a dependency that declares another dependency, `get_settings`. Put another way, `Depends` depends on the result of `get_settings`. The value returned, `Settings`, is then assigned to the `settings` parameter.

If you're new to dependency injection, review the [Dependencies](https://fastapi.tiangolo.com/tutorial/dependencies/) guide from the offical FastAPI docs.

Run the server again. Navigate to [http://localhost:8000/ping](http://localhost:8000/ping) again. This time you should see:

```bash
{
  "ping": "pong!",
  "environment": "dev",
  "testing": false
}
```

- Kill the server and set the following environment variables:
```bash
(env)$ export ENVIRONMENT=prod
(env)$ export TESTING=1
```

Run the server. Now, at http://localhost:8000/ping, you should see:
```bash
{
  "ping": "pong!",
  "environment": "prod",
  "testing": true
}
```

- What happens when you set the `TESTING` environment variable to `foo`? Try this out. Then update the variable to `0`.

- With the server running, navigate to [http://localhost:8000/ping](http://localhost:8000/ping) and then refresh a few times. Back in your terminal, you should see several log messages for:
	- `Loading config settings from the environment...` 

- Essentially, `get_settings` gets called for each request. If we refactored the config so that the settings were read from a file, instead of from environment variables, it would be much too slow.

- Let's use [lru_cache](https://docs.python.org/3/library/functools.html#functools.lru_cache) to cache the settings so `get_settings` is only called once.

Update _config.py_:
```python
# project/app/config.py


import logging
import os
from functools import lru_cache

from pydantic import BaseSettings


log = logging.getLogger("uvicorn")


class Settings(BaseSettings):
    environment: str = os.getenv("ENVIRONMENT", "dev")
    testing: bool = os.getenv("TESTING", 0)


@lru_cache()
def get_settings() -> BaseSettings:
    log.info("Loading config settings from the environment...")
    return Settings()
```

- After the auto-reload, refresh the browser a few times. You should only see one `Loading config settings from the environment...` log message.

### Async Handlers

- Rather than having to go through the trouble of spinning up a task queue (like Celery or RQ) or utilizing threads, FastAPI makes it easy to deliver routes asynchronously. As long as you don't have any blocking I/O calls in the handler, you can simply declare the handler as asynchronous by adding the `async` keyword like so:

```python
@app.get("/ping")
async def pong(settings: Settings = Depends(get_settings)):
    return {
        "ping": "pong!",
        "environment": settings.environment,
        "testing": settings.testing
    }
```


- That's it. Update the handler in your code, and then make sure it still works as expected.

- Kill the server once done. Exit then remove the virtual environment as well. Then, add a _requirements.txt_ file to the "project" directory:

```
fastapi==0.65.3
uvicorn==0.14.0`
```

Finally, add a `.gitignore` to the project root:
```
__pycache__
env
```

- You should now have:
	- .gitignore
	- project
		- app
			- __init__.py
			- config.py
			- main.py
		- requirements.txt

- Init a git repo and commit your code.


## Docker Config

- Let's containerize the FastAPI app.

---

- Start by ensuring that you have Docker and Docker Compose:

```bash
$ docker -v
Docker version 20.10.7, build f0df350

$ docker-compose -v
docker-compose version 1.29.2, build 5becea4c
```


> Make sure to [install](https://docs.docker.com/install/) or upgrade them if necessary.

- Add a _Dockerfile_ to the "project" directory, making sure to review the code comments:

```dockerfile
# pull official base image
FROM python:3.9.6-slim-buster

# set working directory
WORKDIR /usr/src/app

# set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# install system dependencies
RUN apt-get update \
  && apt-get -y install netcat gcc \
  && apt-get clean

# install python dependencies
RUN pip install --upgrade pip
COPY ./requirements.txt .
RUN pip install -r requirements.txt

# add app
COPY . .
```

- Here, we started with a slim-buster-based Docker image for [Python](https://hub.docker.com/_/python) 3.9.6. We then set a working directory along with two environment variables:

1.  `PYTHONDONTWRITEBYTECODE`: Prevents Python from writing pyc files to disc (equivalent to `python -B` [option](https://docs.python.org/3/using/cmdline.html#id1))
2.  `PYTHONUNBUFFERED`: Prevents Python from buffering stdout and stderr (equivalent to `python -u` [option](https://docs.python.org/3/using/cmdline.html#cmdoption-u))

- We then installed the dependencies and copied over the app.

> Depending on your environment, you may need to add `RUN mkdir -p /usr/src/app` just before you set the working directory:
>
	``` bash
	> # set working directory
	> RUN mkdir -p /usr/src/app
	> WORKDIR /usr/src/app
	```

- Add a _.dockerignore_ file to the "project" directory as well:

```dockerfile
env
.dockerignore
Dockerfile
Dockerfile.prod
```

- Like the _.gitignore_ file, the [.dockerignore](https://docs.docker.com/engine/reference/builder/#dockerignore-file) file lets you exclude specific files and folders from being copied over to the image.

> Review [Docker for Python Developers](https://mherman.org/presentations/dockercon-2018) for more on structuring Dockerfiles as well as some best practices for configuring Docker for Python-based development.

- Then add a _docker-compose.yml_ file to the project root:

```yaml
version: '3.8'

services:
  web:
    build: ./project
    command: uvicorn app.main:app --reload --workers 1 --host 0.0.0.0 --port 8000
    volumes:
      - ./project:/usr/src/app
    ports:
      - 8004:8000
    environment:
      - ENVIRONMENT=dev
      - TESTING=0
```

- This config will create a service called `web` from the Dockerfile.

- So, when the container spins up, Uvicorn will run with the following [settings](https://www.uvicorn.org/settings/):
	-   `reload` enables auto reload so the server will restart after changes are made to the code base.
	-   `workers 1` provides a single worker process.
	-   `host 0.0.0.0` defines the address to host the server on.
	-   `port 8000` defines the port to host the server on.

- The `volume` is used to mount the code into the container. This is a must for a development environment in order to update the container whenever a change to the source code is made. Without this, you would have to re-build the image each time you make a change to the code.

- Take note of the [Docker compose file version](https://docs.docker.com/compose/compose-file/) used -- `3.8`. Keep in mind that this version does _not_ directly relate back to the version of Docker Compose installed; it simply specifies the file format that you want to use.

- Build the image:

`$ docker-compose build` 

- This will take a few minutes the first time. Subsequent builds will be much faster since Docker caches the results. If you'd like to learn more about Docker caching, review the [Order Dockerfile commands](https://mherman.org/presentations/dockercon-2018/#46) slide.

- Once the build is done, fire up the container in [detached mode](https://docs.docker.com/engine/reference/run/#detached--d):

`$ docker-compose up -d` 

- Navigate to [http://localhost:8004/ping](http://localhost:8004/ping). Make sure you see the same JSON response as before:

```json
{
  "ping": "pong!",
  "environment": "dev",
  "testing": false
}
```

> If you run into problems with the volume mounting correctly, you may want to remove it altogether by deleting the volume config from the Docker Compose file. You can still go through the course without it; you'll just have to re-build the image after you make changes to the source code.
> 
> _Windows Users_: Having problems getting the volume to work properly? Review the following resources:
> 
> 1.  [Docker on Windows — Mounting Host Directories](https://rominirani.com/docker-on-windows-mounting-host-directories-d96f3f056a2c)
> 2.  [Configuring Docker for Windows Shared Drives](https://blogs.msdn.microsoft.com/stevelasker/2016/06/14/configuring-docker-for-windows-volumes/)
> 
> You also may need to add `COMPOSE_CONVERT_WINDOWS_PATHS=1` to the `environment` portion of your Docker Compose file. Review [Declare default environment variables in file](https://docs.docker.com/compose/env-file/) for more info.

- You should now have:
	```
	- .gitignore
	- docker-compose.yml
	- project
		- .dockerignore
		- Dockerfile
		- app
			- __init__.py
			- config.py
			- main.py
		- requirements.txt
	```
	

## Postgres Setup

### Postgres

- To configure Postgres, we'll need to add a new service to the _docker-compose.yml_ file, add the appropriate environment variables, and install [asyncpg](https://github.com/MagicStack/asyncpg).

- Add a "db" directory to "project", and add a _create.sql_ file in that new directory:

```bash
CREATE DATABASE web_dev;
CREATE DATABASE web_test;
```

- Next, add a _Dockerfile_ to the same directory:

```dockerfile
# pull official base image
FROM postgres:13-alpine

# run create.sql on init
ADD create.sql /docker-entrypoint-initdb.d
```

- Here, we extend the [official Postgres image](https://hub.docker.com/_/postgres/) (an [Alpine](https://alpinelinux.org/)-based image) by adding _create.sql_ to the "docker-entrypoint-initdb.d" directory in the container. This file will execute on init.

- Next, add a new service called `web-db` to _docker-compose.yml_:

```dockerfile
version: '3.8'

services:

  web:
    build: ./project
    command: uvicorn app.main:app --reload --workers 1 --host 0.0.0.0 --port 8000
    volumes:
      - ./project:/usr/src/app
    ports:
      - 8004:8000
    environment:
      - ENVIRONMENT=dev
      - TESTING=0
      - DATABASE_URL=postgres://postgres:postgres@web-db:5432/web_dev        # new
      - DATABASE_TEST_URL=postgres://postgres:postgres@web-db:5432/web_test  # new
    depends_on:   # new
      - web-db

  # new
  web-db:
    build:
      context: ./project/db
      dockerfile: Dockerfile
    expose:
      - 5432
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
```

- Once spun up, Postgres will be available on port `5432` for services running in other containers. Since the `web` service is dependent not only on the container being up and running but also the actual Postgres instance being up and healthy, let's add an _entrypoint.sh_ file to the "project" directory:

```bash
#!/bin/sh

echo "Waiting for postgres..."

while ! nc -z web-db 5432; do
  sleep 0.1
done

echo "PostgreSQL started"

exec "$@"
```

- So, we referenced the Postgres container using the name of the service, `web-db`. The loop continues until something like `Connection to web-db port 5432 [tcp/postgresql] succeeded!` is returned.

- Update _Dockerfile_ to install the appropriate packages and supply an [entrypoint](https://docs.docker.com/engine/reference/builder/#entrypoint):

```dockerfile
# pull official base image
FROM python:3.9.6-slim-buster

# set working directory
WORKDIR /usr/src/app

# set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# install system dependencies
RUN apt-get update \
  && apt-get -y install netcat gcc postgresql \
  && apt-get clean

# install python dependencies
RUN pip install --upgrade pip
COPY ./requirements.txt .
RUN pip install -r requirements.txt

# add app
COPY . .

# add entrypoint.sh
COPY ./entrypoint.sh .
RUN chmod +x /usr/src/app/entrypoint.sh

# run entrypoint.sh
ENTRYPOINT ["/usr/src/app/entrypoint.sh"]
```

- Add asyncpg to _project/requirements.txt_:

`asyncpg==0.23.0` 

- Next, update _config.py_ to read the `DATABASE_URL` and assign the value to `database_url`:

```python
# project/app/config.py

import logging
import os
from functools import lru_cache

from pydantic import BaseSettings, AnyUrl

log = logging.getLogger("uvicorn")

class Settings(BaseSettings):
    environment: str = os.getenv("ENVIRONMENT", "dev")
    testing: bool = os.getenv("TESTING", 0)
    database_url: AnyUrl = os.environ.get("DATABASE_URL")

@lru_cache()
def get_settings() -> BaseSettings:
    log.info("Loading config settings from the environment...")
    return Settings()
```

- Take note of the [AnyUrl](https://pydantic-docs.helpmanual.io/usage/types/#urls) validator.

- Build the new image and spin up the two containers:

```bash
$ chmod +x project/entrypoint.sh
$ docker-compose up -d --build
```

- Once done, check the logs for the `web` service:

`$ docker-compose logs web` 

- You should see:

```markdown
Attaching to fastapi-tdd-docker_web_1
web_1     | Waiting for postgres...
web_1     | PostgreSQL started
web_1     | INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
web_1     | INFO:     Started reloader process [1] using statreload
web_1     | INFO:     Started server process [8]
web_1     | INFO:     Waiting for application startup.
web_1     | INFO:     Application startup complete.
```

> Depending on your environment, you may need to [chmod 755 or 777 instead of +x](https://github.com/testdrivenio/testdriven-app-2.3/issues/17#issuecomment-405422949). If you still get a "permission denied", review the [docker entrypoint running bash script gets "permission denied"](https://stackoverflow.com/questions/38882654/docker-entrypoint-running-bash-script-gets-permission-denied) Stack Overflow question.

- Want to access the database via psql?

`$ docker-compose exec web-db psql -U postgres` 

- Then, you can connect to the database:

```markdown
postgres=# \c web_dev
postgres=# \q
```


### Tortoise ORM

- To simplify interactions with the database, we'll using an async ORM called [Tortoise](https://tortoise-orm.readthedocs.io/).

- Add the dependency to the _requirements.txt_ file:
	- `tortoise-orm==0.17.4` 

- Next, create a new folder called "models" within "project/app". Add two files to "models": ___init__.py_ and _tortoise.py_.

- In _tortoise.py_, add:

```python
# project/app/models/tortoise.py

from tortoise import fields, models

class TextSummary(models.Model):
    url = fields.TextField()
    summary = fields.TextField()
    created_at = fields.DatetimeField(auto_now_add=True)

    def __str__(self):
        return self.url
```

- Here, we defined a new database [model](https://tortoise-orm.readthedocs.io/en/latest/models.html) called `TextSummary`.

- Add the [register_tortoise](https://tortoise-orm.readthedocs.io/en/latest/contrib/fastapi.html#tortoise.contrib.fastapi.register_tortoise) helper to _main.py_ to set up Tortoise on startup and clean up on teardown:

```python
# project/app/main.py

import os

from fastapi import FastAPI, Depends
from tortoise.contrib.fastapi import register_tortoise

from app.config import get_settings, Settings

app = FastAPI()

register_tortoise(
    app,
    db_url=os.environ.get("DATABASE_URL"),
    modules={"models": ["app.models.tortoise"]},
    generate_schemas=True,
    add_exception_handlers=True,
)

@app.get("/ping")
async def pong(settings: Settings = Depends(get_settings)):
    return {
        "ping": "pong!",
        "environment": settings.environment,
        "testing": settings.testing
    }
```

- Sanity check:
	- `$ docker-compose up -d --build` 

- Ensure the `textsummary` table was created:

```markdown
$ docker-compose exec web-db psql -U postgres

psql (13.3)
Type "help" for help.

postgres=# \c web_dev
You are now connected to database "web_dev" as user "postgres".

web_dev=# \dt
            List of relations
 Schema |    Name     | Type  |  Owner
--------+-------------+-------+----------
 public | textsummary | table | postgres
(1 row)

web_dev=# \q
```

- [http://localhost:8004/ping](http://localhost:8004/ping) should still work as well:

```json
{
  "ping": "pong!",
  "environment": "dev",
  "testing": false
}
```

- You should now have:

```markdown
|── .gitignore
├── docker-compose.yml
└── project
    ├── .dockerignore
    ├── Dockerfile
    ├── app
    │   ├── __init__.py
    │   ├── config.py
    │   ├── main.py
    │   └── models
    │       ├── __init__.py
    │       └── tortoise.py
    ├── db
    │   ├── Dockerfile
    │   └── create.sql
    ├── entrypoint.sh
    └── requirements.txt
```


### Migrations

- Tortoise supports [database migrations](https://tortoise-orm.readthedocs.io/en/latest/migration.html) via [Aerich](https://github.com/tortoise/aerich). Let's take a few steps back and configure it.

- First, bring down the containers and volumes to destroy the current database table since we want Aerich to manage the schema:
	- `$ docker-compose down -v` 

- Next, update the `register_tortoise` helper in _project/app/main.py_ so that the schemas are not automatically generated:

```python
register_tortoise(
    app,
    db_url=os.environ.get("DATABASE_URL"),
    modules={"models": ["app.models.tortoise"]},
    generate_schemas=False,  # updated
    add_exception_handlers=True,
)
```

- Spin the containers and volumes back up:
	- `$ docker-compose up -d --build` 

- Ensure the `textsummary` table was NOT created:

```markdown
$ docker-compose exec web-db psql -U postgres

psql (13.3)
Type "help" for help.

postgres=# \c web_dev
You are now connected to database "web_dev" as user "postgres".

web_dev=# \dt
Did not find any relations.

web_dev=# \q
```

- Add Aerich to the requirements file:
	- `aerich==0.5.3` 

- Then, update the containers:
	- `$ docker-compose up -d --build` 

- Aerich requires the Tortoise config, so let's add that to a new file called _project/app/db.py_:

```python
# project/app/db.py

import os

TORTOISE_ORM = {
    "connections": {"default": os.environ.get("DATABASE_URL")},
    "apps": {
        "models": {
            "models": ["app.models.tortoise", "aerich.models"],
            "default_connection": "default",
        },
    },
}
```

- Now we're ready to start using Aerich.

- Init:
	- `$ docker-compose exec web aerich init -t app.db.TORTOISE_ORM` 

- This will create a config file called _project/aerich.ini_:

```mardown
[aerich]
tortoise_orm = app.db.TORTOISE_ORM
location = ./migrations
```

- Create the first migration:

```markdown
$ docker-compose exec web aerich init-db

Success create app migrate location migrations/models
Success generate schema for app "models"
```

- You should see the following migration inside a new file within "migrations/models":

```sql
-- upgrade --
CREATE TABLE IF NOT EXISTS "textsummary" (
    "id" SERIAL NOT NULL PRIMARY KEY,
    "url" TEXT NOT NULL,
    "summary" TEXT NOT NULL,
    "created_at" TIMESTAMPTZ NOT NULL  DEFAULT CURRENT_TIMESTAMP
);
CREATE TABLE IF NOT EXISTS "aerich" (
    "id" SERIAL NOT NULL PRIMARY KEY,
    "version" VARCHAR(255) NOT NULL,
    "app" VARCHAR(20) NOT NULL,
    "content" JSONB NOT NULL
);
```

- That's it! You should now be able to see the two tables:

```
$ docker-compose exec web-db psql -U postgres

psql (13.3)
Type "help" for help.

postgres=# \c web_dev
You are now connected to database "web_dev" as user "postgres".

web_dev=# \dt
            List of relations
 Schema |    Name     | Type  |  Owner
--------+-------------+-------+----------
 public | aerich      | table | postgres
 public | textsummary | table | postgres
(2 rows)

web_dev=# \q
```

- Be sure to review the [documentation](https://tortoise-orm.readthedocs.io/en/latest/migration.html) to learn more about the available commands. Take note of the `downgrade`, `history`, and `upgrade` commands.

## Pytest Setup

### Setup

- Add a "tests" directory to the "project" directory, and then create the following files inside the newly created directory:

1.  ___init__.py_
2.  _conftest.py_
3.  _test_ping.py_

- By default, pytest will auto-discover test files that start or end with `test` -- e.g., `test_*.py` or `*_test.py`. Test functions must begin with `test_`, and if you want to use classes they must also begin with `Test`.

- Example:

```python
# if a class is used, it must begin with Test
class TestFoo:

    # test functions must begin with test_
    def test_bar(self):
        assert "foo" != "bar"
```

> If this is your first time with pytest be sure to review the [Installation and Getting Started](https://docs.pytest.org/en/latest/getting-started.html) guide.


### Fixtures

- Define a `test_app` [fixture](https://docs.pytest.org/en/latest/how-to/fixtures.html) in _conftest.py_:

```python
# project/tests/conftest.py

import os

import pytest
from starlette.testclient import TestClient

from app import main
from app.config import get_settings, Settings

def get_settings_override():
    return Settings(testing=1, database_url=os.environ.get("DATABASE_TEST_URL"))

@pytest.fixture(scope="module")
def test_app():
    # set up
    main.app.dependency_overrides[get_settings] = get_settings_override
    with TestClient(main.app) as test_client:

        # testing
        yield test_client

    # tear down
```

- Here, we imported Starlette's [TestClient](https://www.starlette.io/testclient/), which uses the [Requests](https://requests.readthedocs.io/) library to make requests against the FastAPI app.

- To override the dependencies, we used the [dependency_overrides](https://fastapi.tiangolo.com/advanced/testing-dependencies/#use-the-appdependency_overrides-attribute) attribute:
	- `main.app.dependency_overrides[get_settings] = get_settings_override` 

- `dependency_overrides` is a dict of key/value pairs where the key is the dependency name and the value is what we'd like to override it with:
	-   key: `get_settings`
	-   value: `get_settings_override`

- Fixtures are reusable objects for tests. They have a [scope](https://docs.pytest.org/en/stable/fixture.html#scope-sharing-fixtures-across-classes-modules-packages-or-session) associated with them, which indicates how often the fixture is invoked:

1.  function - once per test function (default)
2.  class - once per test class
3.  module - once per test module
4.  session - once per test session

> Check out [All You Need to Know to Start Using Fixtures in Your pytest Code](https://pybit.es/pytest-fixtures.html) for more on pytest fixtures.

- Take note of the `test_app` fixture. In essence, all code before the `yield` statement serves as setup code while everything after serves as the teardown.

> For more on this review [Fixture finalization / executing teardown code](https://docs.pytest.org/en/latest/explanation/fixtures.html#improvements-over-xunit-style-setup-teardown-functions).

- Then, add pytest and Requests to the requirements file:

```txt
pytest==6.2.4
requests==2.25.1
```

- We need to re-build the Docker images since requirements are installed at build time rather than run time:
	- `$ docker-compose up -d --build` 

- With the containers up and running, run the tests:
	- `$ docker-compose exec web python -m pytest` 

You should see:

```
=============================== test session starts ===============================
platform linux -- Python 3.9.6, pytest-6.2.4, py-1.10.0, pluggy-0.13.1
rootdir: /usr/src/app
collected 0 items

============================== no tests ran in 0.07s ==============================
```


### Tests

- Moving on, let's add a quick test to _test_ping.py_:

```python
# project/tests/test_ping.py

from app import main

def test_ping(test_app):
    response = test_app.get("/ping")
    assert response.status_code == 200
    assert response.json() == {"environment": "dev", "ping": "pong!", "testing": True}
```

- While unittest requires test classes, pytest just requires functions to get up and running. In other words, pytest tests are just functions that either start or end with `test`.

> You can still use classes to organize tests in pytest if that's your preferred pattern.

- To use the fixture, we passed it in as an argument.

- Run the tests again:
	- `$ docker-compose exec web python -m pytest` 

- You should see:

```
=============================== test session starts ===============================
platform linux -- Python 3.9.6, pytest-6.2.4, py-1.10.0, pluggy-0.13.1
rootdir: /usr/src/app
collected 1 item

tests/test_ping.py .                                                        [100%]

================================ 1 passed in 0.13s ================================
```


### Given-When-Then

- When writing tests, try to follow the [Given-When-Then](https://martinfowler.com/bliki/GivenWhenThen.html) framework to help make the process of writing tests easier and faster. It also helps communicate the purpose of your tests better so it should be easier to read by your future self and others.

| **State** | **Explanation**                                   | **Code**                             |
| --------- | ------------------------------------------------- | ------------------------------------ |
| Given     | the state of the application before the test runs | setup code, fixtures, database state |
| When      | the behavior/logic being tested                   | code under test                      |
| Then      | the expected changes based on the behavior        | asserts                              | 

Example:

```python
def test_ping(test_app):
    # Given
    # test_app

    # When
    response = test_app.get("/ping")

    # Then
    assert response.status_code == 200
    assert response.json() == {"environment": "dev", "ping": "pong!", "testing": True}
```

## App Structure

With tests in place, let's refactor the app, adding in FastAPI's `APIRouter`, a new database init function, and a Pydantic `Model`.

---

### APIRouter

- First, add a new folder called "api" to the "app" folder. Add an ___init__.py_ file to the newly created folder.

- Now we can move the `/ping` route to a new file called _project/app/api/ping.py_:

```python
# project/app/api/ping.py

from fastapi import APIRouter, Depends

from app.config import get_settings, Settings

router = APIRouter()

@router.get("/ping")
async def pong(settings: Settings = Depends(get_settings)):
    return {
        "ping": "pong!",
        "environment": settings.environment,
        "testing": settings.testing
    }
```

- Then, update _main.py_ like so to remove the old route, wire the router up to our main app, and use a function to initialize a new app:

```python
# project/app/main.py

import os

from fastapi import FastAPI
from tortoise.contrib.fastapi import register_tortoise

from app.api import ping

def create_application() -> FastAPI:
    application = FastAPI()

    register_tortoise(
        application,
        db_url=os.environ.get("DATABASE_URL"),
        modules={"models": ["app.models.tortoise"]},
        generate_schemas=False,
        add_exception_handlers=True,
    )

    application.include_router(ping.router)

    return application

app = create_application()
```

- Make sure [http://localhost:8004/ping](http://localhost:8004/ping) and [http://localhost:8004/docs](http://localhost:8004/docs) still work.

- Update the `test_app` fixture in _project/tests/conftest.py_ to use the `create_application` function to create a new instance of FastAPI for testing purposes:

```python
# project/tests/conftest.py

import os

import pytest
from starlette.testclient import TestClient

from app.main import create_application  # updated
from app.config import get_settings, Settings

def get_settings_override():
    return Settings(testing=1, database_url=os.environ.get("DATABASE_TEST_URL"))

@pytest.fixture(scope="module")
def test_app():
    # set up
    app = create_application()  # new
    app.dependency_overrides[get_settings] = get_settings_override
    with TestClient(app) as test_client:  # updated

        # testing
        yield test_client

    # tear down
```

- Make sure the tests still pass:

```
$ docker-compose exec web python -m pytest

=============================== test session starts ===============================
platform linux -- Python 3.9.6, pytest-6.2.4, py-1.10.0, pluggy-0.13.1
rootdir: /usr/src/app
collected 1 item

tests/test_ping.py .                                                        [100%]

================================ 1 passed in 0.16s ================================
```

> You can break up and modularize larger projects as well as apply versioning to your API with the `APIRouter`. If you're familiar with Flask, it's equivalent to a [Blueprint](https://flask.palletsprojects.com/blueprints/).

- Your project structure should now look like:

```markdown
├── .gitignore
├── docker-compose.yml
└── project
    ├── .dockerignore
    ├── Dockerfile
    ├── aerich.ini
    ├── app
    │   ├── __init__.py
    │   ├── api
    │   │   ├── __init__.py
    │   │   └── ping.py
    │   ├── config.py
    │   ├── db.py
    │   ├── main.py
    │   └── models
    │       ├── __init__.py
    │       └── tortoise.py
    ├── db
    │   ├── Dockerfile
    │   └── create.sql
    ├── entrypoint.sh
    ├── migrations
    │   └── models
    │       └── 1_20210704142846_None.sql
    ├── requirements.txt
    └── tests
        ├── __init__.py
        ├── conftest.py
        └── test_ping.py`
```


### Database Init

- Next, let's move the `register_tortoise` helper to _project/app/db.py_ to clean up _project/app/main.py_:

```python
# project/app/db.py

import os

from fastapi import FastAPI
from tortoise.contrib.fastapi import register_tortoise

TORTOISE_ORM = {
    "connections": {"default": os.environ.get("DATABASE_URL")},
    "apps": {
        "models": {
            "models": ["app.models.tortoise", "aerich.models"],
            "default_connection": "default",
        },
    },
}

def init_db(app: FastAPI) -> None:
    register_tortoise(
        app,
        db_url=os.environ.get("DATABASE_URL"),
        modules={"models": ["app.models.tortoise"]},
        generate_schemas=False,
        add_exception_handlers=True,
    )
```

- Import `init_db` into _main.py_:

```python
# project/app/main.py

import logging

from fastapi import FastAPI

from app.api import ping
from app.db import init_db

log = logging.getLogger("uvicorn")

def create_application() -> FastAPI:
    application = FastAPI()
    application.include_router(ping.router)

    return application

app = create_application()

@app.on_event("startup")
async def startup_event():
    log.info("Starting up...")
    init_db(app)

@app.on_event("shutdown")
async def shutdown_event():
    log.info("Shutting down...")
```

- Here, we added `startup` and `shutdown` [event handlers](https://fastapi.tiangolo.com/advanced/events/), which get executed before the app starts up or when the app is shutting down, respectively.

- To test, first bring down the containers and volumes:
	- `$ docker-compose down -v` 

- Then, bring the containers back up:
	- `$ docker-compose up -d --build` 

- Since we haven't applied the migrations, you shouldn't see any tables in the database:

```
$ docker-compose exec web-db psql -U postgres

psql (13.3)
Type "help" for help.

postgres=# \c web_dev
You are now connected to database "web_dev" as user "postgres".

web_dev=# \dt
Did not find any relations.

web_dev=# \q
```

- Rather than applying the migrations via Aerich, which can be slow, there may be times where you just want to apply the schema to the database in its final state. So, let's add a `generate_schema` function to _db.py_ for handling that:

```python
# project/app/db.py

import logging  # new
import os

from fastapi import FastAPI
from tortoise import Tortoise, run_async  # new
from tortoise.contrib.fastapi import register_tortoise

log = logging.getLogger("uvicorn") # new

TORTOISE_ORM = {
    "connections": {"default": os.environ.get("DATABASE_URL")},
    "apps": {
        "models": {
            "models": ["app.models.tortoise", "aerich.models"],
            "default_connection": "default",
        },
    },
}

def init_db(app: FastAPI) -> None:
    register_tortoise(
        app,
        db_url=os.environ.get("DATABASE_URL"),
        modules={"models": ["app.models.tortoise"]},
        generate_schemas=False,
        add_exception_handlers=True,
    )

# new
async def generate_schema() -> None:
    log.info("Initializing Tortoise...")

    await Tortoise.init(
        db_url=os.environ.get("DATABASE_URL"),
        modules={"models": ["models.tortoise"]},
    )
    log.info("Generating database schema via Tortoise...")
    await Tortoise.generate_schemas()
    await Tortoise.close_connections()

# new
if __name__ == "__main__":
    run_async(generate_schema())
```

- So, `generate_schema` calls [Tortoise.init](https://tortoise-orm.readthedocs.io/en/latest/setup.html?highlight=init#tortoise.Tortoise.init) to set up Tortoise and then [generates](https://tortoise-orm.readthedocs.io/en/latest/setup.html?highlight=init#tortoise.Tortoise.generate_schemas) the schema.

- Run:
	- `$ docker-compose exec web python app/db.py` 

- You should now see the schema:

```
$ docker-compose exec web-db psql -U postgres

psql (13.3)
Type "help" for help.

postgres=# \c web_dev
You are now connected to database "web_dev" as user "postgres".

web_dev=# \dt
            List of relations
 Schema |    Name     | Type  |  Owner
--------+-------------+-------+----------
 public | textsummary | table | postgres
(1 row)

web_dev=# \q
```

- Finally, since we do want to use Aerich in dev to manage the database schema, bring the containers and volumes down again:
	- `$ docker-compose down -v` 

- bring the containers back up:
	- `$ docker-compose up -d --build` 

- Confirm that the table doesn't exist in the database. Then, apply the migration:
	- `$ docker-compose exec web aerich upgrade` 

- Confirm that the `aerich` and `textsummary` tables now exist.


### Pydantic Model

> First time using Pydantic? Review the [Overview](https://pydantic-docs.helpmanual.io/) guide from the official docs.

- Create a `SummaryPayloadSchema` Pydantic [model](https://pydantic-docs.helpmanual.io/usage/models/) with one required field -- a `url` -- in a new file called _pydantic.py_ in "project/app/models":

```python
# project/app/models/pydantic.py

from pydantic import BaseModel

class SummaryPayloadSchema(BaseModel):
    url: str
```


## RESTful Routes

- Next, let’s set up three new routes, RESTful best practices, with TDD:

| **Endpoint**   | **HTTP Method** | **CRUD Method** | **Result**           |
| -------------- | --------------- | --------------- | -------------------- |
| /summaries     | GET             | READ            | get all summaries    |
| /summaries/:id | GET             | READ            | get a single summary |
| /summaries     | POST            | CREATE          | add a summary        |

- For each, we’ll:
	1. write a test
	2. run the test, to ensure it fails (**red**)
	3. write just enough code to get the test to pass (**green**)
	4. **refactor** (if necessary)

- Let’s start with POST route.

### POST ROUTE

- We'll break from the normal TDD flow for this first route in order to establish the coding pattern that we'll use for the remaining routes.

#### Code

- Create a new file called *summaries.py* in the “project/app/api” folder:

```python
# project/app/api/summaries.py


from fastapi import APIRouter, HTTPException

from app.api import crud
from app.models.pydantic import SummaryPayloadSchema, SummaryResponseSchema


router = APIRouter()


@router.post("/", response_model=SummaryResponseSchema, status_code=201)
async def create_summary(payload: SummaryPayloadSchema) -> SummaryResponseSchema:
    summary_id = await crud.post(payload)

    response_object = {
        "id": summary_id,
        "url": payload.url
    }
    return response_object
```

- Here, we defined a handler that expects a payload, `payload: SummaryPayloadSchema`, with a URL.

- Essentially, when the route is hit with a POST request, FastAPI will read the body of the request and validate the data:
	-   If valid, the data will be available in the payload `parameter`. FastAPI also generates [JSON Schema](https://json-schema.org/) definitions that are then used to automatically generate the OpenAPI schema and the API documentation.
	-   If invalid, an error is immediately returned.

> Review the [Request Body](https://fastapi.tiangolo.com/tutorial/body/) docs for more info.

- It's worth noting that we used the `async` declaration here since the database communication will be asynchronous. In other words, there are no blocking I/O operations in the handler.

- Next, create a new file called _crud.py_ in the "project/app/api" folder:

```python
# project/app/api/crud.py

from app.models.pydantic import SummaryPayloadSchema
from app.models.tortoise import TextSummary

async def post(payload: SummaryPayloadSchema) -> int:
    summary = TextSummary(
        url=payload.url,
        summary="dummy summary",
    )
    await summary.save()
    return summary.id
```

- We added a utility function called `post` for creating new summaries that takes a payload object and then:
1.  [Creates](https://tortoise-orm.readthedocs.io/en/latest/models.html?highlight=save()#tortoise.models.Model.save) a new `TextSummary` instance
2.  Returns the generated ID

- Next, we need to define a new Pydantic model for use as the [response_model](https://fastapi.tiangolo.com/tutorial/response-model/):
	- `@router.post("/", response_model=SummaryResponseSchema, status_code=201)` 

- Update _pydantic.py_ like so:

```python
# project/app/models/pydantic.py

from pydantic import BaseModel

class SummaryPayloadSchema(BaseModel):
    url: str

class SummaryResponseSchema(SummaryPayloadSchema):
    id: int
```

- The `SummaryResponseSchema` model inherits from the `SummaryPayloadSchema` model, adding an `id` field.

- Wire up the new router in _main.py_:

```python
# project/app/main.py

import logging

from fastapi import FastAPI

from app.api import ping, summaries  # updated
from app.db import init_db

log = logging.getLogger("uvicorn")

def create_application() -> FastAPI:
    application = FastAPI()
    application.include_router(ping.router)
    application.include_router(summaries.router, prefix="/summaries", tags=["summaries"])  # new

    return application

app = create_application()

@app.on_event("startup")
async def startup_event():
    log.info("Starting up...")
    init_db(app)

@app.on_event("shutdown")
async def shutdown_event():
    log.info("Shutting down...")
```

- Take note of the prefix URL along with the "summaries" [tag](https://fastapi.tiangolo.com/tutorial/path-operation-configuration/#tags), which will be applied to the OpenAPI schema (for [grouping operations](https://swagger.io/docs/specification/grouping-operations-with-tags/)).

- Test it out with curl or [HTTPie](https://httpie.org/):
	- `$ http --json POST http://localhost:8004/summaries/ url=http://testdriven.io` 

- You should see:

```
HTTP/1.1 201 Created
content-length: 37
content-type: application/json
date: Mon, 05 Jul 2021 18:49:42 GMT
server: uvicorn

{
    "id": 1,
    "url": "http://testdriven.io"
}
```

- You can also interact with the endpoint at [http://localhost:8004/docs](http://localhost:8004/docs).


#### Test

- First, add a new file called _test_summaries.py_ to "project/tests". Then, add the following test:

```python
# project/tests/test_summaries.py

import json

import pytest

def test_create_summary(test_app_with_db):
    response = test_app_with_db.post("/summaries/", data=json.dumps({"url": "https://foo.bar"}))

    assert response.status_code == 201
    assert response.json()["url"] == "https://foo.bar"
```

- Add the new fixture to _conftest.py_:

```python
import os

import pytest
from starlette.testclient import TestClient

from app.main import create_application
from app.config import get_settings, Settings

def get_settings_override():
    return Settings(testing=1, database_url=os.environ.get("DATABASE_TEST_URL"))

@pytest.fixture(scope="module")
def test_app():
    # set up
    app = create_application()
    app.dependency_overrides[get_settings] = get_settings_override
    with TestClient(app) as test_client:

        # testing
        yield test_client

    # tear down

# new
@pytest.fixture(scope="module")
def test_app_with_db():
    # set up
    app = create_application()
    app.dependency_overrides[get_settings] = get_settings_override
    register_tortoise(
        app,
        db_url=os.environ.get("DATABASE_TEST_URL"),
        modules={"models": ["app.models.tortoise"]},
        generate_schemas=True,
        add_exception_handlers=True,
    )
    with TestClient(app) as test_client:

        # testing
        yield test_client

    # tear down
```

- Add the import as well:
	- `from tortoise.contrib.fastapi import register_tortoise` 

- Run the tests:

```
$ docker-compose exec web python -m pytest

=============================== test session starts ===============================
platform linux -- Python 3.9.6, pytest-6.2.4, py-1.10.0, pluggy-0.13.1
rootdir: /usr/src/app
collected 2 items

tests/test_ping.py .                                                        [ 50%]
tests/test_summaries.py .                                                   [100%]

================================ 2 passed in 0.19s ================================
```

> Did a warning get outputted? Pass `-p no:warnings` on the command-line in to [disable](https://docs.pytest.org/en/latest/how-to/capture-warnings.html#disabling-warnings-summary) them.

- This test covers the happy path. What about the exception path?

- Add a new test:

```python
# project/tests/test_summaries.py

import json

import pytest

def test_create_summary(test_app_with_db):
    response = test_app_with_db.post("/summaries/", data=json.dumps({"url": "https://foo.bar"}))

    assert response.status_code == 201
    assert response.json()["url"] == "https://foo.bar"

def test_create_summaries_invalid_json(test_app):
    response = test_app.post("/summaries/", data=json.dumps({}))
    assert response.status_code == 422
    assert response.json() == {
        "detail": [
            {
                "loc": ["body", "url"],
                "msg": "field required",
                "type": "value_error.missing"
            }
        ]
    }
```

It should pass.

With that, we can configure the remaining CRUD routes using Test-Driven Development.

```
├── .gitignore
├── docker-compose.yml
└── project
    ├── .dockerignore
    ├── Dockerfile
    ├── aerich.ini
    ├── app
    │   ├── __init__.py
    │   ├── api
    │   │   ├── __init__.py
    │   │   ├── crud.py
    │   │   ├── ping.py
    │   │   └── summaries.py
    │   ├── config.py
    │   ├── db.py
    │   ├── main.py
    │   └── models
    │       ├── __init__.py
    │       ├── pydantic.py
    │       └── tortoise.py
    ├── db
    │   ├── Dockerfile
    │   └── create.sql
    ├── entrypoint.sh
    ├── migrations
    │   └── models
    │       └── 1_20210704142846_None.sql
    ├── requirements.txt
    └── tests
        ├── __init__.py
        ├── conftest.py
        ├── test_ping.py
        └── test_summaries.py
```

### GET single summary Route

- Start with a test:

```python
def test_read_summary(test_app_with_db):
    response = test_app_with_db.post("/summaries/", data=json.dumps({"url": "https://foo.bar"}))
    summary_id = response.json()["id"]

    response = test_app_with_db.get(f"/summaries/{summary_id}/")
    assert response.status_code == 200

    response_dict = response.json()
    assert response_dict["id"] == summary_id
    assert response_dict["url"] == "https://foo.bar"
    assert response_dict["summary"]
    assert response_dict["created_at"]
```

- Ensure the test fails:

```
>       assert response.status_code == 200
E       assert 404 == 200
E        +  where 404 = <Response [404]>.status_code
```

- Add the following handler:

```python
@router.get("/{id}/", response_model=SummarySchema)
async def read_summary(id: int) -> SummarySchema:
    summary = await crud.get(id)

    return summary
```

- Here, instead of taking a payload, the handler requires an `id`, an integer, which will come from the path -- i.e., /summaries/1/.

- Add the `get` utility function to _crud.py_:

```python
async def get(id: int) -> Union[dict, None]:
    summary = await TextSummary.filter(id=id).first().values()
    if summary:
        return summary[0]
    return None
```

- Add the import:
	- `from typing import Union` 

- Here, we used the [values](https://tortoise-orm.readthedocs.io/en/latest/query.html?highlight=values#tortoise.queryset.QuerySet.values) method to create a [ValuesQuery](https://tortoise-orm.readthedocs.io/en/latest/query.html?highlight=values#tortoise.queryset.ValuesQuery) object. Then, if the `TextSummary` exists, we returned it as a dict.

> `Union[dict, None]` is [equivalent](https://docs.python.org/3/library/typing.html#typing.Optional) to `Optional[dict]`, so feel free to update this if you prefer `Optional` over `Union`.

- Next, update _project/app/models/tortoise.py_ to [generate](https://tortoise-orm.readthedocs.io/en/latest/contrib/pydantic.html?highlight=pydantic_model_creator#basic-usage) a Pydantic Model from the Tortoise Model:

```python
# project/app/models/tortoise.py

from tortoise import fields, models
from tortoise.contrib.pydantic import pydantic_model_creator  # new

class TextSummary(models.Model):
    url = fields.TextField()
    summary = fields.TextField()
    created_at = fields.DatetimeField(auto_now_add=True)

    def __str__(self):
        return self.url

SummarySchema = pydantic_model_creator(TextSummary)  # new
```

- Import `SummarySchema` into _project/app/api/summaries.py_:
	- `from app.models.tortoise import SummarySchema` 

- Ensure the tests pass:

```
$ docker-compose exec web python -m pytest

=============================== test session starts ===============================
platform linux -- Python 3.9.6, pytest-6.2.4, py-1.10.0, pluggy-0.13.1
rootdir: /usr/src/app
collected 4 items

tests/test_ping.py .                                                        [ 25%]
tests/test_summaries.py ...                                                 [100%]

================================ 4 passed in 0.20s ================================
```

- What if the summary ID doesn't exist?

- Add a new test:

```python
def test_read_summary_incorrect_id(test_app_with_db):
    response = test_app_with_db.get("/summaries/999/")
    assert response.status_code == 404
    assert response.json()["detail"] == "Summary not found"
```

- It should fail:

```
>       assert response.status_code == 404
E       assert 200 == 404
E        +  where 200 = <Response [200]>.status_code
```

- Update the code:

```python
@router.get("/{id}/", response_model=SummarySchema)
async def read_summary(id: int) -> SummarySchema:
    summary = await crud.get(id)
    if not summary:
        raise HTTPException(status_code=404, detail="Summary not found")

    return summary
```

- It should now pass:

```
=============================== test session starts ===============================
platform linux -- Python 3.9.6, pytest-6.2.4, py-1.10.0, pluggy-0.13.1
rootdir: /usr/src/app
collected 5 items

tests/test_ping.py .                                                        [ 20%]
tests/test_summaries.py ....                                                [100%]

================================ 5 passed in 0.21s ================================
```

- Before moving on, manually test the new endpoint in the browser, with curl or HTTPie, and/or via the API documentation.


#### GET all summaries Route

Again, let's start with a test:

```python
def test_read_all_summaries(test_app_with_db):
    response = test_app_with_db.post("/summaries/", data=json.dumps({"url": "https://foo.bar"}))
    summary_id = response.json()["id"]

    response = test_app_with_db.get("/summaries/")
    assert response.status_code == 200

    response_list = response.json()
    assert len(list(filter(lambda d: d["id"] == summary_id, response_list))) == 1
```

- Make sure it fails. Then add the handler:

```python
@router.get("/", response_model=List[SummarySchema])
async def read_all_summaries() -> List[SummarySchema]:
    return await crud.get_all()
```

- Import [List](https://docs.python.org/3/library/typing.html#typing.List) from Python's typing module:
	- `from typing import List` 

- The `response_model` is a `List` with a `SummarySchema` subtype.

- Add the CRUD util:

```python
async def get_all() -> List:
    summaries = await TextSummary.all().values()
    return summaries
```

- Update the import:
	- `from typing import Union, List` 

- Does the test past?

```
=============================== test session starts ===============================
platform linux -- Python 3.9.6, pytest-6.2.4, py-1.10.0, pluggy-0.13.1
rootdir: /usr/src/app
collected 6 items

tests/test_ping.py .                                                        [ 16%]
tests/test_summaries.py .....                                               [100%]

================================ 6 passed in 0.22s ================================
```

- Manually test this endpoint as well.


### Selecting Tests

You can [select specific tests](https://docs.pytest.org/en/latest/how-to/usage.html#specifying-tests-selecting-tests) to run using substring matching.

For example, to run all tests that have `ping` in their names:

`$ docker-compose exec web python -m pytest -k ping

=============================== test session starts ===============================
platform linux -- Python 3.9.6, pytest-6.2.4, py-1.10.0, pluggy-0.13.1
rootdir: /usr/src/app
collected 6 items / 5 deselected / 1 selected

tests/test_ping.py .                                                        [100%]

========================= 1 passed, 5 deselected in 0.14s =========================` 

So, you can see that the single test in _test_ping.py_ ran (and passed) while the five tests in _test_summaries.py_ were skipped.

Which test(s) will run when you run this command:

`$ docker-compose exec web python -m pytest -k read`


### Pytest Commands

Before moving on, let's review some useful pytest commands:

```
# normal run
$ docker-compose exec web python -m pytest

# disable warnings
$ docker-compose exec web python -m pytest -p no:warnings

# run only the last failed tests
$ docker-compose exec web python -m pytest --lf

# run only the tests with names that match the string expression
$ docker-compose exec web python -m pytest -k "summary and not test_read_summary"

# stop the test session after the first failure
$ docker-compose exec web python -m pytest -x

# enter PDB after first failure then end the test session
$ docker-compose exec web python -m pytest -x --pdb

# stop the test run after two failures
$ docker-compose exec web python -m pytest --maxfail=2

# show local variables in tracebacks
$ docker-compose exec web python -m pytest -l

# list the 2 slowest tests
$ docker-compose exec web python -m pytest --durations=2
```

## Deployment

- With the routes up and tested, let's get this app deployed to Heroku!

### Gunicorn

- To use [Gunicorn](https://gunicorn.org/), a production-grade WSGI server, first add the dependency to the _requirements.txt_ file:
	- `gunicorn==20.1.0` 

> Why Gunicorn and Uvicorn? To get the best of concurrency and parallelism. Gunicorn manages multiple, concurrent Uvicorn processes.

- Add a new Dockerfile called _Dockerfile.prod_:

```dockerfile
# pull official base image
FROM python:3.8.11-slim-buster

# create directory for the app user
RUN mkdir -p /home/app

# create the app user
RUN addgroup --system app && adduser --system --group app

# create the appropriate directories
ENV HOME=/home/app
ENV APP_HOME=/home/app/web
RUN mkdir $APP_HOME
WORKDIR $APP_HOME

# set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1
ENV ENVIRONMENT prod
ENV TESTING 0

# install system dependencies
RUN apt-get update \
  && apt-get -y install netcat gcc postgresql \
  && apt-get clean

# install python dependencies
RUN pip install --upgrade pip
COPY ./requirements.txt .
RUN pip install -r requirements.txt
RUN pip install "uvicorn[standard]==0.14.0"

# add app
COPY . .

# chown all the files to the app user
RUN chown -R app:app $APP_HOME

# change to the app user
USER app

# run gunicorn
CMD gunicorn --bind 0.0.0.0:$PORT app.main:app -k uvicorn.workers.UvicornWorker
```

- What's different here from the original _Dockerfile_?

- First, we started with a Python 3.8.11 image rather than 3.9.6 since [uvloop](https://github.com/MagicStack/uvloop), which `uvicorn.workers.UvicornWorker` uses, [does not support Python 3.9 yet](https://github.com/MagicStack/uvloop/issues/365).

- Next, we added a `CMD` to run Gunicorn (with a [uvicorn worker class](https://www.uvicorn.org/#running-with-gunicorn)) and configured two new environment variables:

```bash
ENV ENVIRONMENT prod
ENV TESTING 0
```

- We also created and switched to a non-root user, which is [recommended by Heroku](https://devcenter.heroku.com/articles/container-registry-and-runtime#run-the-image-as-a-non-root-user).

- Finally, take note of the `$PORT` environment variable in the Gunicorn command. Essentially, our web app must be listening on a particular port specified by the `$PORT` environment variable, which is [supplied by Heroku](https://devcenter.heroku.com/articles/dynos#web-dynos).

### Heroku

- [Sign up](https://signup.heroku.com/) for a Heroku account (if you don’t already have one), and then install the [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli) (if you haven't already done so).

- Create a new app:

```
$ heroku create
Creating app... done, ⬢ thawing-beach-87010
https://thawing-beach-87010.herokuapp.com/ | https://git.heroku.com/thawing-beach-87010.git
```

- Log in to the [Heroku Container Registry](https://devcenter.heroku.com/articles/container-registry-and-runtime):
	- `$ heroku container:login` 

- Provision a new Postgres database with the [hobby-dev](https://devcenter.heroku.com/articles/heroku-postgres-plans#hobby-tier) plan:
	- `$ heroku addons:create heroku-postgresql:hobby-dev --app thawing-beach-87010` 

- Build the production image and tag it with the following format:
	- `registry.heroku.com/<app>/<process-type>` 

- Make sure to replace `<app>` with the name of the Heroku app that you just created and `<process-type>` with `web` since this will be a [web dyno](https://www.heroku.com/dynos).

- For example:
	- `$ docker build -f project/Dockerfile.prod -t registry.heroku.com/thawing-beach-87010/web ./project` 

- To test locally, spin up the container:
	- `$ docker run --name fastapi-tdd -e PORT=8765 -e DATABASE_URL=sqlite://sqlite.db -p 5003:8765 registry.heroku.com/thawing-beach-87010/web:latest` 

- You should see something similar to:

```
[2021-07-06 12:22:08 +0000] [8] [INFO]  Starting gunicorn 20.1.0
[2021-07-06 12:22:08 +0000] [8] [INFO]  Listening at: http://0.0.0.0:8765 (8)
[2021-07-06 12:22:08 +0000] [8] [INFO]  Using worker: uvicorn.workers.UvicornWorker
[2021-07-06 12:22:08 +0000] [10] [INFO] Booting worker with pid: 10
[2021-07-06 12:22:08 +0000] [10] [INFO] Started server process [10]
[2021-07-06 12:22:08 +0000] [10] [INFO] Waiting for application startup.
[2021-07-06 12:22:08 +0000] [10] [INFO] Application startup complete.
```

- Navigate to [http://localhost:5003/ping/](http://localhost:5003/ping/).

- You should see:

```json
{
  "ping": "pong!",
  "environment": "prod",
  "testing": false
}
```

- Bring down the container once done:
	- `$ docker rm fastapi-tdd -f` 

- Push the image to the registry:
	- `$ docker push registry.heroku.com/thawing-beach-87010/web:latest` 

- Release the image:
	- `$ heroku container:release web --app thawing-beach-87010` 

> Again, replace `thawing-beach-87010` in each of the above commands with the name of your app.

- This will run the container. You should be able to view the app at [https://APP_NAME.herokuapp.com/ping/](https://app_name.herokuapp.com/ping/).

- Apply the migrations:
	- `$ heroku run aerich upgrade --app thawing-beach-87010` 

- How about the `summaries` endpoints? Do they work?

| **Endpoint**   | **HTTP Method** | **CRUD Method** | **Result**         |
| -------------- | --------------- | --------------- | ------------------ |
| /summaries     | GET             | READ            | get all summaries  |
| /summaries/:id | GET             | READ            | get single summary |
| /summaries     | POST            | CREATE          | add a summary      | 


- Try adding a new summary with [HTTPie](https://httpie.org/):
	- `$ http --json POST https://thawing-beach-87010.herokuapp.com/summaries/ url=https://testdriven.io` 

- You should see:

```
HTTP/1.1 201 Created
Connection: keep-alive
Content-Length: 38
Content-Type: application/json
Date: Tue, 06 Jul 2021 12:30:01 GMT
Server: uvicorn
Via: 1.1 vegur

{
    "id": 1,
    "url": "https://testdriven.io"
}
```

- Ensure [https://APP_NAME.herokuapp.com/docs](https://app_name.herokuapp.com/docs) works as well.