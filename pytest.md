---
tags: 💽
aliases: 
  - pytest
cssclass:
---

# [[pytest]]

>This dependency is used for performing unit testing.
---

## Installing

`pip install pytest`


## Running test on specific function inside py test file

`pytest -rP test_file.py::specific_test_method`
NOTE: This will show any print statements used in the test.

## Pytest Discover Fails

> VS Code's Debug/Testing tools not working

- **QUICK FIX:** This occurs because there is an error within one of your test files. More than likely an *import error*. Use the following line in the terminal to view the issues that need fixing: 
	- `pytest --collect-only`



🔗 Links to this page:
[[Python]]
[[Pytest Monkeypatching]]