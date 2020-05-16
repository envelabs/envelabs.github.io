# Git (version control system)
git is a version control system

## Configuration
```
git config --global user.name "<username>"
git config --global user.email "<name>@<domain>.<tld>"
git config --global core.editor "vim"~
```

## Create a new repository locally

```
git init
```

```
git init <dir>
```

## Clone a project from a github repo using HTTPS

```
git clone <https://github.com/USERNAME/REPOSITORY.git>
```

## Clone a project from a github repo using SSH keys

```
git clone git@github.com:USERNAME/REPOSITORY.git
```

## Set remote github repo to use SSH

```
git remote set-url origin git@github.com:USERNAME/REPOSITORY.git
```

## Adding files
check the status of the project
```
git status
```

adding content modified from the current working dir
```
git add .
```
adding specific files
```
git add <file>
```
track changes
```
git commit -m '<message to add to the history of project>'
```

add changes to the history
```
git push
```

## Branching

Create a new branch

```
git branch <new_branch_name>
```

Use branch

```
git checkout <branch_name>
```

Delete local branch

```
git branch -d <branch_name>
```

Delete remote branch

```
git push origin --delete <branch_name>
```
