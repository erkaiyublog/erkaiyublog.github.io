---
published: true
title: A Brief Review of Git Operations 
tags: git
---

---
_As a popular SVN, Git appears in almost every developper's toolkit, regardless of what type of projects they are working on. Git provides us with a convenient way to keep track of all the changes made to a project, so that we can easily revert the project back to certain states if errors occur._

_In this post, I'd like to take down some notes of some common git operations for furture reference. This post is meant to serve as a quick lookup guide for git users instead of a detailed introduction for beginners._

---

***Table of Contents***
* TOC
{:toc}

# Create a Local Repository
Suppose you have a folder called **My_Project** on your local operating system locating at "*~/My_Project*", to turn it into a git repository,

    ~/My_Project# git init

# Clone a Remote Repository
To clone a git repository, we need either its http link or its SSH link. For example, the source code of this blog website is placed on **github**, to clone it, you can use 

    ~# git clone https://github.com/erkaiyublog/erkaiyublog.github.io.git

or 

    ~# git clone git@github.com:erkaiyublog/erkaiyublog.github.io.git

To specify the branch to be cloned, and name the local folder differently,

    ~# git clone <http link/ssh link> -b <branch name> <local folder name>

# Notation (IMPORTANT)

To make things clear, in the later sections, I will just assume that you are working on a Git repository called "**My_Project**" locating at "*~/My_Project*" on your system. I will add a little notation behind the path to specify the Git branch you're working on.

For example, by default, you will be working on a Git branch called **main** (sometimes it's called **master**), so I will write the following prompt notation to represent that

    ~/My_Project(main)# 

That means I ASSUME that your terminal looks like this. So when you are typing commands like ***ls***, it appears as

    ~/My_Project(main)# ls

Your real prompt probably looks different with my notation, but that doesn't matter. Just keep in mind that my notation is to help you understand where should you use the commands I provided.

# Git Config

Git will force you to config before making the first commit.

    ~/My_Project(main)# git config user.email "<your email>"
    ~/My_Project(main)# git config user.name "<your name>"

Note that the email and name provided here are not required to be matched with the ones you used to login **github** or **gitlab**, these are just used as records instead of login authorization. 

Alternatively, you can add a ***--global*** flag to make the Git configuration "global" on your system, so that all the other Git repositories you created on your system will by default use this pair of email and name.

    ~/My_Project(main)# git config --global user.email "<your email>"
    ~/My_Project(main)# git config --global user.name "<your name>"

# Basic Operations on a Single Main Branch 
### Git Status
Git status shows the status of the Git repository. It lists the changes worth noticing in different sectinos. For example, 

    ~/My_Project(main)# git status
    On branch main
    Changes to be committed:
      (use "git restore --staged <file>..." to unstage)
	    new file:   text1.txt

    Untracked files:
      (use "git add <file>..." to include in what will be committed)
	    text2.txt

Following are some sections that might appear in the result of ***git status***

1. Untracked files: Git didn't track this file.
2. Changes to be committed: The file has been tracked, but need ***git commit*** to record this change as a commit in the Git repository.
3. Changes not staged for commit: The file has been tracked, some changes has been made to the file, these changes HAVE NOT been tracked. Need to track this file before ***git commit*** to record this change as a commit in the Git repository.

### Git Add
Track specified file(s) or folder(s),

    ~/My_Project(main)# git add <file1> <file2> ... <folder1> <folder2> ...

Track all changed files,

    ~/My_Project(main)# git add .

### Git Commit
Commit the staged changes (remember to use the ***-m*** flag and leave an non-empty comment for the commit), 

    ~/My_Project(main)# git commit -m "<comments of this commit>"

### Git Push
**MAKE SURE** there's no conflict on the remote branch (meaning there's commit on the remote branch which you don't have on the local branch, can be verified with ***git log***, otherwise you need to use ***git fetch*** before pushing).
    
    ~/My_Project(main)# git push origin

If there are commits on the REMOTE **main** branch, here's what you should do,

    ~/My_Project(main)# git fetch origin
    ~/My_Project(main)# git merge origin/main

If there are conflicts, 

    ~/My_Project(main)# git add <conflicted_file1> <conflicted_file2> ... 
    ~/My_Project(main)# git commit -m "<comment about this merge>"
    ~/My_Project(main)# git push origin   

### Git Fetch
If there are commits on the remote repository, specifically, on the **origin/main** branch, then AFTER you have committed your local changes, do 

    ~/My_Project(main)# git fetch origin

This fetches the remote main branch, namely, the **origin/main** branch, now it's no longer the same as your local **main** branch. 

### Git Merge
In the case of only a **main** branch exists, ***git merge*** is used to synchronize the local branch with the remote one after ***git fetch origin***. 

    ~/My_Project(main)# git merge origin/main

If there're any conflicts, Git will specify them and notify you, just edit the conflicted files and then

    ~/My_Project(main)# git add <conflicted_file1> <conflicted_file2> ... 
    ~/My_Project(main)# git commit -m "<comment about this merge>"

### Git Pull
***git pull*** is the COMBINATION of ***git fetch*** and ***git merge***, so in the case of only one branch, with remote **main** containing changes, you have already committed your local changes, then do the following to push,

    ~/My_Project(main)# git pull origin

If there are conflicts, 

    ~/My_Project(main)# git add <conflicted_file1> <conflicted_file2> ... 
    ~/My_Project(main)# git commit -m "<comment about this merge>"
    ~/My_Project(main)# git push origin

# Operations with Multiple Branches
If you're working on a Git repository with multiple branches, then you should make sure that you've already understood all the operations mentioned in the "Operations on a Single Main Branch" section.

### Git Branch
Create a new local branch with 

    ~/My_Project(main)# git branch <new-branch-name>

After creating the branch, you aren't gonna be automatically switched to that branch, you need to use ***git checkout*** command to switch to the **new-branch** by yourself.

If not given a name, ***git branch*** command simply lists the branch names that exist in this Git repository, with an asterisk symbol indicating where you are.

    ~/My_Project(main)# git branch

To delete a branch,

    ~/My_Project(main)# git branch -d <branch-name>

To delete a remote branch

    ~/My_Project(main)# git push origin -d <branch-name>

### Git Checkout
It's a good idea to commit your changes before switching Git branches. Otherwise, Git will try to preserve them on the destination branch IF POSSIBLE. However, if there're conflicts, Git will not allow you to switch branches, in which case you need to either do a ***git commit*** or use ***git stash***.

Switch to an existing Git branch,

    ~/My_Project(main)# git checkout dev
    ~/My_Project(dev)#

Add a ***-b*** flag to create the branch before switching to it,
    
    ~/My_Project(main)# git checkout -b dev
    ~/My_Project(dev)#

On a certain branch, you can checkout an existing commit,

    ~/My_Project(dev)# git checkout <the commit id>

### Git Stash
Stash the TRACKED changes TEMPORARILY, it CLEANS your work directory so that you're ready to switch to other branches,

    ~/My_Project(main)# git stash
    Saved working directory and index state WIP on main: 3e13d94 <last commit comments>

To apply the changes stashed,

    ~/My_Project(main)# git stash apply

If there're multiple stashes,

    ~/My_Project(main)# git stash apply stash@{n}

***n*** is the index of the stash, you can use ***git stash list*** to find out the existing stashes.

To remove the topmost stash,

    ~/My_Project(main)# git stash drop

To clear all stashes,
    
    ~/My_Project(main)# git stash clear

By default, ***git stash*** only stashes the tracked changes, to include the untracked ones, add a ***-u*** flag,

    ~/My_Project(main)# git stash -u

### Git Fetch

Suppose you're on the **main** branch after cloning a remote repository, to switch to the **dev** branch which EXISTS on the remote repository, use 

    ~/My_Project(main)# git fetch origin
    ~/My_Project(main)# git checkout dev
    ~/My_Project(dev)# 

The first "fetch origin" command fetches all the branches from the origin (remote) branch, so that your local Git knows the **dev** branch.

If you've already done the "fetch origin" once and you're now on the **dev** branch, to fetch the remote updates for merging or rebasing, use 

    ~/My_Project(dev)# git fetch origin dev

### Git Merge
If there're updates done on the remote **dev** branch, and you want to apply the updates to your local **dev** branch (which also contains updates), use ***git fetch*** command followed by ***git merge***,

    ~/My_Project(dev)# git fetch origin dev
    ~/My_Project(dev)# git merge origin/dev

If there’re any conflicts, Git will specify them and notify you, just edit the conflicted files and then 

    ~/My_Project(dev)# git add <conflicted_file1> <conflicted_file2> ... 
    ~/My_Project(dev)# git commit -m "<comment about this merge>"

Suppose there're some updates made in **dev**, and now you want to apply them into **main**. The correct way is to start by MERGING **main** into **dev**, solve all the conflicts, and THEN MERGE the **dev** back into **main**.

    ~/My_Project(dev)# git add .
    ~/My_Project(dev)# git commit -m "<comments for changes made on dev branch>"
    ~/My_Project(dev)# git fetch origin dev
    ~/My_Project(dev)# git merge origin/dev
    --- resolve the conflicts if there're any, make a commit for that ---
    ~/My_Project(dev)# git checkout main
    ~/My_Project(main)# git fetch origin main
    ~/My_Project(main)# git merge origin/main
    --- resolve the conflicts if there're any, make a commit for that ---
    ~/My_Project(main)# git checkout dev
    ~/My_Project(dev)# git merge main
    --- resolve the conflicts if there're any, make a commit for that ---
    ~/My_Project(dev)# git checkout main
    ~/My_Project(main)# git merge dev

### Git Rebase
Rewrites the commit history of a branch on top of the latest changes from another branch. When rebasing commmit A based on commit B, all the changes made to reach B starting from A and B's closest common ancester will be applied to A. 

So, if there're updates on a remote **dev** branch, and you want to apply them locally, you can do

    ~/My_Project(dev)# git fetch origin dev
    ~/My_Project(dev)# git rebase origin/dev

Suppose there're some updates made in **dev**, and now you want to apply them into **main**. Other than ***git merge***, you can also use ***git rebase*** to acheive this goal.

    ~/My_Project(dev)# git add .
    ~/My_Project(dev)# git commit -m "<comments for changes made on dev branch>"
    ~/My_Project(dev)# git fetch origin dev
    ~/My_Project(dev)# git rebase origin/dev
    --- resolve the conflicts if there're any, run command `git rebase --continue` after resolving each conflict ---
    ~/My_Project(dev)# git checkout main
    ~/My_Project(main)# git fetch origin main
    ~/My_Project(main)# git rebase origin/main
    --- resolve the conflicts if there're any, run command `git rebase --continue` after resolving each conflict ---
    ~/My_Project(main)# git checkout dev
    ~/My_Project(dev)# git rebase main
    --- resolve the conflicts if there're any, run command `git rebase --continue` after resolving each conflict ---
    ~/My_Project(dev)# git checkout main
    ~/My_Project(main)# git merge dev

To do the Git rebase process interactively, use the ***-i*** flag,

    ~/My_Project(dev)# git rebase -i origin/dev

# Other Useful Operations 
### Git Log
Use ***git log*** command to view the commits made to reach the current **HEAD**,

    ~/My_Project(main)# git log

Some useful flags to make the log more user-friendly,

    ~/My_Project(main)# git log --oneline --graph --author=<name> --decorate


