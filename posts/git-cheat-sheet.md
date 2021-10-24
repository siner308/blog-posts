---
layout: post
title:  "깃 명령어 정리 - Git cheat sheet"
subtitle: "잘 사용하지 않아서 까먹지만 필요하면 써야하는"
author: "Siner"
header-mask:  0.3
tags:
    - git
date:   2021-08-02
multilingual: false
---

## remove untracked files

```bash
$git clean -f         # files only
$git clean -fd        # include directories
```

## ignore already committed files 
```bash
$git rm -r --cached . # remove cache
$git add .            # add without ignored files
```

## remove cached credential
```bash
$git config --global --unset credential.helper
```
