---
tags: 🖥️
aliases:
  - 
cssclass:
type: language
status: 🟧
---

# [[Python]]

  >Python is the most powerful language you can still read.
  > &nbsp;&nbsp;&nbsp;&nbsp; -Paul Dubois

---

## Environment & Workflow

- [[pyenv]] - Manage python environments
- [[virtualenv|venv]]

### Pip Dependency Installation

Installing pip dependencies from **setup.py** files or from **requirements.txt** files:

```bash
# Ran in root directory where setup.py is located
pip install -e .

# For install from requirements
pip install -r requirements.txt
```

Create **requirements.txt** from current python environment:

```bash
pip freeze > requirements.txt
```

### Pip Dependency Uninstallation

Refer to [StackOverflow Post](https://stackoverflow.com/questions/11248073/how-do-i-remove-all-packages-installed-by-pip) for more information on packages that are installed via VCS or from github/gitlab which have a `@`.

```bash
# Removes all pip packages from python environment
pip freeze | xargs pip uninstall -y
```


## Functions 

### Input Parameters

- [Using `*Args` & `**Kwargs`](https://www.programiz.com/python-programming/args-and-kwargs#:~:text=*args%20passes%20variable%20number%20of,a%20dictionary%20can%20be%20performed.)

## Libraries

### Web Dev

- [[Flask]] - Used for backend REST API dev
- [[gunicorn]] - WSGI HTTP Server for UNIX

### Data Analysis

- [[pandas]] - Powerful data structures for data analysis, time series, and statistics.

## Testing

- [[pytest]] - Run unit tests
- [[tox]] - Runs tests with multiple python versions (3.7, 3.8, 3.9, etc)
- [[logging]] - Logs info related to warnings, debug, info, & exceptions




🔗 Links to this page:
[[Programming Languages]]