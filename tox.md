---
tags: 💽
aliases: 
  - tox
cssclass:
---

# [[tox]]

> Used for running unit tests with multiple different versions of python.

---
> [Tox Docs](https://tox.wiki/en/latest/example/basic.html)

## Installation

```bash
pip install tox

# Use this if you want use tox with already created virtual environment
pip install tox tox-current-env
```

## Example of tox.ini

> **RUN WITH:** 
> `tox` (Creates new virtual environment to execute tox)
> `tox --current-env`  (Uses already created *.venv* for running tox) **Must have `tox-current-env` installed**

This generic tox report will use [[Python]] 3.9. It will perform the following:
- Ignore venv and tox directories
- Execute flake8, pytest, isort, and black linting & testing error checks
```ini
[tox]
envlist = py39
ignore = .tox/*,.venv/*

[testenv]
commands =
    flake8 --exclude .tox,.venv --extend-ignore=E501
    pytest
	isort . --skip .tox --skip .venv
	black .
deps =
	pytest
	flake8
	isort
	black

[testenv:flake8]
commands =
	flake8 --exclude .tox,.venv --extend-ignore=E501
deps =
	flake8

[testenv:isort]
commands =
	isort . --skip .tox --skip .venv
deps =
	isort

[testenv:fix]
commands =
	black --diff --quiet .
deps =
	black
```


## Steps to Set Up

- Create **tox.ini** file at same level as repo's **setup.py**

**tox.ini**
```ini
# content of: tox.ini , put in same dir as setup.py
[tox]
envlist = py37,py38,py39,py310
[testenv]
# install testing framework
# ... or install anything else you might need here
deps = pytest
# run the tests
# ... or run any other command line tool you need to run here
commands = pytest
```

- Next run `tox` from terminal within the directory where tox.ini is.
	- This will install virtual environments for each .py version and run the tests.

>Note: The command→`tox -e py38` can be used to specify which tests you want to run. Running `tox` will automatically run all .py versions set in the tox.ini file.

🔗 Links to this page:
[[Python]]