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

```py
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
```py
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

```py
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
```py
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

```py
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

```txt
fastapi==0.65.3
uvicorn==0.14.0`
```

Finally, add a `.gitignore` to the project root:
```txt
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


