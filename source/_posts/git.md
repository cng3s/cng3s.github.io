---
title: git
date: 2020-03-26 19:59:17
tags: 软件
---
一、开发分支（dev）上的代码达到上线的标准后，要合并到 master 分支
```
    git checkout dev
    git pull
    git checkout master
    git merge dev
    git push -u origin master
```
二、当master代码改动了，需要更新开发分支（dev）上的代码
```
    git checkout master
    git pull
    git checkout dev
    git merge master
    git push -u origin dev
```

# Important commands  
* git init # make a new repository  
* git clone # initialize a repository locally from a remote server  
* git status # MOST IMPORTANT COMMAND
* git log # show commit history. Can use --decorate --graph --all to make it pretty
* git add # stages files to be committed. Flags: --a, -u
* git commit -m # commit the changes in the staged files (use good messages!)
* git push # push changes to a remote server (--set-upstream origin branchname)
* git pull # pull changes from a server
* git branch # make a new branch
* git checkout # switch to a different branch. Can use -b to make a new branch
* git merge name # merge “name” branch into your current branch
* git reset HEAD # Used to unstage files
* git reset --hard + hash # Used to reset to an old commit (with a commit hash)

# Configuring git  
$ git config --global user.name "<Your Name>"   
$ git config --global user.email “<Your Email>”  
$ git config --global push.default simple  
(Make sure the email is your Andrew ID, and make sure to add that email to your GitHub account!)

# Committing, pushing, pulling  
1. $ lswe have 4 files here 
2. $ git status # branch is up to date with the server, nothing to commit
3. $ git log --graph --decorate --all
   * i.Shows a pretty graph of the commit history.
4. $ vim example.txt # lets make some changes to example.txt
5. $ git status # now shows that we have unstaged files
6. $ git add example.txt # stages the file to be committed
7. $ git reset HEAD example.txt # unstages the file (to show you how to do that)
8. $ git add example.txt # to restage the file
9. $ git commit -m  “insert a relevant commit message here”
10. $ git status # shows you are 1 commit ahead of “origin” = remote server
11. $ git push # this updates the remote server
12. $ git log --graph --decorate --all #  now we can see the new commit on top of all the old ones

# Merging
1. $ git log --graph --decorate --all # note the other branch “realistic ending” that branches away from master
2. $ git checkout realistic_ending switch to the other branch
3. $ git branch shows all of our branches
4. $ ls note that there are different files here
5. vim example.txt # we can see the story is different than in the master branch-finish it!
6. Add and commit the file, push to the server.
7. $ git checkout master # switch back to the master branch
8. $ git merge realistic_ending # will attempt to merge the two branches, but there’s a conflict
   1. a.$ git statusshows that the conflict is in example.txt
   2. b.$ vim example.txt fix the story
   3. c.$ git add example.txt
   4. d.$ git commit -m “appropriate message for a merge” now the merge is complete
9. $ git log -- decorate --graph --all # shows that now you still have 2 branches, but they’ve been merged and point to the same files
