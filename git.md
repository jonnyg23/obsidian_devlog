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

## Rebasing

> This is used when you have a branch that is referencing a local master which is behind the remote master.

[Link to YT video that helped with this](https://www.youtube.com/watch?v=f1wnYdLEpgI)
Steps to rebase your feature branch:

1. git checkout <local-master-branch> (probably just run `git checkout master`)
2. Once inside the local master branch run `git pull` to get any change that are on the remote master.
3. `git checkout <feature-branch>`
4. `git rebase master`
5. (If there are any conflicts, you can choose each individual change you want to keep. However, if you want to make all of the incoming changes then run `git checkout –theirs <file-name-with-conflict>`)
6. Finally, since your feature branch is up to date with the remote master branch, you can now do your pull requests and commits.