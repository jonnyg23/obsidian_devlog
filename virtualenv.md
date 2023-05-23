---
tags: 💽
aliases: 
  - virtualenv
  - venv
cssclass:
---

# [[virtualenv]]

>Also known as **venv**
---

## Setting up venv with windows

- `py -3.9 -m venv c:\path\to\wherever\you\want\it` 
- Example:
	- `py -3.9 -m venv venv` will use Python version 3.6 and store the virtual environment in a location called "venv"
	- Ensure you run `pip install --upgrade pip` to update pip

```bash
# activate the virtual environment 
.\venv\Scripts\activate

# check the python version
 python --version

# list all packages installed by default
pip list

# deactivate the virtual environment
deactivate
```

## Setting up venv with MacOS

```bash
# Initialize virtual environment
python3.9 -m venv .venv

# Enter the venv
source .venv/bin/activate

# Update pip
pip install --upgrade pip

# Connect to remote aws pip
./connect_pip.sh
```


🔗 Links to this page:
[[Python]]