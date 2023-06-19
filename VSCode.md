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
- `recommendations` - VSCode workspace extensions that are recommended for viewing and editing the repo.
	- ✏️ Note: **To see names of installed VSCode extensions, execute the following in the terminal:** `code --list-extensions`
- `python.testing.pytestArgs` - List of folder names that contain pytests.

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

---



Tags:

Reference:

Related:


🔗 Links to this page:

