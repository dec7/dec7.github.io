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
- ```git add```
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

## ch3
- ```git branch testing```
- ```git checkout testing```
  - ```HEAD```
- ```git checkout -b hotfix```
- ```git branch```
  - ```git branch -v```
  - ```git branch --merged```
    - ```git branch -d {branchname}``` 
  - ```git branch --no-merged```
    - ```git branch -D {branchname}```
- ```git fetch origin```
- ```git push origin serverfix:serverfix```
  - ```refs/heads/serverfix:refs/heads/serverfix```
- ```git merge origin/serverfix```
  - ```git checkout -b serverfix origin/serverfix```
  - ```git checkout -b {branch} {remotename/branch}```
  - ```git checkout --track {remotename/branch}```
- ```git push {remotename} :{branch}```
  - ```git push {remotename} {localbracnh}:{remotebranch}```
- ```git rebase --onto master A B```
    - ```git checkout master, git merge B```
    - ```git rebase master A```
    - ```git checkout master, git merge A```

## ch5
- ```git rebase -i {수정을 시장할 이전커밋}```
- ```git request-pull orgin/master myfork```
- ```git merge --no-commit --squash featureB```
- ```git format-patch```
  - ```git format-patch -M origin/master```
- ```git send-email *.patch```
- ```git apply /tmp/patch-ruby-client.patch```
  - - ```git apply --check```   
- ```git am {patchname}.patch```
  - ```git am --resolved```
  - ```git am -3 {patchname}.patch```
- ```git log contrib --not master```
  - ```git log -p```
  - ```git diff master```
    - ```git merge-base {target-branch} master```
    - ```gif diff {조상커밋}```
- ```git diff master...{target-branch}```
- ```git cherry-pick {checksum}```

## ch6
- ```git log --abbrev-commit```
- ```git rev-parse {branchname}```
- ```git reflog``
- ```git show HEAD@{4}```
- ```git show master@{yesterday}```
- ```git log -g```
- ```git show HEAD^```
- ```git show HEAD~```
- ```git log master..experiment```
  - ```git log refA..refB```
  - ```git log ^refA refB```
  - ```git log refB --not refA```
  - ```git log refA refB ^refC```
  - ```git log refA refB --not refC```
- ```git log master...experiment```
  - ```git log --left-right master...experiment```
- ```git add -i```
- ```git stash```
  - ```git stash list```
  - ```git stash apply```
  - ```git stash drop```
  - ```git stash pop```
  - ```git stash branch {branchname}```
- ```git commit --amend```
- ```git rebase -i HEAD~3```
  - ```git rebase --continue```
- ```git filter-branch --tree-filter 'rm -f password.txt' HEAD```
- ```git filter-branch --subdirectory-filter truck HEAD```
- ```git blame -- {filename}```
- ```git bisect start```
  - ```git bisect bad```
  - ```git bisect good {good commit}```
  - ```git bisect [good|bad]```
  - ```git bisect reset```
- ```git submodule add {repo-url} {directory-name}```
  - ```git diff -cached {directory-name}```
  - ```git submodule init```
  - ```git submodule update```
- ```git read-tree --prefix={directory}/ -u {branch_name}```
- ```git diff-tree -p {branch_name}```

## ch7
- ```git config --global commit.template #HOME/.gitmessage.txt```
- ```git config --global core.pager [''|less|more]```
- ```git config --global user.signingkey {gpg-key_id}```
- ```git config --global color.ui [true|false|always]```
- ```git config --global core.autocrlf true```
- ```git config --global core.whitespace trailing-space,space-before-tab,indent-with-non-tab```
- ```git config --system receive.fsckObjects true```
- ```git config --system receive.denyNonFastForwards true```
- ```git config --system receive.denyDeletes true```

## ch9
```echo 'test content' | git hash-object --stdin```
```git cat-file -p {checksum}```
  - ```git cat-file -t {checksum}```
  - ```git cat-file -p master^{tree}```
- ```git update-index --add --cacheinfo 100644 {checksum} test.txt```
- ```git write-tree```
- ```echo 'first commit' | git commit-tree {checksum}```
- ```git update-ref refs/heads/master {checksum}```
- ```git symbolic-ref HEAD```
  - ```git symbolic-ref HEAD refs/heads/test```
  - ```git update-ref refs/tags/v1.0 {checksum}```
- ```git gc```
- ```git verify-pack -v {idx파일 위치}```
- ```git fsck --full```
- ```git count-objects -v```
- ```git rev-list --objects --all | grep {checksum}```
- ```git filter-branch --index-filter 'git rm --cached --ignore-unmatch git.tbz2' -- {checksum}^..```
- ```git prune --expire```
