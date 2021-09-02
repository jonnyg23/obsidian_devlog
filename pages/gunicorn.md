-
  > WSGI HTTP Server for UNIX
## Using with [[Python]]
	- Running a Flask Python backend server: (Activate your pyenv before running the following)
		-
		  ```bash
		  # activate with 
		  source env/bin/activate
		  # end environment with
		  deactivate
		  ```
		  
		  ```bash
		  # After navigating inside the src backend folder that contains your app.py file
		  # Install with pip if you don't have gunicorn yet
		  pip install gunicorn
		  
		  # Run gunicorn on port 4000 with
		  gunicorn -b :5000 app:app
		  
		  # if app.py is actually called api.py then use the following command instead
		  gunicorn -b :5000 api:app
		  ```