---
tags: 📝/🌱
aliases:
  -
cssclass:
---

# [[VSCode]]

---

## Creating `settings.json` file

> File usually goes inside of the root directory of the repository under .vscode/**settings.json**

### Typical settings

- `cSpell.words` - Sets commonly used words that are unique and not in the English dictionary.
- [[isort]]`.args` - For sorting imports
	- ✏️ Note:  Ensure [[isort#Setting up auto import sorting in VSCode |follow instructions]] for this to work.
- `editor.defaultFormatter` - Sets the default Python formatter to `black`
- `python.testing.pytestArgs` - List of folder names that contain pytests.
- These settings will automatically set your venv when opening repo similar to manually running `source .venv/bin/activate`
	- `"python.terminal.activateEnvInCurrentTerminal": true` 
	  `"python.defaultInterpreterPath": "~/venv/bin/python"`
	- 🔗 Link: [Stackoverflow - Auto-activate virtual environment in visual studio code](https://stackoverflow.com/questions/58433333/auto-activate-virtual-environment-in-visual-studio-code#:~:text=Actually%20the%20earlier%20suggested%20solutions%20didn%27t%20work%20for%20me%2C%20instead%20I%20added%20the%20following%20in%20my%20settings%3A)

```json
{
	"[python]": {
		"editor.defaultFormatter": "ms-python.black-formatter",
		"editor.codeActionsOnSave": {	
			"source.organizeImports": true
		}
	},
	"python.formatting.provider": "black",
	"isort.args": [
		"--profile",
		"black"
	],
	"cSpell.words": [
		"dtype",
		"dtypes",
		"iloc",
		"isin",
		"isna",
		"iterrows",
		"pytests",
	],
	"python.testing.pytestArgs": [
		"tests"
	],
	"python.testing.unittestEnabled": false,
	"python.testing.pytestEnabled": true,
	"python.terminal.activateEnvInCurrentTerminal": true,
	"python.defaultInterpreterPath": "~/venv/bin/python"
	
}
```

## Creating `extensions.json` file

- Put this file inside of .vscode directory. These are the extensions that vscode will recommend that you use.
	- `recommendations` - VSCode workspace extensions that are recommended for viewing and editing the repo.
	- ✏️ Note: **To see names of installed VSCode extensions, execute the following in the terminal:** 
		- `code --list-extensions`

```json
{
"recommendations": [
		"ms-python.black-formatter",
		"ms-python.flake8",
		"ms-python.isort",
		"ms-python.python",
		"ms-python.vscode-pylance",
		"njpwerner.autodocstring",
		"yzhang.markdown-all-in-one",
		"ZainChen.json"
	]
}
```



Tags:

Reference:

Related:


🔗 Links to this page:

