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

