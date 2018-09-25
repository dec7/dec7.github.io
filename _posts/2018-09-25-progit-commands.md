---
layout: post
title: "Progit Commands"
description: "Progit Commands"
date: 2018-09-25
tags: [git,command]
comments: true
share: true
---

# Commands
## Ch1
- ```git config --list```  
  - 설정 확인

## Ch2
- ```git status```
  - 파일 상태 확인
- ```git add```
  - 새로운 파일 추적
  - 아규먼트: 파일, 디렉토리 (하위 재귀적 포함)
- ```git init```
- ```git clone```
- ```git diff```
- ```git diff```
  - ```git diff --staged```
- ```git commit```
  - ```git commit -v```
  - ```git commit -m```
  - ```git commit --amend```
- ```git rm```
  - ```git rm -f```
  - ```git rm --cached {filename}```
  - ```git rm log/\*.log```
- ```git log```
- ```git reset HEAD {filename}```
- ```git checkout -- {filename}```
- ```git remote```
  - ```git remote -v```
  - ```git remote add upstream git://github.com/id/repo.git```
  - ```git remote show {remote repo name}```
  - ```git remote rename {before} {after}```
  - ```git remote rm {repo name}```
- ```git fetch {repositoryname}```
- ```git pull upstream {branchname}```
- ```git push origin master```
- ```git tag```
  - ```git tag -l 'v1.4.2*'```
  - ```git tag v1.0 ```
  - ```git tag -a v1.0 -m {message}```
  - ```git show v1.0```
  - ```git tag -s v1.0 -m {message}```
  - ```git tag -v v1.0```
  - ```git tag -a v1.2 -m "v1.2" {checksum}```
  - ```git push {remote} {tagname}```
    - ```git push origin --tags```
- ```git config --global alias.unstate 'reset HEAD --'```
- 
