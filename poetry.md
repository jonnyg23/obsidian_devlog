---
tags: 💽
aliases: 
  - poetry
cssclass:
---

# [[poetry]]

---

> [Poetry](https://python-poetry.org/docs/basic-usage/) allows you to externally maintain Python packages and run scripts as you usually would using Makefiles. 

## Perks

- No more using `venv` or `virtualenv` directly! 🎉
- No manual virtual environment activation is required! ⭐
	- `poetry run` takes care of this
- Gone are the days of **requirements.txt** files
- Package Management
	- Dependency Resolution
	- Version Locking
	- Separation of Environments
	- Dev vs. Prod Dependencies

## Setup Poetry

1. [Install pipx](https://pipx.pypa.io/stable/#on-macos:~:text=the%20limitations%20there.-,On%20macOS,-brew%20install%20pipx)
2. [Install poetry](https://python-poetry.org/docs/) with `pipx install poetry`
	1. If you have it installed then update with `pipx upgrade poetry`

## Example Commands

### Integrate Poetry With Existing Project

```zsh
# Creates pyproject.toml & poetry.lock files
# Note: It'll walk you through setting up the project details
poetry init
```

**Note:** If you only need poetry for dependency management and not package management, you can do the following:
- Add `package-mode = false` to the [tool.poetry] section of your **pyproject.toml**. This will make `name` and `version` optional metadata.

### Workflow Commands

```bash
# Set this before anything to ensure your virtual environment 
# created is local in your project
poetry config virtualenvs.in-project true

# Add a package as dependency to your project
poetry add <package-name>

# Add pip package that are only used for development
poetry add <package-name> --dev

# Run a Python script
# (refer to '[tool.poetry.scripts]' section of your pyproject.toml file for specific functions)
poetry run lint

# Install packages and create an environment based on project specs
# Reads dependencies and versions from poetry.lock file
# Sets up and recreates a virtual environment
poetry install 

# Anytime you update the pyproject.toml file you need to update the lock file
# You can do this with:
poetry lock --no-update
```

#### Notes

- `poetry install` reminds me of Javascript `npm install` for installing node modules from a package.json file.
- `poetry add` builds project's "blueprint" of dependencies.
- `poetry install` constructs the actual working environment from that blueprint.

## Supporting Multiple Python Versions

**Specify Python Version:**
In your pyproject.toml file, you can indicate the range of Python versions your project supports:
```
[tool.poetry.dependencies]
python = "^3.8"  # Supports 3.8 and newer
```

**Virtual Environments per Python Version**
Poetry can automatically create virtual environments for different Python versions as needed. 

**Per-Environment Installation**
To install your project into a specific environment, use the `poetry env use` command. For example:
```bash
poetry env use python3.10
poetry install
```



🔗 Links to this page:
[[Python]]
[[virtualenv|venv]]
[[pyenv]]