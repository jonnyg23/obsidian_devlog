## Getting Started
	- ### Setup
		- Create a new project and install FastAPI along with [[Uvicorn]], an [[ASGI]] server used to serve up FastAPI: (You may need [[pyenv]] for more details on python environment setup)
		- ```bash
		  $ mkdir fastapi-tdd-docker && cd fastapi-tdd-docker
		  $ mkdir project && cd project
		  $ mkdir app
		  $ python3.9 -m venv env
		  $ source env/bin/activate
		  
		  (env)$ pip install fastapi==0.65.3
		  (env)$ pip install uvicorn==0.14.0
		  ```
		- > Feel free to swap out vitualenv and Pip for [Poetry](https://python-poetry.org/) and [Pipenv](https://pipenv.pypa.io/). For more, review [Modern Python Environments](https://testdriven.io/blog/python-environments/)
		- Add an __init__.py