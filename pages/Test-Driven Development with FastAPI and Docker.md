## Getting Started
	- ### Setup
		- Create a new project and install FastAPI along with [[Uvicorn]], an [[ASGI]] server used to serve up FastAPI:
		- ```bash
		  $ mkdir fastapi-tdd-docker && cd fastapi-tdd-docker
		  $ mkdir project && cd project
		  $ mkdir app
		  $ python3.9 -m venv env
		  $ source env/bin/activate
		  
		  (env)$ pip install fastapi==0.65.3
		  (env)$ pip install uvicorn==0.14.0
		  ```