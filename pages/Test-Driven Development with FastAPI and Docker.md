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
				- ```bash
				  uvicorn app.main:app
				  ```
				- `app.main:app` tells Uvicorn where it can find the FastAPI application --e.g., "within the 'app' module, you'll find the app, `app = FastAPI()`, in the 'main.py' file.
				- Navigate to http://localhost:8000/ping in your browser. You should see:
					- ```json
					  {
					    "ping": "pong!"
					  }
					  ```
				- > Why did we use Uvicorn to serve up FastAPI rather than a development server?
				  > Unlike Django or Flask, FastAPI does not have a built-in development server. 
				  > New to ASGI? Read through the excellent [Introduction to ASGI: Emergence of an Async Python Web Ecosystem](https://florimond.dev/blog/articles/2019/08/introduction-to-asgi-async-python-web/)
				- FastAPI automatically generates a schema based on the [OpenAPI](https://swagger.io/docs/specification/about/) standard. You can view the raw JSON at http://localhost:8000/openapi.json. This can be used to automatically generate client-side code for a front-end or mobile application. FastAPI uses it along with [Swagger UI](https://github.com/swagger-api/swagger-ui) to create interactive API documentation, which can be viewed at http://localhost:8000/docs:
				-
				  -
				- kill the server
				-
				  -
				-