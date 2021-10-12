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
		- Add an `__init__.py` file to the "app" directory along with a `main.py` file. Within `main.py`, create a new instance of FastAPI and set up a synchronous sanity check route:
			- ```py
			  # project/app/main.py
			  
			  
			  from fastapi import FastAPI
			  
			  app = FastAPI()
			  
			  
			  @app.get("/ping")
			  def pong():
			      return {"ping": "pong!"}
			  ```
			- That's all you need to get a basic route up and running!
			- You should now have:
			  |__ project  
			      |__ app
			         |__ `__init__.py`
			         |__ `main.py`
			- Run the server from the "project" directory: