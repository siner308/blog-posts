---
title: "레디스 명령어 정리 - Redis cheat sheet"
subtitle: "잘 사용하지 않아서 까먹지만 필요하면 써야하는"
tags:
    - database
    - cheat sheet
date: 2021-08-02
image: https://user-images.githubusercontent.com/34048253/155849784-228fb73b-f86f-4848-83bd-29e954327047.png
---

## remove all keys
[flushall](https://redis.io/commands/FLUSHALL)

```bash
redis> flushall
Are you sure (y/n)? y
OK
redis> 
```

## get values by keys (in bash)
```bash
echo 'keys devtalk:qa:viewerCount:*' | redis-cli | sed 's/^/get /' | redis-cli
```
