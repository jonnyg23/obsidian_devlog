---
tags: 💽
aliases: 
  - isort
cssclass:
---

# [[isort]]

---

## Setting up auto import sorting in VSCode

> [Stack Overflow Solution](https://stackoverflow.com/questions/67059648/vscode-how-to-config-organize-imports-for-python-isort#:~:text=You%20can%20make%20isort%20compatible%20with%20black%20and%20enjoy%20a%20formatted%20code%20upon%20saving%20your%20document.%20Here%20are%20two%20ways%20to%20achieve%20this%3A)

1. First, `pip install isort` & install `isort` from VSCode extensions
2. Then, to make `isort` compatible with `black`, create a **pyproject.toml** file at the root of your repo.
3. Add the following to this file:
	```python
	[tool.isort]
	multi_line_output = 3
	include_trailing_comma = true
	force_grid_wrap = 0
	line_length = 88
	profile = "black"
	```
4. Edit /.vscode/**settings.json**  and configure `black` and `isort` by adding:
```json
{
   "[python]": {
    "editor.codeActionsOnSave": {
      "source.organizeImports": true
       }
     },

   "python.formatting.provider": "black",
   "isort.args": ["--profile", "black"],
}
```
5. `source.organizeImports: true` runs `isort` automatically upon saving document.


🔗 Links to this page:
[[Python]]