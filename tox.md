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