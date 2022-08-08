---
title: Git commands
---

## Git命令

##### 1. delete all local branches
```shell
git branch --merged | grep -v \* | xargs git branch -D
```
