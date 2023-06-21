# Git

> Version control software for projects

## Cloning Repos

In order to clone a private guy repo, run the following in your directory that you wish to clone the private repo:

```bash
git clone https://USERNAME@github.com/USERNAME/repo_name
```

## Adding files to .gitignore that were already committed

1. I used `git rm -r <filenames>`  where the filename is the file that you wish git to stop tracking. 
2. I then ran `git commit -m "CommitMessage"`. 
3. Finally, `git push` pushed the changes to the remote repo on Github.

## Updating local branch when local master is behind remote origin

> This is a situation where the local branch you have been adding to is depending on a local master that is behind remote origin commits.
> Usually, this occurs when others have added code changes to the remote master leaving your local master branch behind.

Before submitting new PR do the following:

1. `git checkout master` your local master branch and `git pull` to get remote master changes.
2. After doing so, `git checkout <feature-branch>` your local feature branch and also run a `git pull`.
3. Then run a `git merge master` (this will merge your feature branch with any recent master branch changes that were pulled in).
4. NOTE: If there are any conflicts with merging in this step, ensure to fix each conflict before it will successfully merge latest master into feature branch.
5. You're done!! Now, test your feature branch before submitting a pull request.


## Remove a Git Branch (Locally)

```bash
git branch --delete BRANCH_NAME
```


## .gitignore file

- .gitignore command [guide](https://www.atlassian.com/git/tutorials/saving-changes/gitignore)

Here is an example **.gitignore** file:

```
# Created by https://www.toptal.com/developers/gitignore/api/python,windows,osx
# Edit at https://www.toptal.com/developers/gitignore?templates=python,windows,osx

### OSX ###

# General
.DS_Store
.AppleDouble
.LSOverride

# Icon must end with two \r
Icon


# Thumbnails
._*

# Files that might appear in the root of a volume
.DocumentRevisions-V100
.fseventsd
.Spotlight-V100
.TemporaryItems
.Trashes
.VolumeIcon.icns
.com.apple.timemachine.donotpresent

# Directories potentially created on remote AFP share
.AppleDB
.AppleDesktop
Network Trash Folder
Temporary Items
.apdisk

### Python ###

# Byte-compiled / optimized / DLL files
__pycache__/
*.py[cod]
*$py.class

# C extensions
*.so

# Distribution / packaging
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
share/python-wheels/
*.egg-info/
.installed.cfg
*.egg
MANIFEST

# PyInstaller
#   Usually these files are written by a python script from a template
#   before PyInstaller builds the exe, so as to inject date/other infos into it.
*.manifest
*.spec

# Installer logs
pip-log.txt
pip-delete-this-directory.txt

# Unit test / coverage reports
htmlcov/
.tox/
.nox/
.coverage
.coverage.*
.cache
nosetests.xml
coverage.xml
*.cover
*.py,cover
.hypothesis/
.pytest_cache/
cover/

# Translations
*.mo
*.pot

# Django stuff:
*.log
local_settings.py
db.sqlite3
db.sqlite3-journal

# Flask stuff:
instance/
.webassets-cache

# Scrapy stuff:
.scrapy

# Sphinx documentation
docs/_build/
  

# PyBuilder
.pybuilder/
target/

# Jupyter Notebook
.ipynb_checkpoints

# IPython
profile_default/
ipython_config.py

# pyenv
#   For a library or package, you might want to ignore these files since the code is
#   intended to run in multiple environments; otherwise, check them in:
# .python-version

# pipenv
#   According to pypa/pipenv#598, it is recommended to include Pipfile.lock in version control.
#   However, in case of collaboration, if having platform-specific dependencies or dependencies
#   having no cross-platform support, pipenv may install dependencies that don't work, or not
#   install all needed dependencies.
#Pipfile.lock


# poetry
#   Similar to Pipfile.lock, it is generally recommended to include poetry.lock in version control.
#   This is especially recommended for binary packages to ensure reproducibility, and is more
#   commonly ignored for libraries.
#   https://python-poetry.org/docs/basic-usage/#commit-your-poetrylock-file-to-version-control
#poetry.lock

  

# PEP 582; used by e.g. github.com/David-OConnor/pyflow
__pypackages__/

# Celery stuff
celerybeat-schedule
celerybeat.pid

# SageMath parsed files
*.sage.py

# Environments
.env
.venv
env/
venv/
ENV/
env.bak/
venv.bak/

# Spyder project settings
.spyderproject
.spyproject

# Rope project settings
.ropeproject

# mkdocs documentation
/site

# mypy
.mypy_cache/
.dmypy.json
dmypy.json

# Pyre type checker
.pyre/

# pytype static type analyzer
.pytype/

# Cython debug symbols
cython_debug/

# PyCharm
#   JetBrains specific template is maintainted in a separate JetBrains.gitignore that can
#   be found at https://github.com/github/gitignore/blob/main/Global/JetBrains.gitignore
#   and can be added to the global gitignore or merged into this file. For a more nuclear
#   option (not recommended) you can uncomment the following to ignore the entire idea folder.
#.idea/

  

### Windows ###

# Windows thumbnail cache files
Thumbs.db
Thumbs.db:encryptable
ehthumbs.db
ehthumbs_vista.db

# Dump file
*.stackdump

# Folder config file
[Dd]esktop.ini

# Recycle Bin used on file shares
$RECYCLE.BIN/

# Windows Installer files
*.cab
*.msi
*.msix
*.msm
*.msp

# Windows shortcuts
*.lnk

# End of https://www.toptal.com/developers/gitignore/api/python,windows,osx
```


## Troubleshooting

### Git Push is frozen

- Fix: Use this in order to increase the buffer size

```bash
git config --global http.postBuffer 157286400
```


### How to remove files that are listed in the .gitignore but still on the repository?

[source: StackOverflow](https://stackoverflow.com/questions/13541615/how-to-remove-files-that-are-listed-in-the-gitignore-but-still-on-the-repositor)
```bash
git rm --cached `git ls-files -i -c --exclude-from=.gitignore`
```
