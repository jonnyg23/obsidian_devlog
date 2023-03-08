---
tags: 💽/🐍
aliases: 
  - pathlib
cssclass:
---

# [[pathlib]]

> Part of the Standard Library; Created around Python 3.4 I believe

---

## Why Use?

- This is a much better alternative to the "old way" of using `os` to determine if file or folder exists.

- The library will structure the path according to the OS you are using. For instance, on macOS, the path separators are `/` whereas on windows they can be `\` or even `\\`


## Path.mkdir

> [Docs Ref](https://docs.python.org/3/library/pathlib.html#pathlib.Path.mkdir)

```python
# Creates folder's and intermediate folders leading to path
Path(FOLDER_PATH).mkdir(parents=True, exist_ok=True)
```


## Path.open

```python
# `.open` can be used similar to the generic Python `.open` method
with Path(FILE_PATH).open(mode="wb") as f:
	f.write(response.body)
```


🔗 Links to this page:
[[Python]]