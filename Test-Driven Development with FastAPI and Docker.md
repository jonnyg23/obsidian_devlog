# TDD with FastAPI & Docker

## Getting Started

### Setup

- Create a new project and install FastAPI along with [[Uvicorn]], an [[ASGI]] server used to serve up FastAPI: (You may need [[pyenv]] for more details on python environment setup)

```bash
$ mkdir fastapi-tdd-docker && cd fastapi-tdd-docker
$ mkdir project && cd project
$ mkdir app
$ python3.9 -m venv env
# Use [[pyenv]] Obsidian steps if the above line doesn't work

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
```json
{
  "ping": "pong!",
  "environment": "prod",
  "testing": true
}
```

- What happens when you set the `TESTING` environment variable to `foo`? Try this out. Then update the variable to `0`.

- With the server running, navigate to [http://localhost:8000/ping](http://localhost:8000/ping) and then refresh a few times. Back in your terminal, you should see several log messages for:
	- `Loading config settings from the environment...` 

- **Essentially, `get_settings` gets called for each request. If we refactored the config so that the settings were read from a file, instead of from environment variables, it would be much too slow.**

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

### [[Async]] Handlers

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
uvicorn==0.14.0
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


## [[Docker]] Config

- Let's containerize the [[FastAPI]] app.

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

```bash
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
	- `$ docker-compose build` 

- This will take a few minutes the first time. Subsequent builds will be much faster since Docker caches the results. If you'd like to learn more about Docker caching, review the [Order Dockerfile commands](https://mherman.org/presentations/dockercon-2018/#46) slide.

- Once the build is done, fire up the container in [detached mode](https://docs.docker.com/engine/reference/run/#detached--d):
	- `$ docker-compose up -d` 

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
	```markdown
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

### [[Postgres]]

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

`asyncpg==0.23.0` or whatever version is newer that doesn't cause errors.

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
	- `$ docker-compose logs web` 

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
	- `$ docker-compose exec web-db psql -U postgres` 

- Then, you can connect to the database:

```markdown
postgres=# \c web_dev
postgres=# \q
```


### [[Tortoise ORM]]

- To simplify interactions with the database, we'll using an async [[ORM]] called [Tortoise](https://tortoise-orm.readthedocs.io/).

- Add the dependency to the _requirements.txt_ file:
	- `tortoise-orm==0.17.4` 

- Next, create a new folder called "models" within "project/app". Add two files to "models": `__init__.py` and `tortoise.py`.

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

```bash
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


> ✏️ Note: **I was unable to get the above `\dt` command to yield the same results & the localhost URL also does not work at this point, however, It should after following the migrations steps below.** Edit: It worked for me on 2/3/2022 despite the previous note.

### Migrations

- Tortoise supports [[Database Migrations]](https://tortoise-orm.readthedocs.io/en/latest/migration.html) via [[Aerich]](https://github.com/tortoise/aerich). Let's take a few steps back and configure it.

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

```bash
$ docker-compose exec web-db psql -U postgres

psql (13.3)
Type "help" for help.

postgres=# \c web_dev
You are now connected to database "web_dev" as user "postgres".

web_dev=# \dt
Did not find any relations.

web_dev=# \q
```

- Add `Aerich` to the requirements file:
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

---

- I have updated this section of the tutorial where the versions I am using of the dependecies at this point (*as of 12-23-2021*) are:
	- `fastapi==0.65.3`
	- `uvicorn==0.16.0`
	- `asyncpg==0.25.0`
	- `tortoise-orm==0.17.8`
	- `aerich==0.6.1`

> ❗Breaking changes! the line below does not work with aerich v0.6.1 as the config file has changed to using [[TOML]] (Tom's Obvious Minimal Language) and the config file is renamed to `pyproject.toml`. *Version 0.6.1 of Aerich is not yet compatible with tortoise-orm v0.18.0 so I used 0.17.8 instead!* 
- Init:
	- `$ docker-compose exec web aerich init -t app.db.TORTOISE_ORM` 

- This will create a config file called _project/aerich.ini_:

```ini
[aerich]
tortoise_orm = app.db.TORTOISE_ORM
location = ./migrations
```
> ✏️ Note: If using aerich v0.6.1+, then the config file will not be a `.ini` but a `.toml` file with the following shown below.

```toml
# The contents of the `pyproject.toml` export will look like this
[tool.aerich]
tortoise_orm = "app.db.TORTOISE_ORM"
location = "./migrations"
src_folder = "./."
```

---

- Create the first migration:

```bash
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

```bash
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

- **Be sure to review the [documentation](https://tortoise-orm.readthedocs.io/en/latest/migration.html) to learn more about the available commands. Take note of the `downgrade`, `history`, and `upgrade` commands.**

## Pytest Setup

### Setup

- Add a "tests" directory to the "project" directory, and then create the following files inside the newly created directory:

1.  `__init__.py`
2.  `conftest.py`
3.  `test_ping.py`

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

>✏️ Note: In this section I used `pytest==6.2.5` & `requests==2.26.0`
```txt
pytest==6.2.4
requests==2.25.1
```

- We need to re-build the Docker images since requirements are installed at build time rather than run time:
	- `$ docker-compose up -d --build` 

- With the containers up and running, run the tests:
	- `$ docker-compose exec web python -m pytest` 

You should see:

```bash
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

```bash
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

- First, add a new folder called "api" to the "app" folder. Add an `__init__.py` file to the newly created folder.

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

```bash
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

```bash
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

```bash
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

- **Finally, since we do want to use Aerich in dev to manage the database schema, bring the containers and volumes down again:**
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


## [[REST API | RESTful]] Routes

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

```bash
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

- Ensure the test fails with `docker-compose exec web python -m pytest`:

```
>       assert response.status_code == 200
E       assert 404 == 200
E        +  where 404 = <Response [404]>.status_code
```

- Add the following handler to `summaries.py`:

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
        return summary
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

```bash
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

```bash
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

```bash
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

```bash
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

```bash
$ docker-compose exec web python -m pytest -k ping

=============================== test session starts ===============================
platform linux -- Python 3.9.6, pytest-6.2.4, py-1.10.0, pluggy-0.13.1
rootdir: /usr/src/app
collected 6 items / 5 deselected / 1 selected

tests/test_ping.py .                                                        [100%]

========================= 1 passed, 5 deselected in 0.14s =========================
```

So, you can see that the single test in _test_ping.py_ ran (and passed) while the five tests in _test_summaries.py_ were skipped.

Which test(s) will run when you run this command:

`$ docker-compose exec web python -m pytest -k read`


### Pytest Commands

Before moving on, let's review some useful pytest commands:

```bash
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

### [[Gunicorn]]

- To use [Gunicorn](https://gunicorn.org/), a production-grade WSGI server, first add the dependency to the _requirements.txt_ file:
	- `gunicorn==20.1.0` 

> Why Gunicorn and Uvicorn? To get the best of concurrency and parallelism. Gunicorn manages multiple, concurrent Uvicorn processes.

- Add a new Dockerfile called _Dockerfile.prod_:

```dockerfile
# pull official base image
FROM python:3.8.12-slim-buster

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

- First, we started with a Python 3.8.12 image rather than 3.9.6 since [uvloop](https://github.com/MagicStack/uvloop), which `uvicorn.workers.UvicornWorker` uses, [does not support Python 3.9 yet](https://github.com/MagicStack/uvloop/issues/365).

- Next, we added a `CMD` to run Gunicorn (with a [uvicorn worker class](https://www.uvicorn.org/#running-with-gunicorn)) and configured two new environment variables:

```bash
ENV ENVIRONMENT prod
ENV TESTING 0
```

- We also created and switched to a non-root user, which is [recommended by Heroku](https://devcenter.heroku.com/articles/container-registry-and-runtime#run-the-image-as-a-non-root-user).

- Finally, take note of the `$PORT` environment variable in the Gunicorn command. Essentially, our web app must be listening on a particular port specified by the `$PORT` environment variable, which is [supplied by Heroku](https://devcenter.heroku.com/articles/dynos#web-dynos).

### [[Heroku]]

- [Sign up](https://signup.heroku.com/) for a Heroku account (if you don’t already have one), and then install the [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli) (if you haven't already done so).

- Create a new app:

```bash
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

```bash
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

```bash
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

## Code Coverage and Quality

In this chapter, we'll add tools for checking code coverage and quality to the project.

---

### [[Code Coverage]]

- **[Code coverage](https://en.wikipedia.org/wiki/Code_coverage) is the measure of how much code is executed during testing. By adding code coverage to your test suite, you can find areas of your code not covered by tests.**

- **[Coverage.py](https://coverage.readthedocs.io/) is a popular tool for measuring code coverage in Python-based applications. Now, since we're using pytest, we'll integrate Coverage.py with pytest using [[pytest-cov]](https://pytest-cov.readthedocs.io/).**

- Add pytest-cov to the _requirements.txt_ file:
	- `pytest-cov==2.12.1` 

- Next, we can configure the coverage reports in a _.coveragerc_ file. Add this file to the "project" directory, and then add the following config to exclude the tests from the coverage results and enable [branch coverage measurement](https://coverage.readthedocs.io/en/latest/branch.html):

```
[run]
omit = tests/*
branch = True
```

- Update the containers:
	- `$ docker-compose up -d --build` 

- Run the tests with coverage:
	- `$ docker-compose exec web python -m pytest --cov="."` 

- You should see something similar to:

```bash
=============================== test session starts ===============================
platform linux -- Python 3.9.6, pytest-6.2.4, py-1.10.0, pluggy-0.13.1
rootdir: /usr/src/app
plugins: cov-2.12.1
collected 6 items

tests/test_ping.py .                                                        [ 16%]
tests/test_summaries.py .....                                               [100%]

----------- coverage: platform linux, python 3.9.6-final-0 -----------
Name                     Stmts   Miss Branch BrPart  Cover
----------------------------------------------------------
app/__init__.py              0      0      0      0   100%
app/api/__init__.py          0      0      0      0   100%
app/api/crud.py             15      0      2      0   100%
app/api/ping.py              6      0      0      0   100%
app/api/summaries.py        20      0      2      0   100%
app/config.py               13      2      0      0    85%
app/db.py                   17      7      2      1    58%
app/main.py                 18      3      0      0    83%
app/models/__init__.py       0      0      0      0   100%
app/models/pydantic.py       5      0      0      0   100%
app/models/tortoise.py       9      1      0      0    89%
----------------------------------------------------------
TOTAL                      103     13      6      1    87%

================================ 6 passed in 0.56s ================================
```

- Want to view an HTML version?
	- `$ docker-compose exec web python -m pytest --cov="." --cov-report html` 

- The HTML version can be viewed within the newly created "htmlcov" directory. With it, you can quickly see which parts of the code are, and are not, covered by a test.
	- `$ open project/htmlcov/index.html` 

- Add this directory along with the _.coverage_ file to the _.gitignore_ and _.dockerignore_ files.

>**Just keep in mind that while code coverage is a good metric to look at, it does not measure the overall effectiveness of the test suite. In other words, having 100% coverage means that every line of code is being tested; it does not mean that the tests handle every scenario.**
> 
> **Just because you have 100% test coverage doesn’t mean you're testing the right things.**

### [[Code Quality]]

- [Linting](https://stackoverflow.com/a/8503586/1799408) is the process of checking your code for stylistic or programming errors. Although there are a [number](https://github.com/vintasoftware/python-linters-and-code-analysis) of commonly used linters for Python, we'll use [Flake8](https://gitlab.com/pycqa/flake8) since it combines two other popular linters -- [pep8](https://pypi.python.org/pypi/pep8) and [pyflakes](https://pypi.python.org/pypi/pyflakes).
	- [[Linting]]
	- [[Flake8]]

- Add Flake8 to the _requirements.txt_ file:
	- `flake8==3.9.2` 

- To [configure](https://flake8.pycqa.org/en/latest/user/configuration.html) Flake8, add a _setup.cfg_ file to the "project" directory:

```
[flake8]
max-line-length = 119
```

- This sets the maximum allowed line length to 119.

- Update the containers:
	- `$ docker-compose up -d --build` 

- Run Flake8:
	- `$ docker-compose exec web flake8 .` 

Were any errors found?

**Correct any issues before moving on.**

- Next, let's add [Black](https://black.readthedocs.io/), which is used for formatting your code so that "code looks the same regardless of the project you're reading". This helps to speed up code reviews. "Formatting becomes transparent after a while and you can focus on the content instead."
	- [[Black]]

- Add the dependency to the requirements file:
	- `black==21.6b0` 

- Update the containers with `docker-compose up -d --build`, and then run Black with the following:

```bash
$ docker-compose exec web black . --check
```

- Since we used the [check](https://black.readthedocs.io/en/stable/usage_and_configuration/the_basics.html#command-line-options) option, you should see the status:

```bash
would reformat app/api/ping.py
would reformat app/main.py
would reformat app/api/summaries.py
would reformat tests/test_summaries.py
Oh no! 💥 💔 💥
4 files would be reformatted, 11 files would be left unchanged.
```

- Try running it with the [diff](https://black.readthedocs.io/en/stable/usage_and_configuration/the_basics.html#command-line-options) option as well before applying the changes:

```bash
$ docker-compose exec web black . --diff

$ docker-compose exec web black .
```

- Finally, let's add [isort](https://github.com/timothycrosley/isort) to the project as well to quickly sort all our imports alphabetically and automatically separate them into sections.

- Again, add the dependency:
	- `isort==5.9.1` 

- Run it with the [check-only](https://github.com/timothycrosley/isort#the---check-only-option) and [diff](https://github.com/timothycrosley/isort#using-isort) options:

```bash
$ docker-compose up -d --build
$ docker-compose exec web isort . --check-only
$ docker-compose exec web isort . --diff
```

- Then, apply the changes:
	- `$ docker-compose exec web isort .` 

- Verify one last time that Flake8, Black, and isort all pass:

```bash
$ docker-compose exec web flake8 .
$ docker-compose exec web black . --check
$ docker-compose exec web isort . --check-only
```

*Commit your code.*

## Continuous Integration

Next, we'll add continuous integration (CI), via [GitHub Actions](https://github.com/features/actions), to our project. We'll also set up [GitHub Packages](https://github.com/features/packages), a package management service, to store Docker images.

---

> Do you usually use a different [CI service](https://github.com/ligurio/awesome-ci) or run [Jenkins](https://jenkins.io/)? It's encouraged to check your understanding by continuing to use the CI service that you're accustomed to. Convert the GitHub configuration examples as necessary.

### GitHub Packages

- [GitHub Packages](https://github.com/features/packages) is a package management service, fully integrated with GitHub. It allows you to host your software packages, publicly or privately, for use within your projects on GitHub. We'll use it to store Docker images.

- Assuming you have an account on GitHub, create a new repository for this project called `fastapi-tdd-docker`.

- To test locally, you'll need to [create](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line) a personal access token. Within your [Developer Settings](https://github.com/settings/developers), click "Personal access tokens". Then, click "Generate new token". Provide a descriptive note and select the following scopes:

1.  `write:packages`
2.  `read:packages`
3.  `delete:packages`
4.  `workflow`

![[github_personal_access_token.png]]


- Take note of the token.

- Build and tag the image:

```bash
$ docker build -f project/Dockerfile.prod -t docker.pkg.github.com/<USERNAME>/<REPOSITORY_NAME>/summarizer:latest ./project

# example:
# docker build -f project/Dockerfile -t docker.pkg.github.com/jonnyg23/fastapi-tdd-docker/summarizer:latest ./project
```

- Next, using your personal access token, [authenticate](https://help.github.com/en/packages/using-github-packages-with-your-projects-ecosystem/configuring-docker-for-use-with-github-packages#authenticating-to-github-packages) to GitHub Packages with Docker:

```bash
$ docker login docker.pkg.github.com -u <USERNAME> -p <TOKEN>

# example:
# docker login docker.pkg.github.com -u jonnyg23 -p 3f18407a02445cb1837d0851f21a9757eccfdc8a
```

- Push the image to the Docker registry on GitHub Packages:

```bash
$ docker push docker.pkg.github.com/<USERNAME>/<REPOSITORY_NAME>/summarizer:latest

# example:
# docker push docker.pkg.github.com/testdrivenio/fastapi-tdd-docker/summarizer:latest
```

- You should now be able to see the package at the following URL:
	- `https://github.com/<USERNAME>/<REPOSITORY_NAME>/packages`


![[github_packages.png]]


### GitHub Actions

- To configure [GitHub Actions](https://github.com/features/actions), start by adding a new directory called ".github" in the root of your project. Within that directory add another directory called "workflows". Now, to configure a workflow, which is made up of one or more jobs, create a new file in the "workflows" directory called _main.yml_.

```yaml
name: Continuous Integration and Delivery

on: [push]

env:
  IMAGE: docker.pkg.github.com/$(echo $GITHUB_REPOSITORY | tr '[A-Z]' '[a-z]')/summarizer

jobs:

  build:
    name: Build Docker Image
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2.4.0
        with:
          ref: master
      - name: Log in to GitHub Packages
        run: echo ${GITHUB_TOKEN} | docker login -u ${GITHUB_ACTOR} --password-stdin docker.pkg.github.com
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      - name: Pull image
        run: |
          docker pull ${{ env.IMAGE }}:latest || true
      - name: Build image
        run: |
          docker build \
            --cache-from ${{ env.IMAGE }}:latest \
            --tag ${{ env.IMAGE }}:latest \
            --file ./project/Dockerfile.prod \
            "./project"
      - name: Push image
        run: |
          docker push ${{ env.IMAGE }}:latest

  test:
    name: Test Docker Image
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Checkout
        uses: actions/checkout@v2.4.0
        with:
          ref: master
      - name: Log in to GitHub Packages
        run: echo ${GITHUB_TOKEN} | docker login -u ${GITHUB_ACTOR} --password-stdin docker.pkg.github.com
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      - name: Pull image
        run: |
          docker pull ${{ env.IMAGE }}:latest || true
      - name: Build image
        run: |
          docker build \
            --cache-from ${{ env.IMAGE }}:latest \
            --tag ${{ env.IMAGE }}:latest \
            --file ./project/Dockerfile.prod \
            "./project"
      - name: Run container
        run: |
          docker run \
            -d \
            --name fastapi-tdd \
            -e PORT=8765 \
            -e ENVIRONMENT=dev \
            -e DATABASE_URL=sqlite://sqlite.db \
            -e DATABASE_TEST_URL=sqlite://sqlite.db \
            -p 5003:8765 \
            ${{ env.IMAGE }}:latest
      - name: Pytest
        run: docker exec fastapi-tdd python -m pytest .
      - name: Flake8
        run: docker exec fastapi-tdd python -m flake8 .
      - name: Black
        run: docker exec fastapi-tdd python -m black . --check
      - name: isort
        run: docker exec fastapi-tdd python -m isort . --check-only
```

- Here, after setting the global `IMAGE` environment variable, we defined two [jobs](https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobs), `build` and `test`.

- In the `build` job, we:

1.  Check out the repository so the job has access to it
2.  Log in to GitHub Packages
3.  Pull the image if it exists
4.  Build the image
5.  Push the image up to GitHub Packages

- Then, after building and running the image, in the `test` job, we run pytest, Flake8, Black, and isort.

- Commit your changes, and then push to GitHub. This _should_ trigger a new build, which _should_ pass.

- Once done, add a _README.md_ file to the project root, adding the GitHub status badge:

```markdown
![Continuouse Integration and Delivery](https://github.com/jonnyg23/meg-sews-things-co/workflows/Continuous%20Integration%20and%20Delivery/badge.svg?branch=main)
```

> Be sure to replace `YOUR_GITHUB_NAMESPACE` with your actual GitHub username or organization.


## Continuous Delivery

- In this chapter, we'll add continuous delivery (CD) to the existing GitHub Actions config.

---

- First, retrieve your [Heroku auth token](https://devcenter.heroku.com/articles/authentication):
	- `$ heroku auth:token` 

- Then, save the token as a new variable called `HEROKU_AUTH_TOKEN` within your repository's [secrets](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets#creating-encrypted-secrets) (Settings > Secrets):

![[github-secrets.png]]


- Next, add a new `deploy` job to the GitHub config file inside of `jobs` in line vertically with `build`:

```yaml
deploy:
  name: Deploy to Heroku
  runs-on: ubuntu-latest
  needs: [build, test]
  env:
    HEROKU_APP_NAME: <APP_NAME>
    HEROKU_REGISTRY_IMAGE: registry.heroku.com/${HEROKU_APP_NAME}/summarizer
  steps:
    - name: Checkout
      uses: actions/checkout@v2.4.0
      with:
        ref: master
    - name: Log in to GitHub Packages
      run: echo ${GITHUB_TOKEN} | docker login -u ${GITHUB_ACTOR} --password-stdin docker.pkg.github.com
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    - name: Pull image
      run: |
        docker pull ${{ env.IMAGE }}:latest || true
    - name: Build image
      run: |
        docker build \
          --cache-from ${{ env.IMAGE }}:latest \
          --tag ${{ env.HEROKU_REGISTRY_IMAGE }}:latest \
          --file ./project/Dockerfile.prod \
          "./project"
    - name: Log in to the Heroku Container Registry
      run: docker login -u _ -p ${HEROKU_AUTH_TOKEN} registry.heroku.com
      env:
        HEROKU_AUTH_TOKEN: ${{ secrets.HEROKU_AUTH_TOKEN }}
    - name: Push to the registry
      run: docker push ${{ env.HEROKU_REGISTRY_IMAGE }}
    - name: Set environment variables
      run: |
        echo "HEROKU_REGISTRY_IMAGE=${{ env.HEROKU_REGISTRY_IMAGE }}" >> $GITHUB_ENV
        echo "HEROKU_AUTH_TOKEN=${{ secrets.HEROKU_AUTH_TOKEN }}" >> $GITHUB_ENV
    - name: Release
      run: |
        chmod +x ./release.sh
        ./release.sh
```

> Make sure to replace `<APP_NAME>` with your Heroku app's actual name. As well as the file path under *docker build* and `release.sh` so it points to the files specified. For example: I had all of the backend in a folder titled "backend" so I had to use this path `./backend/project/Dockerfile.prod`. Do this for the `"./project"` section as well. When using this tutorial/course, the section called `ref`  said `main`, however, I had to change this to `master` since that was the branch I was using.

- Add _release.sh_ to the project root:

```sh
#!/bin/sh

set -e

IMAGE_ID=$(docker inspect ${HEROKU_REGISTRY_IMAGE} --format={{.Id}})
PAYLOAD='{"updates": [{"type": "web", "docker_image": "'"$IMAGE_ID"'"}]}'

curl -n -X PATCH https://api.heroku.com/apps/$HEROKU_APP_NAME/formation \
  -d "${PAYLOAD}" \
  -H "Content-Type: application/json" \
  -H "Accept: application/vnd.heroku+json; version=3.docker-releases" \
  -H "Authorization: Bearer ${HEROKU_AUTH_TOKEN}"
```

- In the `deploy` stage, we:

1.  Check out the repository so the job has access to it
2.  Log in to GitHub Packages
3.  Pull the image if it exists
4.  Build and tag the new image
5.  Log in to the [Heroku Container Registry](https://devcenter.heroku.com/articles/container-registry-and-runtime)
6.  Push the image up to the registry
7.  [Set](https://help.github.com/en/actions/reference/workflow-commands-for-github-actions#setting-an-environment-variable) the `HEROKU_REGISTRY_IMAGE` and `HEROKU_AUTH_TOKEN` environment variables with `set-env` so they can be accessed within the release file
8.  Create a new release via the [Heroku API](https://devcenter.heroku.com/articles/container-registry-and-runtime#api) using the image ID within the _release.sh_ script

- To test, make a quick change to the `/ping` GET route by removing the exclamation point (`!`):

```python
@router.get("/ping")
async def pong(settings: Settings = Depends(get_settings)):
    return {
        "ping": "pong",
        "environment": settings.environment,
        "testing": settings.testing,
    }
```

- Update the test as well:

```python
# project/tests/test_ping.py

def test_ping(test_app):
    response = test_app.get("/ping")
    assert response.status_code == 200
    assert response.json() == {"environment": "dev", "ping": "pong", "testing": True}
```

- Commit your code and again push it up to GitHub. Your app should be auto deployed to Heroku! Navigate to [https://APP_NAME.herokuapp.com/ping/](https://app_name.herokuapp.com/ping/). You should now see:

```json
{
  "ping": "pong",
  "environment": "dev",
  "testing": false
}
```


### Jonny Important Note!!

>Note: I had to change the python production version for deployment to `3.8.12-slim-buster` instead of `3.8.11-slim-buster`. I also had to ensure that there were not any unused dependencies in any python files or the *flake8* test would fail in Github Actions.


## Remaining Routes

Next, let's finish setting up the routes:

| **Endpoint**   | **HTTP Method** | **CRUD Method** | **Result**           |
| -------------- | --------------- | --------------- | -------------------- |
| /summaries     | GET             | READ            | get all summaries    |
| /summaries/:id | GET             | READ            | get a single summary |
| /summaries     | POST            | CREATE          | add a summary        |
| /summaries/:id | PUT             | UPDATE          | update a summary     |
| /summaries/:id | DELETE          | DELETE          | delete a summary     | 


Again, for each, we'll:
1.  write a test
2.  run the test, to ensure it fails (**red**)
3.  write just enough code to get the test to pass (**green**)
4.  **refactor** (if necessary)


### DELETE Route

- Add the following tests to _test_summaries.py_ in "project/tests":

```python
def test_remove_summary(test_app_with_db):
    response = test_app_with_db.post(
        "/summaries/", data=json.dumps({"url": "https://foo.bar"})
    )
    summary_id = response.json()["id"]

    response = test_app_with_db.delete(f"/summaries/{summary_id}/")
    assert response.status_code == 200
    assert response.json() == {"id": summary_id, "url": "https://foo.bar"}

def test_remove_summary_incorrect_id(test_app_with_db):
    response = test_app_with_db.delete("/summaries/999/")
    assert response.status_code == 404
    assert response.json()["detail"] == "Summary not found"
```

- Run the tests to ensure they both fail:
	- `$ docker-compose exec web python -m pytest` 

- You should the following errors:

```bash
>       assert response.status_code == 200
E       assert 405 == 200
E        +  where 405 = <Response [405]>.status_code

>       assert response.status_code == 404
E       assert 405 == 404
E        +  where 405 = <Response [405]>.status_code
```

- Then add the route handler to _project/app/api/summaries.py_:

```python
@router.delete("/{id}/", response_model=SummaryResponseSchema)
async def delete_summary(id: int) -> SummaryResponseSchema:
    summary = await crud.get(id)
    if not summary:
        raise HTTPException(status_code=404, detail="Summary not found")

    await crud.delete(id)

    return summary
```

- Add the CRUD util to _project/app/api/crud.py_:

```python
async def delete(id: int) -> int:
    summary = await TextSummary.filter(id=id).first().delete()
    return summary
```

- Ensure the tests pass:

```bash
=============================== test session starts ===============================
platform linux -- Python 3.9.6, pytest-6.2.4, py-1.10.0, pluggy-0.13.1
rootdir: /usr/src/app
plugins: cov-2.12.1
collected 8 items

tests/test_ping.py .                                                        [ 12%]
tests/test_summaries.py .......                                             [100%]

================================ 8 passed in 0.26s ================================
```


### PUT Route

- Start with some tests:

```python
def test_update_summary(test_app_with_db):
    response = test_app_with_db.post(
        "/summaries/", data=json.dumps({"url": "https://foo.bar"})
    )
    summary_id = response.json()["id"]

    response = test_app_with_db.put(
        f"/summaries/{summary_id}/",
        data=json.dumps({"url": "https://foo.bar", "summary": "updated!"})
    )
    assert response.status_code == 200

    response_dict = response.json()
    assert response_dict["id"] == summary_id
    assert response_dict["url"] == "https://foo.bar"
    assert response_dict["summary"] == "updated!"
    assert response_dict["created_at"]

def test_update_summary_incorrect_id(test_app_with_db):
    response = test_app_with_db.put(
        "/summaries/999/",
        data=json.dumps({"url": "https://foo.bar", "summary": "updated!"})
    )
    assert response.status_code == 404
    assert response.json()["detail"] == "Summary not found"

def test_update_summary_invalid_json(test_app_with_db):
    response = test_app_with_db.post(
        "/summaries/", data=json.dumps({"url": "https://foo.bar"})
    )
    summary_id = response.json()["id"]

    response = test_app_with_db.put(
        f"/summaries/{summary_id}/",
        data=json.dumps({})
    )
    assert response.status_code == 422
    assert response.json() == {
        "detail": [
            {
                "loc": ["body", "url"],
                "msg": "field required",
                "type": "value_error.missing",
            },
            {
                "loc": ["body", "summary"],
                "msg": "field required",
                "type": "value_error.missing",
            }
        ]
    }

def test_update_summary_invalid_keys(test_app_with_db):
    response = test_app_with_db.post(
        "/summaries/", data=json.dumps({"url": "https://foo.bar"})
    )
    summary_id = response.json()["id"]

    response = test_app_with_db.put(
        f"/summaries/{summary_id}/",
        data=json.dumps({"url": "https://foo.bar"})
    )
    assert response.status_code == 422
    assert response.json() == {
        "detail": [
            {
                "loc": ["body", "summary"],
                "msg": "field required",
                "type": "value_error.missing",
            }
        ]
    }
```

- Ensure the tests fail.

Add the handler:

```python
@router.put("/{id}/", response_model=SummarySchema)
async def update_summary(id: int, payload: SummaryUpdatePayloadSchema) -> SummarySchema:
    summary = await crud.put(id, payload)
    if not summary:
        raise HTTPException(status_code=404, detail="Summary not found")

    return summary
```

- Add a new Pydantic model to _project/app/models/pydantic.py_:

```python
class SummaryUpdatePayloadSchema(SummaryPayloadSchema):
    summary: str
```

- Make sure to import it in _project/app/api/summaries.py_:

```python
from app.models.pydantic import (  # isort:skip
    SummaryPayloadSchema,
    SummaryResponseSchema,
    SummaryUpdatePayloadSchema,
)
```

- Did you notice the [isort:skip](https://pycqa.github.io/isort/docs/configuration/action_comments.html#isort-skip) action comment? This prevents isort from formatting the associated import. It's necessary as both isort and Black will attempt to format it in different ways.

- Util:

```python
async def put(id: int, payload: SummaryPayloadSchema) -> Union[dict, None]:
    summary = await TextSummary.filter(id=id).update(url=payload.url, summary=payload.summary)
    if summary:
        updated_summary = await TextSummary.filter(id=id).first().values()
        return updated_summary[0]
    return None
```

- Do the tests pass now?

```bash
=============================== test session starts ===============================
platform linux -- Python 3.9.6, pytest-6.2.4, py-1.10.0, pluggy-0.13.1
rootdir: /usr/src/app
plugins: cov-2.12.1
collected 12 items

tests/test_ping.py .                                                        [  8%]
tests/test_summaries.py ...........                                         [100%]

=============================== 12 passed in 0.30s ================================
```

### Additional Validation

Let's add some additional validation to the routes, checking that:

1.  The id is greater than `0` for reading, updating, and deleting a single summary
2.  The URL is valid for adding and updating a summary

#### GET

- Update the `test_read_summary_incorrect_id` test:

```python
def test_read_summary_incorrect_id(test_app_with_db):
    response = test_app_with_db.get("/summaries/999/")
    assert response.status_code == 404
    assert response.json()["detail"] == "Summary not found"

    response = test_app_with_db.get("/summaries/0/")
    assert response.status_code == 422
    assert response.json() == {
        "detail": [
            {
                "loc": ["path", "id"],
                "msg": "ensure this value is greater than 0",
                "type": "value_error.number.not_gt",
                "ctx": {"limit_value": 0},
            }
        ]
    }
```

- The test should fail:

```
>       assert response.status_code == 422
E       assert 404 == 422
E        +  where 404 = <Response [404]>.status_code
```

- Update the handler:

```python
@router.get("/{id}/", response_model=SummarySchema)
async def read_summary(id: int = Path(..., gt=0)) -> SummarySchema:
    summary = await crud.get(id)
    if not summary:
        raise HTTPException(status_code=404, detail="Summary not found")

    return summary
```

- Make sure to import `Path`:
	- `from fastapi import APIRouter, HTTPException, Path` 


So, we added the following metadata to the parameter with [Path](https://fastapi.tiangolo.com/tutorial/path-params-numeric-validations/):
1.  `...` - the value is required ([Ellipsis](https://docs.python.org/3/library/constants.html#Ellipsis))
2.  `gt` - the value must be [greater than](https://fastapi.tiangolo.com/tutorial/path-params-numeric-validations/#number-validations-greater-than-or-equal) 0

The tests should pass. Try out the API documentation as well:

![[swagger-ui-error.png]]


#### POST

- Update the `test_create_summaries_invalid_json` test:

```python
def test_create_summaries_invalid_json(test_app):
    response = test_app.post("/summaries/", data=json.dumps({}))
    assert response.status_code == 422
    assert response.json() == {
        "detail": [
            {
                "loc": ["body", "url"],
                "msg": "field required",
                "type": "value_error.missing",
            }
        ]
    }

    response = test_app.post("/summaries/", data=json.dumps({"url": "invalid://url"}))
    assert response.status_code == 422
    assert response.json()["detail"][0]["msg"] == "URL scheme not permitted"
```

The test should fail:

```
>       assert response.status_code == 422
E       assert 201 == 422
E        +  where 201 = <Response [201]>.status_code
```

- To get the test to pass, update the `SummaryPayloadSchema` model like so:

```python
class SummaryPayloadSchema(BaseModel):
    url: AnyHttpUrl
```

- Here, we added additional validation to the Pydantic model with the [AnyHttpUrl](https://pydantic-docs.helpmanual.io/usage/types/#urls) validator.

- Add the import:
	- `from pydantic import BaseModel, AnyHttpUrl`

#### PUT

- Update the `test_update_summary_incorrect_id` test:

```python
def test_update_summary_incorrect_id(test_app_with_db):
    response = test_app_with_db.put(
        "/summaries/999/",
        data=json.dumps({"url": "https://foo.bar", "summary": "updated!"})
    )
    assert response.status_code == 404
    assert response.json()["detail"] == "Summary not found"

    response = test_app_with_db.put(
        f"/summaries/0/",
        data=json.dumps({"url": "https://foo.bar", "summary": "updated!"})
    )
    assert response.status_code == 422
    assert response.json() == {
        "detail": [
            {
                "loc": ["path", "id"],
                "msg": "ensure this value is greater than 0",
                "type": "value_error.number.not_gt",
                "ctx": {"limit_value": 0},
            }
        ]
    }
```

- Update the handler:

```python
@router.put("/{id}/", response_model=SummarySchema)
async def update_summary(payload: SummaryUpdatePayloadSchema, id: int = Path(..., gt=0)) -> SummarySchema:
    summary = await crud.put(id, payload)
    if not summary:
        raise HTTPException(status_code=404, detail="Summary not found")

    return summary
```

- Update `test_update_summary_invalid_keys` as well:

```python
def test_update_summary_invalid_keys(test_app_with_db):
    response = test_app_with_db.post(
        "/summaries/", data=json.dumps({"url": "https://foo.bar"})
    )
    summary_id = response.json()["id"]

    response = test_app_with_db.put(
        f"/summaries/{summary_id}/",
        data=json.dumps({"url": "https://foo.bar"})
    )
    assert response.status_code == 422
    assert response.json() == {
        "detail": [
            {
                "loc": ["body", "summary"],
                "msg": "field required",
                "type": "value_error.missing",
            }
        ]
    }

    response = test_app_with_db.put(
        f"/summaries/{summary_id}/",
        data=json.dumps({"url": "invalid://url", "summary": "updated!"})
    )
    assert response.status_code == 422
    assert response.json()["detail"][0]["msg"] == "URL scheme not permitted"
```

**This should pass.**


#### DELETE

- Test:

```python
def test_remove_summary_incorrect_id(test_app_with_db):
    response = test_app_with_db.delete("/summaries/999/")
    assert response.status_code == 404
    assert response.json()["detail"] == "Summary not found"

    response = test_app_with_db.delete("/summaries/0/")
    assert response.status_code == 422
    assert response.json() == {
        "detail": [
            {
                "loc": ["path", "id"],
                "msg": "ensure this value is greater than 0",
                "type": "value_error.number.not_gt",
                "ctx": {"limit_value": 0},
            }
        ]
    }
```

- Make sure the tests fail.

- Handler:

```python
@router.delete("/{id}/", response_model=SummaryResponseSchema)
async def delete_summary(id: int = Path(..., gt=0)) -> SummaryResponseSchema:
    summary = await crud.get(id)
    if not summary:
        raise HTTPException(status_code=404, detail="Summary not found")

    await crud.delete(id)

    return summary
```

- **All tests should pass!**

```bash
=============================== test session starts ===============================
platform linux -- Python 3.9.6, pytest-6.2.4, py-1.10.0, pluggy-0.13.1
rootdir: /usr/src/app
plugins: cov-2.12.1
collected 12 items

tests/test_ping.py .                                                        [  8%]
tests/test_summaries.py ...........                                         [100%]

=============================== 12 passed in 0.31s ================================
```


### Parametrizing Test Functions

- Parameterized tests allow a developer to run the same test multiple times with different data inputs.

- Take note of these three tests:

```python
def test_update_summary_incorrect_id(test_app_with_db):
    response = test_app_with_db.put(
        "/summaries/999/",
        data=json.dumps({"url": "https://foo.bar", "summary": "updated!"})
    )
    assert response.status_code == 404
    assert response.json()["detail"] == "Summary not found"

    response = test_app_with_db.put(
        f"/summaries/0/",
        data=json.dumps({"url": "https://foo.bar", "summary": "updated!"})
    )
    assert response.status_code == 422
    assert response.json() == {
        "detail": [
            {
                "loc": ["path", "id"],
                "msg": "ensure this value is greater than 0",
                "type": "value_error.number.not_gt",
                "ctx": {"limit_value": 0},
            }
        ]
    }

def test_update_summary_invalid_json(test_app_with_db):
    response = test_app_with_db.post(
        "/summaries/", data=json.dumps({"url": "https://foo.bar"})
    )
    summary_id = response.json()["id"]

    response = test_app_with_db.put(
        f"/summaries/{summary_id}/",
        data=json.dumps({})
    )
    assert response.status_code == 422
    assert response.json() == {
        "detail": [
            {
                "loc": ["body", "url"],
                "msg": "field required",
                "type": "value_error.missing",
            },
            {
                "loc": ["body", "summary"],
                "msg": "field required",
                "type": "value_error.missing",
            }
        ]
    }

def test_update_summary_invalid_keys(test_app_with_db):
    response = test_app_with_db.post(
        "/summaries/", data=json.dumps({"url": "https://foo.bar"})
    )
    summary_id = response.json()["id"]

    response = test_app_with_db.put(
        f"/summaries/{summary_id}/",
        data=json.dumps({"url": "https://foo.bar"})
    )
    assert response.status_code == 422
    assert response.json() == {
        "detail": [
            {
                "loc": ["body", "summary"],
                "msg": "field required",
                "type": "value_error.missing",
            }
        ]
    }

    response = test_app_with_db.put(
        f"/summaries/{summary_id}/",
        data=json.dumps({"url": "invalid://url", "summary": "updated!"})
    )
    assert response.status_code == 422
    assert response.json()["detail"][0]["msg"] == "URL scheme not permitted"
```

- These tests really differ only in their inputs and expected outputs, so we can use the [pytest.mark.parametrize](https://docs.pytest.org/en/latest/how-to/parametrize.html#pytest-mark-parametrize-parametrizing-test-functions) decorator to enable parametrization of arguments in order to run the same test multiple times with different data configurations.

- Replace the three tests with these two:

```python
@pytest.mark.parametrize("summary_id, payload, status_code, detail", [
    [999, {"url": "https://foo.bar", "summary": "updated!"}, 404, "Summary not found"],
    [
        0,
        {"url": "https://foo.bar", "summary": "updated!"},
        422,
        [{"loc": ["path", "id"], "msg": "ensure this value is greater than 0", "type": "value_error.number.not_gt", "ctx": {"limit_value": 0}}]
    ],
    [
        1,
        {},
        422,
        [
            {"loc": ["body", "url"], "msg": "field required", "type": "value_error.missing"},
            {"loc": ["body", "summary"], "msg": "field required", "type": "value_error.missing"}
        ]
    ],
    [
        1,
        {"url": "https://foo.bar"},
        422,
        [{"loc": ["body", "summary"], "msg": "field required", "type": "value_error.missing"}]
    ],
])
def test_update_summary_invalid(test_app_with_db, summary_id, payload, status_code, detail):
    response = test_app_with_db.put(
        f"/summaries/{summary_id}/",
        data=json.dumps(payload)
    )
    assert response.status_code == status_code
    assert response.json()["detail"] == detail

def test_update_summary_invalid_url(test_app):
    response = test_app.put(
        "/summaries/1/",
        data=json.dumps({"url": "invalid://url", "summary": "updated!"})
    )
    assert response.status_code == 422
    assert response.json()["detail"][0]["msg"] == "URL scheme not permitted"
```

- Add the import:
	- `import pytest` 

**Make sure the tests still pass.**

### Tests

- How does our test coverage look?

```bash
$ docker-compose exec web python -m pytest --cov="."

=============================== test session starts ===============================
platform linux -- Python 3.9.6, pytest-6.2.4, py-1.10.0, pluggy-0.13.1
rootdir: /usr/src/app
plugins: cov-2.12.1
collected 14 items

tests/test_ping.py .                                                        [  7%]
tests/test_summaries.py .............                                       [100%]

----------- coverage: platform linux, python 3.9.6-final-0 -----------
Name                     Stmts   Miss Branch BrPart  Cover
----------------------------------------------------------
app/__init__.py              0      0      0      0   100%
app/api/__init__.py          0      0      0      0   100%
app/api/crud.py             24      0      4      0   100%
app/api/ping.py              6      0      0      0   100%
app/api/summaries.py        33      0      6      0   100%
app/config.py               13      2      0      0    85%
app/db.py                   17      7      2      1    58%
app/main.py                 18      3      0      0    83%
app/models/__init__.py       0      0      0      0   100%
app/models/pydantic.py       7      0      0      0   100%
app/models/tortoise.py       9      1      0      0    89%
----------------------------------------------------------
TOTAL                      127     13     12      1    90%

=============================== 14 passed in 0.68s ================================
```

- Run Flake8, Black, and isort:

```bash
$ docker-compose exec web flake8 .
$ docker-compose exec web black .
$ docker-compose exec web isort .
```

- Commit and push your code up to GitHub to trigger a new build. Once the deploy is complete, test all the routes with HTTPie.

- GET all summaries:

```bash
$ http GET https://thawing-beach-87010.herokuapp.com/summaries/

HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 114
Content-Type: application/json
Date: Wed, 07 Jul 2021 15:41:48 GMT
Server: uvicorn
Via: 1.1 vegur

[
    {
        "created_at": "2021-07-06T12:30:02.622318+00:00",
        "id": 1,
        "summary": "dummy summary",
        "url": "https://testdriven.io"
    }
]
```

- GET single summary:

```bash
$ http GET https://thawing-beach-87010.herokuapp.com/summaries/1/

HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 112
Content-Type: application/json
Date: Wed, 07 Jul 2021 15:42:14 GMT
Server: uvicorn
Via: 1.1 vegur

{
    "created_at": "2021-07-06T12:30:02.622318+00:00",
    "id": 1,
    "summary": "dummy summary",
    "url": "https://testdriven.io"
}
```

- POST:

```bash
$ http --json POST https://thawing-beach-87010.herokuapp.com/summaries/ url=https://testdriven.io

HTTP/1.1 201 Created
Connection: keep-alive
Content-Length: 38
Content-Type: application/json
Date: Wed, 07 Jul 2021 15:42:40 GMT
Server: uvicorn
Via: 1.1 vegur

{
    "id": 2,
    "url": "https://testdriven.io"
}
```

- PUT:

```bash
$ http --json PUT https://thawing-beach-87010.herokuapp.com/summaries/2/ url=https://testdriven.io summary=super

HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 104
Content-Type: application/json
Date: Wed, 07 Jul 2021 15:43:06 GMT
Server: uvicorn
Via: 1.1 vegur

{
    "created_at": "2021-07-07T15:42:40.975366+00:00",
    "id": 2,
    "summary": "super",
    "url": "https://testdriven.io"
}
```

- DELETE:

```bash
$ http DELETE https://thawing-beach-87010.herokuapp.com/summaries/2/

HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 38
Content-Type: application/json
Date: Wed, 07 Jul 2021 15:43:31 GMT
Server: uvicorn
Via: 1.1 vegur

{
    "id": 2,
    "url": "https://testdriven.io"
}
```


## [[Pytest Monkeypatching]]

In this chapter, we'll use the pytest `monkeypatch` fixture to mock functionality in our tests.

---

### Monkeypatching

- Monkeypatching is the act of dynamically changing a piece of code at runtime. Essentially, it allows you to override the default behavior of a module, object, method, or function without changing its source code.

Let's look at a quick example.

Source code:

```python
# twitter.py

import os
import tweepy

consumer_key = os.getenv("CONSUMER_KEY")
consumer_secret = os.getenv("CONSUMER_SECRET")
access_key = os.getenv("ACCESS_KEY")
access_secret = os.getenv("ACCESS_SECRET")

def get_friends(user):
    """
 Given a valid Twitter user 20 friends are returned
 """

    auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
    auth.set_access_token(access_key, access_secret)
    api = tweepy.API(auth)

    friends = []

    try:
        user = api.get_user(user)
    except tweepy.error.TweepError:
        return "Failed to get request token."

    for friend in user.friends():
        friends.append(friend.screen_name)

    return friends
```

- So, the `get_friends` function uses `tweepy` to authenticate against the Twitter API. Once authenticated, it then returns a list of twenty friends.

- Test:

```python
# test.py

import twitter

def test_get_friends(monkeypatch):
    def mock_get_friends(user):
        return [(lambda x: f"friend{x}")(x) for x in range(20)]

    monkeypatch.setattr(twitter, "get_friends", mock_get_friends)

    assert len(twitter.get_friends("testdrivenio")) == 20
```

- `test_get_friends` uses the pytest [monkeypatch](https://docs.pytest.org/en/latest/how-to/monkeypatch.html) fixture to create a mocked version of `get_friends`, called `mock_get_friends`, that returns a list of twenty strings. Then, during a test run, `mock_get_friends` gets called rather than the real `get_friends` function. This not only decreases the amount of time it will take for the test to run, but it also makes the test more predictable since it is not affected by network connectivity issues, outages in the Twitter API, or rate limiting issues.

- That said, keep in mind that the test is not actually testing the `get_friends` function call; it's replacing the function's default behavior (authenticating and calling the Twitter API) with new behavior (simply returning a list of strings).

- While mocking or monkeypatching can speed up test runs and make the tests more predictable, at some point in the testing process, possibly in a staging environment, you should test out all external communication so that you can be confident that the system works as expected. This is often achieved with some form of end-to-end tests.

- In our tests, we'll use the `monkeypatch` fixture to mock database calls.

### Unit Tests

- Add a new file to "project/tests" called _test_summaries_unit.py_:

```python
# project/tests/test_summaries_unit.py

import json
from datetime import datetime

import pytest

from app.api import crud, summaries

def test_create_summary(test_app, monkeypatch):
    pass

def test_create_summaries_invalid_json(test_app):
    response = test_app.post("/summaries/", data=json.dumps({}))
    assert response.status_code == 422
    assert response.json() == {
        "detail": [
            {
                "loc": ["body", "url"],
                "msg": "field required",
                "type": "value_error.missing",
            }
        ]
    }

    response = test_app.post("/summaries/", data=json.dumps({"url": "invalid://url"}))
    assert response.status_code == 422
    assert response.json()["detail"][0]["msg"] == "URL scheme not permitted"

def test_read_summary(test_app, monkeypatch):
    pass

def test_read_summary_incorrect_id(test_app, monkeypatch):
    pass

def test_read_all_summaries(test_app, monkeypatch):
    pass

def test_remove_summary(test_app, monkeypatch):
    pass

def test_remove_summary_incorrect_id(test_app, monkeypatch):
    pass

def test_update_summary(test_app, monkeypatch):
    pass

@pytest.mark.parametrize(
    "summary_id, payload, status_code, detail",
    [
        [
            999,
            {"url": "https://foo.bar", "summary": "updated!"},
            404,
            "Summary not found",
        ],
        [
            0,
            {"url": "https://foo.bar", "summary": "updated!"},
            422,
            [
                {
                    "loc": ["path", "id"],
                    "msg": "ensure this value is greater than 0",
                    "type": "value_error.number.not_gt",
                    "ctx": {"limit_value": 0},
                }
            ],
        ],
        [
            1,
            {},
            422,
            [
                {
                    "loc": ["body", "url"],
                    "msg": "field required",
                    "type": "value_error.missing",
                },
                {
                    "loc": ["body", "summary"],
                    "msg": "field required",
                    "type": "value_error.missing",
                },
            ],
        ],
        [
            1,
            {"url": "https://foo.bar"},
            422,
            [
                {
                    "loc": ["body", "summary"],
                    "msg": "field required",
                    "type": "value_error.missing",
                }
            ],
        ],
    ],
)
def test_update_summary_invalid(test_app, monkeypatch, summary_id, payload, status_code, detail):
    pass

def test_update_summary_invalid_url(test_app):
    response = test_app.put(
        f"/summaries/1/",
        data=json.dumps({"url": "invalid://url", "summary": "updated!"}),
    )
    assert response.status_code == 422
    assert response.json()["detail"][0]["msg"] == "URL scheme not permitted"
```

- So, we'll write all the same tests as before, but we'll monkeypatch calls to the CRUD service.

#### Add Summary

```python
def test_create_summary(test_app, monkeypatch):
    test_request_payload = {"url": "https://foo.bar"}
    test_response_payload = {"id": 1, "url": "https://foo.bar"}

    async def mock_post(payload):
        return 1

    monkeypatch.setattr(crud, "post", mock_post)

    response = test_app.post("/summaries/", data=json.dumps(test_request_payload),)

    assert response.status_code == 201
    assert response.json() == test_response_payload
```

#### Get Summary

```python
def test_read_summary(test_app, monkeypatch):
    test_data = {
        "id": 1,
        "url": "https://foo.bar",
        "summary": "summary",
        "created_at": datetime.utcnow().isoformat(),
    }

    async def mock_get(id):
        return test_data

    monkeypatch.setattr(crud, "get", mock_get)

    response = test_app.get("/summaries/1/")
    assert response.status_code == 200
    assert response.json() == test_data


def test_read_summary_incorrect_id(test_app, monkeypatch):
    async def mock_get(id):
        return None

    monkeypatch.setattr(crud, "get", mock_get)

    response = test_app.get("/summaries/999/")
    assert response.status_code == 404
    assert response.json()["detail"] == "Summary not found"


def test_read_all_summaries(test_app, monkeypatch):
    test_data = [
        {
            "id": 1,
            "url": "https://foo.bar",
            "summary": "summary",
            "created_at": datetime.utcnow().isoformat(),
        },
        {
            "id": 2,
            "url": "https://testdrivenn.io",
            "summary": "summary",
            "created_at": datetime.utcnow().isoformat(),
        }
    ]

    async def mock_get_all():
        return test_data

    monkeypatch.setattr(crud, "get_all", mock_get_all)

    response = test_app.get("/summaries/")
    assert response.status_code == 200
    assert response.json() == test_data
```

#### Remove Summary

```python
def test_remove_summary(test_app, monkeypatch):
    async def mock_get(id):
        return {
            "id": 1,
            "url": "https://foo.bar",
            "summary": "summary",
            "created_at": datetime.utcnow().isoformat(),
        }

    monkeypatch.setattr(crud, "get", mock_get)

    async def mock_delete(id):
        return id

    monkeypatch.setattr(crud, "delete", mock_delete)

    response = test_app.delete("/summaries/1/")
    assert response.status_code == 200
    assert response.json() == {"id": 1, "url": "https://foo.bar"}


def test_remove_summary_incorrect_id(test_app, monkeypatch):
    async def mock_get(id):
        return None

    monkeypatch.setattr(crud, "get", mock_get)

    response = test_app.delete("/summaries/999/")
    assert response.status_code == 404
    assert response.json()["detail"] == "Summary not found"
```

#### Update Summary

```python
def test_update_summary(test_app, monkeypatch):
    test_request_payload = {"url": "https://foo.bar", "summary": "updated"}
    test_response_payload = {
        "id": 1,
        "url": "https://foo.bar",
        "summary": "summary",
        "created_at": datetime.utcnow().isoformat(),
    }

    async def mock_put(id, payload):
        return test_response_payload

    monkeypatch.setattr(crud, "put", mock_put)

    response = test_app.put("/summaries/1/", data=json.dumps(test_request_payload),)
    assert response.status_code == 200
    assert response.json() == test_response_payload


@pytest.mark.parametrize(
    "summary_id, payload, status_code, detail",
    [
        [
            999,
            {"url": "https://foo.bar", "summary": "updated!"},
            404,
            "Summary not found",
        ],
        [
            0,
            {"url": "https://foo.bar", "summary": "updated!"},
            422,
            [
                {
                    "loc": ["path", "id"],
                    "msg": "ensure this value is greater than 0",
                    "type": "value_error.number.not_gt",
                    "ctx": {"limit_value": 0},
                }
            ],
        ],
        [
            1,
            {},
            422,
            [
                {
                    "loc": ["body", "url"],
                    "msg": "field required",
                    "type": "value_error.missing",
                },
                {
                    "loc": ["body", "summary"],
                    "msg": "field required",
                    "type": "value_error.missing",
                },
            ],
        ],
        [
            1,
            {"url": "https://foo.bar"},
            422,
            [
                {
                    "loc": ["body", "summary"],
                    "msg": "field required",
                    "type": "value_error.missing",
                }
            ],
        ],
    ],
)
def test_update_summary_invalid(test_app, monkeypatch, summary_id, payload, status_code, detail):
    async def mock_put(id, payload):
        return None

    monkeypatch.setattr(crud, "put", mock_put)

    response = test_app.put(f"/summaries/{summary_id}/", data=json.dumps(payload))
    assert response.status_code == status_code
    assert response.json()["detail"] == detail
```

#### Test

- Make sure the tests pass:

```bash
=============================== test session starts ===============================
platform linux -- Python 3.9.6, pytest-6.2.4, py-1.10.0, pluggy-0.13.1
rootdir: /usr/src/app
plugins: cov-2.12.1
collected 27 items

tests/test_ping.py .                                                        [  3%]
tests/test_summaries.py .............                                       [ 51%]
tests/test_summaries_unit.py .............                                  [100%]

=============================== 27 passed in 0.39s ================================
```

### Parallel Test Runs

- Next, since we're no longer hitting a database, we can run these tests in parallel with [pytest-xdist](https://github.com/pytest-dev/pytest-xdist).

- Add the package to the _requirements.txt_ file:
	- `pytest-xdist==2.3.0` 

- Update the containers:
	- `$ docker-compose up -d --build` 

- Run the unit tests in parallel:

```bash
$ docker-compose exec web pytest -k "unit" -n auto

=============================== test session starts ===============================
platform linux -- Python 3.9.6, pytest-6.2.4, py-1.10.0, pluggy-0.13.1
rootdir: /usr/src/app
plugins: cov-2.12.1, xdist-2.3.0, forked-1.3.0
gw0 [13] / gw1 [13] / gw2 [13] / gw3 [13] / gw4 [13] / gw5 [13] / gw6 [13] / gw7 [13]
.............                                                               [100%]
=============================== 13 passed in 2.78s ================================
```

> We'll continue to run all tests locally throughout the rest of this course. That said, you are more than welcome to just run the unit tests locally and run all tests on GitHub.



## Text Summarization

In this chapter, we'll develop the text summarization piece of our application.

---

### Newspaper3k

- Our objective is to create a real-time text summarization service used for creating article summaries from a given URL.

- We'll need to use a web scraping library to extract the text of an article and then use a machine learning library to generate the actual summary. We could roll our own solution with say [Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) (for scraping) and [Bart](https://huggingface.co/transformers/model_doc/bart.html) (for summarizing), but there's a library that will do both: [Newspaper3k](https://newspaper.readthedocs.io/).

- So, start by adding the library to the requirements file:
	- `newspaper3k==0.2.8` 

- Install:
	- `$ docker-compose up -d --build`

### Summarizer

- Add a new file called _project/app/summarizer.py_:

```python
# project/app/summarizer.py

from newspaper import Article

def generate_summary(url: str) -> str:
    article = Article(url)
    article.download()
    article.parse()
    article.nlp()
    return article.summary
```

That's it.

- After creating a new `Article` instance, we downloaded the given URL's HTML, extracted the meaningful content (via `parse()`) and the relevant keywords (via `nlp()`), and then generated a summary.

> For more, review the [Quickstart](https://newspaper.readthedocs.io/en/latest/user_guide/quickstart.html) guide from the official docs.

- The `nlp` method requires the [punkt](https://www.nltk.org/api/nltk.tokenize.html?highlight=punk#module-nltk.tokenize.punkt) tokenizer from the [Natural Language Toolkit](https://www.nltk.org/index.html) (NLTK), so update _summarizer.py_ like so to install it:

```python
# project/app/summarizer.py

import nltk
from newspaper import Article

def generate_summary(url: str) -> str:
    article = Article(url)
    article.download()
    article.parse()

    try:
        nltk.data.find("tokenizers/punkt")
    except LookupError:
        nltk.download("punkt")
    finally:
        article.nlp()

    return article.summary
```

- Next, update the `post` function in _project/app/api/crud.py_ to generate the summary when a new URL is added:

```python
async def post(payload: SummaryPayloadSchema) -> int:
    article_summary = generate_summary(payload.url)
    summary = TextSummary(url=payload.url, summary=article_summary)
    await summary.save()
    return summary.id
```

- Don't forget the import:
	- `from app.summarizer import generate_summary` 

- Test it out:

```bash
$ http --json POST http://localhost:8004/summaries/ url=http://testdriven.io

HTTP/1.1 201 Created
content-length: 37
content-type: application/json
date: Thu, 08 Jul 2021 00:10:38 GMT
server: uvicorn

{
    "id": 1,
    "url": "http://testdriven.io"
}

$ http GET http://localhost:8004/summaries/1/

HTTP/1.1 200 OK
content-length: 779
content-type: application/json
date: Thu, 08 Jul 2021 00:11:02 GMT
server: uvicorn

{
    "created_at": "2021-07-08T00:10:39.390969+00:00",
    "id": 1,
    "summary": "Our courses and tutorials teach practical application of popular tech used by both enterprise and startups.\nYou will quickly become effective in your chosen stack, as we provide pragmatic examples and detailed walkthroughs of the workings of each technology.\nThrough our courses, you will not only become more comfortable with specific tools and technologies like AWS ECS, Docker, and Flask, to name a few -- but you will also gain skills necessary to contribute to a team or launch your own exciting project.\nEvery tool taught in our courses is supported by large communities and is in high-demand by hiring managers around the globe.\nWe teach tools we love and use every day.",
    "url": "http://testdriven.io"
}
```

### Background Task

- The `generate_summary` function introduces a blocking operation into the `post` function. Since both `article.parse()` and `article.nlp()` are expensive operations, let's move the entire `generate_summary` function out of the request/response flow by running it as a [background task](https://fastapi.tiangolo.com/tutorial/background-tasks/).

- First, update `post`:

```python
async def post(payload: SummaryPayloadSchema) -> int:
    summary = TextSummary(url=payload.url, summary="")
    await summary.save()
    return summary.id
```

- Next, update `create_summary`:

```python
@router.post("/", response_model=SummaryResponseSchema, status_code=201)
async def create_summary(payload: SummaryPayloadSchema, background_tasks: BackgroundTasks) -> SummaryResponseSchema:
    summary_id = await crud.post(payload)

    background_tasks.add_task(generate_summary, summary_id, payload.url)

    response_object = {"id": summary_id, "url": payload.url}
    return response_object
```

- Make sure to import both `BackgroundTasks` and `generate_summary`:

```python
from fastapi import APIRouter, HTTPException, Path, BackgroundTasks

from app.summarizer import generate_summary
```

- Finally, update `generate_summary` like so:

```python
# project/app/summarizer.py

import nltk
from newspaper import Article

from app.models.tortoise import TextSummary

async def generate_summary(summary_id: int, url: str) -> None:
    article = Article(url)
    article.download()
    article.parse()

    try:
        nltk.data.find("tokenizers/punkt")
    except LookupError:
        nltk.download("punkt")
    finally:
        article.nlp()

    summary = article.summary

    await TextSummary.filter(id=summary_id).update(summary=summary)
```

- Now, the summary is generated _after_ the response is sent back to the client. Once generated, the database is updated. To test, add `asyncio.sleep` to generate_summary:

```python
# project/app/summarizer.py

import asyncio

import nltk
from newspaper import Article

from app.models.tortoise import TextSummary

async def generate_summary(summary_id: int, url: str) -> None:
    article = Article(url)
    article.download()
    article.parse()

    try:
        nltk.data.find("tokenizers/punkt")
    except LookupError:
        nltk.download("punkt")
    finally:
        article.nlp()

    summary = article.summary

    await asyncio.sleep(10)

    await TextSummary.filter(id=summary_id).update(summary=summary)
```

- Then, add a new summary:

```bash
$ http --json POST http://localhost:8004/summaries/ url=http://testdriven.io

HTTP/1.1 201 Created
content-length: 34
content-type: application/json
date: Sun, 10 May 2020 15:59:54 GMT
server: uvicorn

{
    "id": 5,
    "url": "http://testdriven.io"
}
```

- Get the summary:

```bash
$ http GET http://localhost:8004/summaries/5/

HTTP/1.1 200 OK
content-length: 134
content-type: application/json
date: Sun, 10 May 2020 16:00:09 GMT
server: uvicorn

{
    "created_at": "2020-05-10T15:59:55.098074",
    "id": 5,
    "summary": "",
    "url": "http://testdriven.io"
}
```

- Wait about ten seconds before getting the summary again:

```bash
$ http GET http://localhost:8004/summaries/5/

HTTP/1.1 200 OK
content-length: 134
content-type: application/json
date: Sun, 10 May 2020 16:00:09 GMT
server: uvicorn

{
    "created_at": "2020-05-10T15:59:55.098074",
    "id": 5,
    "summary": "Our courses and tutorials teach practical application of popular tech used by both enterprise and startups.\nYou will quickly become effective in your chosen stack, as we provide pragmatic examples and detailed walkthroughs of the workings of each technology.\nThrough our courses, you will not only become more comfortable with specific tools and technologies like AWS ECS, Docker, and Flask, to name a few -- but you will also gain skills necessary to contribute to a team or launch your own exciting project.\nEvery tool taught in our courses is supported by large communities and is in high-demand by hiring managers around the globe.\nWe teach tools we love and use every day.",
    "url": "http://testdriven.io"
}
```

- Be sure to remove `asyncio.sleep` once done.

### Tests

- Do the tests pass?
	- `$ docker-compose exec web python -m pytest` 

- You should see six failures:

```bash
>           raise ArticleException('Article `download()` failed with %s on URL %s' %
                  (self.download_exception_msg, self.url))
E           newspaper.article.ArticleException: Article `download()` failed with HTTPSConnectionPool(host='foo.bar', port=443): Max retries exceeded with url: / (Caused by NewConnectionError('<urllib3.connection.HTTPSConnection object at 0x7f00e5582e50>: Failed to establish a new connection: [Errno -2] Name or service not known')) on URL https://foo.bar`
```

- Let's use `monkeypatch` again.

- Example:

```python
def test_create_summary(test_app_with_db, monkeypatch):
    def mock_generate_summary(summary_id, url):
        return None
    monkeypatch.setattr(summaries, "generate_summary", mock_generate_summary)

    response = test_app_with_db.post(
        "/summaries/", data=json.dumps({"url": "https://foo.bar"})
    )

    assert response.status_code == 201
    assert response.json()["url"] == "https://foo.bar"
```

- Add the import:
	- `from app.api import summaries` 

- Fix the other failing tests on your own.

- Run Flake8, Black, and isort:

```bash
$ docker-compose exec web flake8 .
$ docker-compose exec web black .
$ docker-compose exec web isort .
```

- Commit your code and push it to GitHub. Test the summarizer on Heroku.


## Advanced CI

In this chapter, we'll update the CI process and set up a multistage Docker build for production.

---

### Multistage Docker Build

- Start by updating Dockerfile.prod like so:

```dockerfile
###########
# BUILDER #
###########

# pull official base image
FROM python:3.8.11-slim-buster as builder

# install system dependencies
RUN apt-get update \
  && apt-get -y install gcc postgresql \
  && apt-get clean

# set work directory
WORKDIR /usr/src/app

# set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# install dependencies
RUN pip install --upgrade pip
COPY ./requirements.txt .
RUN pip wheel --no-cache-dir --no-deps --wheel-dir /usr/src/app/wheels -r requirements.txt

# lint
COPY . /usr/src/app/
RUN pip install black==21.6b0 flake8==3.9.2 isort==5.9.1
RUN flake8 .
RUN black --exclude=migrations .
RUN isort .


#########
# FINAL #
#########

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
COPY --from=builder /usr/src/app/wheels /wheels
COPY --from=builder /usr/src/app/requirements.txt .
RUN pip install --upgrade pip
RUN pip install --no-cache /wheels/*
RUN pip install "uvicorn[standard]==0.14.0"

# add app
COPY . .

# chown all the files to the app user
RUN chown -R app:app $HOME

# change to the app user
USER app

# run gunicorn
CMD gunicorn --bind 0.0.0.0:$PORT app.main:app -k uvicorn.workers.UvicornWorker
```

- Here, we used a Docker [multistage](https://stackoverflow.com/a/53101932/1799408) build to reduce the final image size. Essentially, `builder` is a temporary image that's used for building the Python wheels. The wheels are then copied over to the final production image and the builder image is discarded.

- Did you notice that Flake8, Black, and isort are being run in the `builder` image? If any of them fail, the build stops.

- To test locally, build the new image and spin up the container:

```bash
$ docker build -f project/Dockerfile.prod -t web ./project

$ docker run --name fastapi-tdd -e PORT=8765 -e DATABASE_URL=sqlite://sqlite.db -p 5003:8765 web:latest
```

- You should see something similar to:

```bash
[2021-07-08 11:54:28 +0000] [7] [INFO] Starting gunicorn 20.1.0
[2021-07-08 11:54:28 +0000] [7] [INFO] Listening at: http://0.0.0.0:8765 (7)
[2021-07-08 11:54:28 +0000] [7] [INFO] Using worker: uvicorn.workers.UvicornWorker
[2021-07-08 11:54:28 +0000] [9] [INFO] Booting worker with pid: 9
[2021-07-08 11:54:28 +0000] [9] [INFO] Started server process [9]
[2021-07-08 11:54:28 +0000] [9] [INFO] Waiting for application startup.
[2021-07-08 11:54:28 +0000] [9] [INFO] Application startup complete.
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

- Commit and push your code up to GitHub to trigger a new build. Ensure it passes.

### Docker Caching

- Next, to speed up the build on GitHub Actions, update the _.github/workflows/main.yml_ file:

```yaml
name: Continuous Integration and Delivery

on: [push]

env:
  IMAGE: docker.pkg.github.com/$(echo $GITHUB_REPOSITORY | tr '[A-Z]' '[a-z]')/summarizer

jobs:

  build:
    name: Build Docker Image
    runs-on: ubuntu-latest
    steps:
      - name: Checkout master
        uses: actions/checkout@v2.3.4
      - name: Log in to GitHub Packages
        run: echo ${GITHUB_TOKEN} | docker login -u ${GITHUB_ACTOR} --password-stdin docker.pkg.github.com
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      - name: Pull images
        run: |
          docker pull ${{ env.IMAGE }}-builder:latest || true
          docker pull ${{ env.IMAGE }}-final:latest || true
      - name: Build images
        run: |
          docker build \
            --target builder \
            --cache-from ${{ env.IMAGE }}-builder:latest \
            --tag ${{ env.IMAGE }}-builder:latest \
            --file ./project/Dockerfile.prod \
            "./project"
          docker build \
            --cache-from ${{ env.IMAGE }}-final:latest \
            --tag ${{ env.IMAGE }}-final:latest \
            --file ./project/Dockerfile.prod \
            "./project"
      - name: Push images
        run: |
          docker push ${{ env.IMAGE }}-builder:latest
          docker push ${{ env.IMAGE }}-final:latest

  test:
    name: Test Docker Image
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Checkout master
        uses: actions/checkout@v2.3.4
      - name: Log in to GitHub Packages
        run: echo ${GITHUB_TOKEN} | docker login -u ${GITHUB_ACTOR} --password-stdin docker.pkg.github.com
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      - name: Pull images
        run: |
          docker pull ${{ env.IMAGE }}-builder:latest || true
          docker pull ${{ env.IMAGE }}-final:latest || true
      - name: Build images
        run: |
          docker build \
            --target builder \
            --cache-from ${{ env.IMAGE }}-builder:latest \
            --tag ${{ env.IMAGE }}-builder:latest \
            --file ./project/Dockerfile.prod \
            "./project"
          docker build \
            --cache-from ${{ env.IMAGE }}-final:latest \
            --tag ${{ env.IMAGE }}-final:latest \
            --file ./project/Dockerfile.prod \
            "./project"
      - name: Run container
        run: |
          docker run \
            -d \
            --name fastapi-tdd \
            -e PORT=8765 \
            -e ENVIRONMENT=dev \
            -e DATABASE_TEST_URL=sqlite://sqlite.db \
            -p 5003:8765 \
            ${{ env.IMAGE }}-final:latest
      - name: Pytest
        run: docker exec fastapi-tdd python -m pytest .
      - name: Flake8
        run: docker exec fastapi-tdd python -m flake8 .
      - name: Black
        run: docker exec fastapi-tdd python -m black . --check
      - name: isort
        run: docker exec fastapi-tdd python -m isort . --check-only

  deploy:
    name: Deploy to Heroku
    runs-on: ubuntu-latest
    needs: [build, test]
    env:
      HEROKU_APP_NAME: thawing-beach-87010
      HEROKU_REGISTRY_IMAGE: registry.heroku.com/${HEROKU_APP_NAME}/summarizer
    steps:
      - name: Checkout master
        uses: actions/checkout@v2.3.4
      - name: Log in to GitHub Packages
        run: echo ${GITHUB_TOKEN} | docker login -u ${GITHUB_ACTOR} --password-stdin docker.pkg.github.com
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      - name: Pull images
        run: |
          docker pull ${{ env.IMAGE }}-builder:latest || true
          docker pull ${{ env.IMAGE }}-final:latest || true
      - name: Build images
        run: |
          docker build \
            --target builder \
            --cache-from ${{ env.IMAGE }}-builder:latest \
            --tag ${{ env.IMAGE }}-builder:latest \
            --file ./project/Dockerfile.prod \
            "./project"
          docker build \
            --cache-from ${{ env.IMAGE }}-final:latest \
            --tag ${{ env.IMAGE }}:latest \
            --tag ${{ env.HEROKU_REGISTRY_IMAGE }}:latest \
            --file ./project/Dockerfile.prod \
            "./project"
      - name: Log in to the Heroku Container Registry
        run: docker login -u _ -p ${HEROKU_AUTH_TOKEN} registry.heroku.com
        env:
          HEROKU_AUTH_TOKEN: ${{ secrets.HEROKU_AUTH_TOKEN }}
      - name: Push to the registry
        run: docker push ${{ env.HEROKU_REGISTRY_IMAGE }}:latest
      - name: Set environment variables
        run: |
          echo "HEROKU_REGISTRY_IMAGE=${{ env.HEROKU_REGISTRY_IMAGE }}" >> $GITHUB_ENV
          echo "HEROKU_AUTH_TOKEN=${{ secrets.HEROKU_AUTH_TOKEN }}" >> $GITHUB_ENV
      - name: Release
        run: |
          chmod +x ./release.sh
          ./release.sh
```

- We're now pulling, building, and pushing the `builder` image using the `--target` [option](https://docs.docker.com/compose/compose-file/#target).

> For more on Docker caching, review the [Faster CI Builds with Docker Cache](https://testdriven.io/blog/faster-ci-builds-with-docker-cache/) post.

### Update Requirements

- Next, let's move development-only requirements to a new file.

_requirements-dev.txt_:

```txt
black==21.6b0
flake8==3.9.2
isort==5.9.1
pytest==6.2.4
pytest-cov==2.12.1
pytest-xdist==2.3.0

-r requirements.txt
```

_requirements.txt_:

```txt
aerich==0.5.3
asyncpg==0.23.0
fastapi==0.65.3
gunicorn==20.1.0
newspaper3k==0.2.8
requests==2.25.1
tortoise-orm==0.17.4
uvicorn==0.14.0
```

- Update _Dockerfile_:

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
COPY ./requirements-dev.txt .
RUN pip install -r requirements-dev.txt

# add app
COPY . .

# add entrypoint.sh
COPY ./entrypoint.sh .
RUN chmod +x /usr/src/app/entrypoint.sh

# run entrypoint.sh
ENTRYPOINT ["/usr/src/app/entrypoint.sh"]
```

- Test:

```bash
$ docker-compose down -v
$ docker-compose up -d --build
$ docker-compose exec web aerich upgrade

$ docker-compose exec web python -m pytest
$ docker-compose exec web flake8 .
$ docker-compose exec web black .
$ docker-compose exec web isort .
```

- Install the requirements in the `test` job of _.github/workflows/main.yml_:

```yaml
- name: Install requirements
  run: docker exec fastapi-tdd pip install black==21.6b0 flake8==3.9.2 isort==5.9.1 pytest==6.2.4

Full job:

`test:
  name: Test Docker Image
  runs-on: ubuntu-latest
  needs: build
  steps:
    - name: Checkout master
      uses: actions/checkout@v2.3.4
    - name: Log in to GitHub Packages
      run: echo ${GITHUB_TOKEN} | docker login -u ${GITHUB_ACTOR} --password-stdin docker.pkg.github.com
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    - name: Pull images
      run: |
        docker pull ${{ env.IMAGE }}-builder:latest || true
        docker pull ${{ env.IMAGE }}-final:latest || true
    - name: Build images
      run: |
        docker build \
          --target builder \
          --cache-from ${{ env.IMAGE }}-builder:latest \
          --tag ${{ env.IMAGE }}-builder:latest \
          --file ./project/Dockerfile.prod \
          "./project"
        docker build \
          --cache-from ${{ env.IMAGE }}-final:latest \
          --tag ${{ env.IMAGE }}-final:latest \
          --file ./project/Dockerfile.prod \
          "./project"
    - name: Run container
      run: |
        docker run \
          -d \
          --name fastapi-tdd \
          -e PORT=8765 \
          -e ENVIRONMENT=dev \
          -e DATABASE_TEST_URL=sqlite://sqlite.db \
          -p 5003:8765 \
          ${{ env.IMAGE }}-final:latest
    - name: Install requirements
      run: docker exec fastapi-tdd pip install black==21.6b0 flake8==3.9.2 isort==5.9.1 pytest==6.2.4
    - name: Pytest
      run: docker exec fastapi-tdd python -m pytest .
    - name: Flake8
      run: docker exec fastapi-tdd python -m flake8 .
    - name: Black
      run: docker exec fastapi-tdd python -m black . --check
    - name: isort
      run: docker exec fastapi-tdd python -m isort . --check-only
```

- Again, commit and push your code up to GitHub to trigger a new build. Make sure it passes before moving on.

## Workflow

> Reference Guide.

### Alias

- To save some precious keystrokes, let's create an alias for the `docker-compose` command -- `dc`.

- Simply add the following line to your _.bashrc_ file:
	- `alias dc='docker-compose'` 

- Save the file, then execute it:
	- `$ source ~/.bashrc` 

- Test it out!

> On Windows? You will first need to create a [PowerShell Profile](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_profiles?view=powershell-7#how-to-use-a-profile) (if you don't already have one), and then you can add the alias to it using [Set-Alias](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/set-alias?view=powershell-6): `Set-Alias dc docker-compose`.


### Common Commands

- Build the images:
	- `$ docker-compose build` 

- Run the containers:
	- `$ docker-compose up -d` 

- Apply the migrations:

```bash
$ docker-compose exec web aerich upgrade

# prefer just to apply the latest changes to the database, without the migrations?
# $ docker-compose exec web python app/db.py
```

- Run the tests:
	- `$ docker-compose exec web python -m pytest` 

- Run the tests with coverage:
	- `$ docker-compose exec web python -m pytest --cov="."` 

- Lint:
	- `$ docker-compose exec web flake8 .` 

- Run Black and isort with check options:

```bash
$ docker-compose exec web black . --check
$ docker-compose exec web isort . --check-only
```

- Make code changes with Black and isort:

```bash
$ docker-compose exec web black .
$ docker-compose exec web isort .
```

### Other Commands

- To stop the containers:
	- `$ docker-compose stop` 

- To bring down the containers:
	- `$ docker-compose down` 

- Want to force a build?
	- `$ docker-compose build --no-cache` 

- Remove images:
	- `$ docker rmi $(docker images -q)`


### Postgres

- Want to access the database via psql?
	- `$ docker-compose exec web-db psql -U postgres` 

- Then, you can connect to the database and run SQL queries. For example:

```sql
# \c web_dev
# select * from textsummary;
```


## Structure

For reference, your project structure should look like this:

```markdown
├── .github
│   └── workflows
│       └── main.yml
├── .gitignore
├── README.md
├── docker-compose.yml
├── project
│   ├── .coverage
│   ├── .coveragerc
│   ├── .dockerignore
│   ├── Dockerfile
│   ├── Dockerfile.prod
│   ├── aerich.ini
│   ├── app
│   │   ├── __init__.py
│   │   ├── api
│   │   │   ├── __init__.py
│   │   │   ├── crud.py
│   │   │   ├── ping.py
│   │   │   └── summaries.py
│   │   ├── config.py
│   │   ├── db.py
│   │   ├── main.py
│   │   ├── models
│   │   │   ├── __init__.py
│   │   │   ├── pydantic.py
│   │   │   └── tortoise.py
│   │   └── summarizer.py
│   ├── db
│   │   ├── Dockerfile
│   │   └── create.sql
│   ├── entrypoint.sh
│   ├── htmlcov
│   ├── migrations
│   │   └── models
│   │       └── 1_20210704142846_None.sql
│   ├── requirements-dev.txt
│   ├── requirements.txt
│   ├── setup.cfg
│   └── tests
│       ├── __init__.py
│       ├── conftest.py
│       ├── test_ping.py
│       ├── test_summaries.py
│       └── test_summaries_unit.py
└── release.sh
```

- You can find the source code in the [fastapi-tdd-docker](https://github.com/testdrivenio/fastapi-tdd-docker) repo on GitHub.



## Next Steps

We've covered a lot in this course. Take a few minutes to step back and reflect on what you've learned. Ask yourself the following questions:

1.  What were my objectives?
2.  How far did I get? How much is left?
3.  How was the process? Am I on the right track?

Objectives:

1.  Develop an asynchronous RESTful API with Python and FastAPI
2.  Practice Test-Driven Development
3.  Test a FastAPI app with pytest
4.  Interact with a Postgres database asynchronously
5.  Containerize FastAPI and Postgres inside a Docker container
6.  Run unit and integration tests with code coverage inside a Docker container
7.  Check your code for any code quality issues via a linter
8.  Configure GitHub Actions for continuous integration and deployment
9.  Use GitHub Packages to store Docker Images
10.  Speed up a Docker-based CI build with Docker Cache
11.  Deploy FastAPI, Uvicorn, and Postgres to Heroku with Docker
12.  Parameterize test functions and mock functionality in tests with pytest
13.  Run tests in parallel with pytest-xdist
14.  Document a RESTful API with Swagger/OpenAPI
15.  Run a background process outside the request/response flow

---

Now it's your turn! Spend some time refactoring and dealing with tech debt on your own.

Some ideas:

1.  **Test coverage**: Add more tests to increase the overall test coverage.
2.  **DRY out the code**: There's plenty of places in the code base that could be refactored.
3.  **CORS Middleware**: Use [CORSMiddleware](https://fastapi.tiangolo.com/tutorial/cors/) to handle cross-origin requests -- e.g., requests that originate from a different protocol, IP address, domain name, or port to prevent [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)-based attacks.
4.  **Security Vulnerability Scanners**: Along with checking for code quality and style, linters can also be used for finding security vulnerabilities. Scan your-
    -   Code base with [Bandit](https://bandit.readthedocs.io/)
    -   Dependencies with [Safety](https://pyup.io/safety/)
    -   Docker images with [Trivy](https://github.com/aquasecurity/trivy)
5.  **Third-Party Extensions**: Try out some of the extensions from the [Awesome FastAPI](https://github.com/mjhea0/awesome-fastapi) repo. Can't find what you need? Write your own!