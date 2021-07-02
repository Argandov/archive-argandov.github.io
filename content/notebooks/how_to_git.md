---
title: "How to git"
date: 2021-06-30T21:06:05-06:00
draft: false
tags: ["tools"]
Author: "Argandov"
lead: "Quick reference for git projects"
toc: true 

---

Step by step for new repositories, working locally and remotely

<!--more-->

## New local repository

`git config --global user.name "my name"`
`git config --global user.email "my email"`
*(For reference when working with teams; not necessarily related to the account)*

1.- `git init`

\*Adds a new .git directory on the current folder (pwd). To stop git from following this repo, delete .git directory

2.- `touch .gitignore`

Add here any file to be "ignored" in upstream (Private info or irrelevant to the project)
\* **Note: If the files assigned in .gitignore are already in the repository, it's necessary to remove them:**
`git rm -rf --cached .`

I.e. `echo "id_rsa" >> .gitignore`
or `echo "local_tests" >> .gitignore`

2.1.- `git status`

Checking the staging (or unstaged) files, before commiting.
Red: unstaged - Will be ignored if we commit at this stage
Green: Staged - Ready to commit

3.- `git add -A`

(Or `git add some_file.md`)

To be executed AFTER any changes

Will select our desired files (Here's where .gitignore takes place) to commit.

4.- `git status` (again)

Double check for staging/unstaged files

5.- `git reset`

Undo changes to staging

6.- `git commit -m "My commit message"`

Final changes made, before remote pushing

7.- `git log`

See recent changes, hashes, messages, etc.

---

## Working remotely

`git clone https://\*/\*.git`

"Download" an existing or new, remote repository

`git remote -v`

Show urls and branches

`git branch -a`

Show current branches on the project

`git diff`

Uses "less" in Linux. Shows detailed differences (inside files) between last and current states.

`git add -A && git commit -m "my commit message"`

`git pull origin main`

For work in teams. Shows the difference between last and current pull.
(In personal repositories we should see no changes here)

`git push origin main`

(Name the correct branch: main/master/etc.)
Ready to go upstream.

Depending on the setup, it will ask for username/password, or if ssh keys are already configured, it will push automatically.

---

## Branching

`git branch my_new_branch`

Create a new branch (But we're not working on it yet)

`git branch`

Shows the current working branch

`git checkout my_new_branch`

Think "checkout" like "switch".
We "hop" to the new branch, and any changes from here after, will be made to this new branch.

`git push -u origin my_new_branch`

Our main branch is still unchanged.
At this point, we have 2 remote branches.

`git branch -a`

Show all branches

`git branch --merged`

Already merged branches

## Merge branches

`git branch --merged`

Check merged branches

`git merge my_new_branch`

`git push`

**To delete old branch:**

`git branch --merged`
`git branch -d my_new_branch`
`git branch -a`
`git push origin --delete my_new_branch`
