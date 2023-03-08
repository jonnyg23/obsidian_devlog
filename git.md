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


