Git and VS Code Workflow

This is a basic workflow we can use when working with GitHub through VS Code. The sequence is pretty straightforward:

```text
Install Git
    ↓
Set up your Git identity
    ↓
Create/switch to your branch
    ↓
Make your changes
    ↓
Stage your changes
    ↓
Commit
    ↓
Push to GitHub
    ↓
Create a Pull Request in VS Code
    ↓
Review
    ↓
Merge
```

## Table of Contents

* [1. Install Git First](#1-install-git-first)
* [2. Tell Git Your Identity](#2-tell-git-your-identity)
* [3. Create a Branch for Your Work](#3-create-a-branch-for-your-work)
* [4. Make Your Changes](#4-make-your-changes)
* [5. Stage Your Changes](#5-stage-your-changes)
* [6. Commit Your Changes](#6-commit-your-changes)
* [7. Push the Branch to GitHub](#7-push-the-branch-to-github)
* [8. Create a Pull Request Directly from VS Code](#8-create-a-pull-request-directly-from-vs-code)
* [9. Video Demo](#9-video-demo)
* [Don't Mix These Three Up](#dont-mix-these-three-up)
* [Quick Reference](#quick-reference)
* [The Main Idea](#the-main-idea)

## 1. Install Git First

Before you can use Git commands in the VS Code terminal, you need to have Git installed on your computer.

You can download Git here: [Download Git](https://git-scm.com/downloads?utm_source=chatgpt.com)

After installing it, open VS Code and open a new terminal:

<img width="780" height="82" alt="image" src="https://github.com/user-attachments/assets/9e23c4f8-f379-4924-8f5d-f2f034b2d7e7" />

```text
VS Code
→ Terminal
→ New Terminal
```

Then check if Git is working:

```bash
git --version
```

If everything is okay, you should see something like:

```text
git version 2.55.0.windows.5
```

If there's a version number, you're good to go.

## 2. Tell Git Your Identity

Before you start making commits, Git needs to know your name and email.

Run:

```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

You only need to do this once on your computer.

Check your configuration with:

```bash
git config --global --list
```

## 3. Create a Branch for Your Work

Don't make your changes directly on `main` if you're working as part of a team. Create your own branch first.

To see which branch you're currently on:

```bash
git branch --show-current
```

Create a branch:

```bash
git branch <branch-name>
```

Then switch to it:

```bash
git switch <branch-name>
```

For example:

```bash
git branch test-vscode-pr
git switch test-vscode-pr
```

The branch = your own workspace for a particular change:

```text
main
 └── test-vscode-pr
        └── your changes
```

## 4. Make Your Changes

Now you can work on your files normally in VS Code.

For example, you might:

* Add a new notebook
* Modify an existing query
* Update documentation

Once you're done with your changes, tell Git which changes you want to save.

## 5. Stage Your Changes

Run:

```bash
git add .
```

The `.` means "stage all changes in this directory."

Staging = preparing your changes for a commit:

```text
Your files
   ↓
git add .
   ↓
Staged changes
```

## 6. Commit Your Changes

Once your changes are staged, create a commit:

```bash
git commit -m "Add validation checks"
```

Commit = your checkpoint = set of changes in your local Git history.

So now:

```text
Your files
   ↓
git add .
   ↓
Staging area
   ↓
git commit
   ↓
Local Git history
```

Make your commit message describe what you actually changed.

## 7. Push the Branch to GitHub

At this point, your commit exists on your computer. It hasn't been sent to GitHub yet.

Push your branch:

```bash
git push -u origin <branch-name>
```

For example:

```bash
git push -u origin test-vscode-pr
```

Now the branch and its commits are available on GitHub.

## 8. Create a Pull Request Directly from VS Code

Instead of opening GitHub in a browser, we can create and manage PRs directly in VS Code using the **GitHub Pull Requests and Issues** extension.

<img width="931" height="526" alt="image" src="https://github.com/user-attachments/assets/c352a0a6-e04f-4c38-961e-53cab8417327" />

### Install the extension

In VS Code:

```text
Extensions
→ Search: GitHub Pull Requests and Issues
→ Install
```

Sign in to your GitHub account when prompted.

### Create the PR

First, make sure you have already:

```text
Created your branch
       ↓
Made your changes
       ↓
Committed your changes
       ↓
Pushed your branch to GitHub
```

Then in VS Code:

```text
Source Control / GitHub Pull Requests and Issues
        ↓
Pull Requests
        ↓
Create Pull Request
```

<img width="435" height="597" alt="image" src="https://github.com/user-attachments/assets/ef0f1c95-2d12-4003-b0f7-bc70f42cad67" />

Select branches when prompted:

```text
base: main
compare: test-vscode-pr
```

Then fill in the PR details:

```text
Title:
Add validation checks

Description:
- Added validation checks
- Updated the validation notebook
- Verified the results
```

Review the changes and create the Pull Request.

After creating it, you can view the PR directly inside VS Code. The extension also lets us review changes, add reviewers, leave comments, and monitor PRs.

## 9. Video Demo

Here's a video demonstration of the workflow:

**[▶️ Watch the Git and VS Code Workflow Demo](https://drive.google.com/drive/folders/127WPMWegDWV_J4lb4T1PwZlxaBBbQKVY?usp=sharing)**

The demo walks through the basic process:

```text
Create branch
     ↓
Make changes
     ↓
git add .
     ↓
git commit
     ↓
git push
     ↓
Create Pull Request
     ↓
Review
     ↓
Merge
```

> Replace `PASTE-YOUR-VIDEO-LINK-HERE` with the actual video link before publishing this TIL.

## Don't Mix These Three Up

```text
git commit
    ↓
Saves your changes to your local Git history

git push
    ↓
Sends your commits and branch to GitHub

Pull Request
    ↓
Asks the team to review your changes
and potentially merge them into main
```

Just because you committed something doesn't mean it's already on GitHub.

And just because you pushed something doesn't mean it has been merged into `main`.

The whole process looks like this:

```text
Install Git
     ↓
Configure Git
     ↓
Create branch
     ↓
Make changes
     ↓
git add .
     ↓
git commit
     ↓
git push
     ↓
Create Pull Request
     │
     └── GitHub Pull Requests and Issues extension
              ↓
            Review
              ↓
            Merge
```

## Quick Reference

| What you want to do              | Command / Action                                                                |
| -------------------------------- | ------------------------------------------------------------------------------- |
| Check if Git is installed        | `git --version`                                                                 |
| Check the status of your changes | `git status`                                                                    |
| Set your name                    | `git config --global user.name "Your Name"`                                     |
| Set your email                   | `git config --global user.email "your-email@example.com"`                       |
| Check current branch             | `git branch --show-current`                                                     |
| List local and remote branches   | `git branch -a`                                                                 |
| Create a branch                  | `git branch <branch-name>`                                                      |
| Switch branches                  | `git switch <branch-name>`                                                      |
| Stage changes                    | `git add .`                                                                     |
| Restore/discard unstaged changes | `git restore <file-name>`                                                       |
| Commit changes                   | `git commit -m "your message"`                                                  |
| View commit history              | `git log`                                                                       |
| View commit history as a graph   | `git log --oneline --graph --all`                                               |
| Push to GitHub                   | `git push -u origin <branch-name>`                                              |
| Delete a local branch            | `git branch -d <branch-name>`                                                   |
| Create a PR                      | VS Code → GitHub Pull Requests and Issues → Pull Requests → Create Pull Request |

## The Main Idea

You can do almost the entire workflow without leaving VS Code:

```text
VS Code
  │
  ├── Write/edit your code
  ├── Stage changes
  ├── Commit
  ├── Push
  └── Create/manage Pull Request
          │
          ↓
       GitHub
```

### Takeaway

Git handles the version control and the **GitHub Pull Requests and Issues** extension gives us a convenient way to work with GitHub directly from VS Code.
