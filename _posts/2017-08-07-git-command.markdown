---
layout: post
title: "learn git command"
date: 2018-08-07 13:07:00 +0800
categories: jekyll update
---

* git init
* git add filename
* git commit -m "text"
* git log
* git reflog
* git reset --hard commit_ID
* git status 查看commit之前的改动
* git diff a_version
* git checkout -- filename
drop change
* git reset HEAD filename
the other way to drop change
* git rm filename
'git rm' means remove a file.
* git clone
* git push
* git pull
* git branch 
show all branch
* git branch branch_name
* git checkout branch_name

* git checkout -b branch_name 
build a new branch and switch to new branch
* git merge branch_name
merge a branch with present branch
* git branch -d branch_name
delete a branch
* git branch -D branch_name
force delete a branch 
* git revert version_name

* git stash
* git stash pop
you can use 'git stash' command to temporary storage your work and fix a bug.
After the bug has be fixed,you can use 'git stash pop' to continue you work.
