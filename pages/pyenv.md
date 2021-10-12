- > **pyenv** is used in the workflow for managing python environments:
###
	-
	  1. Installing latest python version in pyenv:
	  ```bash
	  # First check list of pyenv versions available and your versions
	  pyenv install --list
	  
	  pyenv versions
	  
	  # Choose version and install
	  pyenv install -v 3.9.6
	  
	  # If versions not found, try updating pyenv with 
	  brew update && brew upgrade pyenv
	  ```