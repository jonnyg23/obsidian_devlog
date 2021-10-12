- > **pyenv** is used in the workflow for managing python environments:
## Installing new python versions in pyenv
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
## Switching global python version
	- You can find the pyenv versions installed on your computer with the `pyenv versions` command.
	- ```bash
	  pyenv global <VersionNumber>
	  ```
	- To set up pyenv correctly, you can run the following in Bash or zsh: (I had to use this command because `pyenv global` would not change my `python -V` version)
		- ```bash
		  PATH=$(pyenv root)/shims:$PATH
		  ```
## Initializing a new virtual python environment