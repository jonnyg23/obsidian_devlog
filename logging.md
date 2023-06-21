---
tags: 💽
aliases: 
  - logging
cssclass:
---

# [[logging]]

---

## [[Python]]

### Using logging

In the main python file of repo (such as repo/source/main.py) add the following:
> ✏️ Note: This can be used in every python file that needs to log data.

```python
import logging

# Sets up a logger
logging.basicConfig() # This line is not usually included adding logging to other files
logger = logging.getLogger(__name__) # Will name the logger the file path
logger.setLevel(logging.INFO)
```

OR you can use the below code for more specific logging other than INFO:

```python
import logging

# Create a logger
logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

# Set up a file handler
file_handler = logging.FileHandler('app.log')
file_handler.setLevel(logging.DEBUG)

# Set up a console handler
console_handler = logging.StreamHandler()
console_handler.setLevel(logging.INFO)

# Define a formatter
formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')

# Add the formatter to the handlers
file_handler.setFormatter(formatter)
console_handler.setFormatter(formatter)

# Add the handlers to the logger
logger.addHandler(file_handler)
logger.addHandler(console_handler)

# Start logging
logger.debug('This is a debug message')
logger.info('This is an info message')
logger.warning('This is a warning message')
logger.error('This is an error message')
logger.critical('This is a critical message')

```



🔗 Links to this page:
