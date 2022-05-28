---
title: Fundamentals of GitHub
date: 2022-05-01 12:00:00 -0000
categories: [Computer Science,General]
tags: [git,github,cli,fundamental,basic,version-control,code,programming,open-source]
---

# Common CLI Commands

1. `git clone [repository link]` &rarr; Clone the remote repository from the link to a local repository.
2. `git status` &rarr; Look at changes, committed and uncommitted.
3. `git pull origin master` or `git pull` &rarr; Pull remote master branch to local branch.
4. `git add [file]` &rarr; Add changed files to be committed.
5. `git add .` &rarr; Add all changes to be committed (can cause accidental conflicts, so better to add individual files that have been changed).
6. `git commit -m "[description]"` &rarr; Push commits to the local repository with description.
7. `git push origin master` or `git push` &rarr; Push commits to the remote master branch.

# GitHub collaboration (Single branch)

Add collaborators to a repository. Collaborators can update a file at the same time, but will result in a conflict instead. These merge conflicts need to be resolved. To avoid problems, it's best to claim a file to work and inform collaborators as well as refrain from working on unclaimed files. Only add modified files to the local commit queue.

Keeping many small files instead of a few big ones is better for management. Pull and push workflows must be done often and collaborators must be informed of them. Text editors have their own way of resolving merge conflicts. Merge conflicts have the following syntax within the code &rarr;

```
<<<<<<< HEAD

Local push code

=======

Existing push code

>>>>>>> 445486de81907127c9f1d611ee10d90480f965e6
```

The `=`’s are a separator. The last string is the commit ID of the most recent change.

> To resolve a conflict, all separators except the required code must be deleted and then pushed in the normal way.
{: .prompt-tip }

# GitHub branches

To create separate branches, use &rarr; `git branch <branch_name> && git checkout <branch_name>` or `git checkout -b <branch_name>` to make a branch and switch to it.

Push the branch to commit it in the remote repository. Pulling master in the branch will update the branch to the code in master. To list the branches in the git repository, use command &rarr; `git branch` And for listing all branches (remote too), use command &rarr; `git branch -a`. To delete a branch, use command &rarr; `git branch -d <branch_name>`

To switch to another branch from current branch with uncommitted changes, push the changes on a stack and pop with it to get the changes back (to be done in the correct branch only) &rarr; `git stash [pop]`

# Pull requests

Pull requests are requests to add the branch code to the master branch (merging). Can be created from the web interface of GitHub after selecting the required branch. To add a remote repository as the upstream (original remote), use the following &rarr; `git remote add upstream <https://github.com/><user>/<repo>.git`

To sync the current repository with remote upstream, use &rarr; `git fetch upstream`

# GitHub CI Actions

CI/CD can be performed on a repository in GitHub using GitHub Actions. There are different requirements for running these workloads, but the free tier usually suffices for general software/code.

A workflow can be defined by adding a yaml file to the `.github/workflows` directory in the root location for the repository. An example of such a workflow from the [dockers](https://github.com/tanq16/dockers) repository is as follows &rarr;

```yaml
name: Security Image

on:
  push:
    branches:
      - 'main'
  schedule:
    - cron: '0 0 15 * *'
  workflow_dispatch:
    inputs:
      tags:
        description: 'run'
        required: false 
        type: boolean

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      
      - name: Docker Login
        uses: docker/login-action@v1.10.0
        with:
          username: tanq16
          password: ${{ secrets.DOCKER_ACCESS_TOKEN }}
        
      - name: Build and push security docker
        uses: docker/build-push-action@v2
        with:
          context: security_docker
          push: true
          tags: tanq16/sec_docker:main
```

1. The `on` defines the events which trigger the workflow. In this case, evey commit on the main branch and the cron job of every 1st, 14th and 28th day of the month triggers the workflow.
2. The `jobs` define the actual commands/actions that are executed in the workflow.
3. The `uses` defines a public action that is inherited to run in the current workflow. Alternatively, the `run` can be used to define exact commands that need to be run in the CI environment.
4. Multiple actions/commands can be defined within the `steps` to denote the serial execution of them in the given order.
