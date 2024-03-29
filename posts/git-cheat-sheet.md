---
title: "깃 명령어 정리 - Git cheat sheet"
subtitle: "잘 사용하지 않아서 까먹지만 필요하면 써야하는"
tags:
    - git
    - cheat sheet
date: 2021-08-02
image: https://user-images.githubusercontent.com/34048253/155849835-6ff01102-41c3-4761-87a0-f75c898e0e0e.png
---

## remove ignored files

```bash
$ git clean -dfX
```

> git-clean - Remove untracked files from the working tree<br>
> `-d` for removing directories<br>
> `-f` remove forcefully<br>
> `-n` Don’t actually remove anything, just show what would be done.<br>
> `-X` Remove only files ignored by Git. This may be useful to rebuild everything from scratch, but keep manually created files.

## remove untracked files

```bash
$ git clean -f         # files only
$ git clean -fd        # include directories
```

## ignore already committed files 
```bash
$ git rm -r --cached . # remove cache
$ git add .            # add without ignored files
```

## remove cached credential
```bash
$ git config --global --unset credential.helper
```
## remove all fetched branches
This will prune any branches that no longer exist on the remote.

```bash
$ git fetch origin --prune
```

## remove all local branches
```bash
$ git branch -d $(git branch)
```

## remove all git stashes
```bash
$ git stash clear
```
