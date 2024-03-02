---
published: true
title: A Brief Review of Git Operations 
tags: git
---

---
_As a popular SVN, Git appears in almost every developper's toolkit, regardless of what type of projects they are working on. Git provides us with a convenient way to keep track of all the changes made to a project, so that we can easily revert the project back to certain states if errors occur._

_In this post, I'd like to take down some notes of some common git operations for furture reference. This post is meant to serve as a quick lookup guide for git users instead of a detailed introduction for beginners._

---

## Create a Local Repository
Suppose you have a folder called **My_Project** on your local operating system, and you want to turn it into a git repository. Regardless of whether there're any files inside **My_Project**, all you need to run is 

    ~/My_Project# git init

This will create a folder called **.git** under **My_Project** which makes it a git repository. 

Note that at this point, the git repository doesn't have any file recorded even if there are files under **My_Project**. To make Git record the existing files in **My_Project**, you need to manually use ***git add*** operations as introduced in later sections.

## Clone a Remote Repository
Compared with creating a local repository, a more common scenario of Git usage is to fetch a repository online, this process is done with ***git clone***. 

Usually, we clone git repositories from websites like **github**, **gitlab**,it's important to be aware of the difference in between websites like **github** and the tool Git. **github** is a website that stores many git repositories uploaded by developpers, so that they can contribute and collaborate easily. 

To clone a git repository, we need either its http link or its SSH link. For example, the source code of this blog website is placed on **github**, to clone it, you can use 

    ~# git clone https://github.com/erkaiyublog/erkaiyublog.github.io.git

or 

    ~# git clone git@github.com:erkaiyublog/erkaiyublog.github.io.git

Usually, cloning with HTTP link is more convenient, since cloning with SSH requires you to configure your local machine's SSH setting. 

Two common options are worth noticing when using ***git clone***. By default, Git will clone the *master* or *main* branch of the remote repository, you can specify the branch you want to clone with a ***-b*** flag when cloning. Adding a folder name after the cloning link will rename the cloned rerpository with the provided name. For example, when using HTTP link to clone a git repository's *dev* branch and name it as *local_repo*,

    ~# git clone https://someurl.git -b dev local_repo

~
