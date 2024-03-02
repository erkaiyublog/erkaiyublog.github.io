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

## Create a Local Repository
Suppose you have a folder called **My_Project** on your local operating system locating at "*~/My_Project*", to turn it into a git repository,

    ~/My_Project# git init

## Clone a Remote Repository
To clone a git repository, we need either its http link or its SSH link. For example, the source code of this blog website is placed on **github**, to clone it, you can use 

    ~# git clone https://github.com/erkaiyublog/erkaiyublog.github.io.git

or 

    ~# git clone git@github.com:erkaiyublog/erkaiyublog.github.io.git

To specify the branch to be cloned, and name the local folder differently,

    ~# git clone <http link/ssh link> -b <branch name> <local folder name>

## Notation (IMPORTANT)

To make things clear, in the later sections, I will just assume that you are working on a Git repository called "**My_Project**" locating at "*~/My_Project*" on your system. I will add a little notation behind the path to specify the Git branch you're working on.

For example, by default, you will be working on a Git branch called **main** (sometimes it's called **master**), so I will write the following prompt notation to represent that

    ~/My_Project(main)# 

That means I ASSUME that your terminal looks like this. So when you are typing commands like ***ls***, it appears as

    ~/My_Project(main)# ls

Your real prompt probably looks different with my notation, but that doesn't matter. Just keep in mind that my notation is to help you understand where should you use the commands I provided.

## Git Config

Git will force you to config before making the first commit.

    ~/My_Project(main)# git config user.email "<your email>"
    ~/My_Project(main)# git config user.name "<your name>"

Note that the email and name provided here are not required to be matched with the ones you used to login **github** or **gitlab**, these are just used as records instead of login authorization. 

Alternatively, you can add a ***--global*** flag to make the Git configuration "global" on your system, so that all the other Git repositories you created on your system will by default use this pair of email and name.

    ~/My_Project(main)# git config --global user.email "<your email>"
    ~/My_Project(main)# git config --global user.name "<your name>"

## Basic Operations on the Main Branch 
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

## Operations with Multiple Branches


