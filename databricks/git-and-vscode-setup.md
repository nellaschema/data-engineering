# TIL: Git & VS Code Workflow for Batchmates

If you're new to Git, this is the basic workflow I use when working with GitHub through VS Code. The first time you see all these commands, they can look like a lot, but they follow a pretty straightforward sequence.

Think of it as:

```text
Install Git
    ↓
Set up your Git identity
    ↓
Create / switch to your branch
    ↓
Make your changes
    ↓
Stage your changes
    ↓
Commit
    ↓
Push to GitHub
    ↓
Create a Pull Request
```

## 1. Install Git First

Before you can use Git commands in the VS Code terminal, you need to have Git installed on your computer.

You can download Git here:

[Download Git](https://git-scm.com/downloads?utm_source=chatgpt.com)

After installing it, open VS Code and open a new terminal:

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
git version 2.x.x
```

If you get a version number, you're good to go.

## 2. Tell Git Who You Are

Before you start making commits, Git needs to know your name and email.

Run:

```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

You only need to do this once on your computer.

You can check your configuration with:

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

Now you can work on your files normally in VSCode.

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

## 7. Push Your Branch to GitHub

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

## 8. Create a Pull Request

Once you've pushed your branch, you can create a Pull Request (PR).

PR = you telling the team:

> "I've made these changes. Please review them and, if everything is okay, merge them into `main`."

You can create the PR directly through GitHub. After pushing, Git may also give you a link in the terminal that you can click to open the PR page.

But I wondered if I can create a PR directly from VSCode so I came across the **GitHub Pull Requests and Issues** extension in VS Code to create and manage PRs without leaving VS Code.

## Don't Mix These Three Up

```text
git commit
    ↓
Saves your changes to your local Git history

git push
    ↓
Sends your commits to GitHub

Pull Request
    ↓
Asks the team to review and potentially merge your changes to main
```

So just because you committed something doesn't mean it's already on GitHub.

And just because you pushed something doesn't mean it has been merged into `main`.

## Complete Example

Let's say you're adding validation checks to your project.

First, create and switch to your branch:

```bash
git branch feature/add-validation
git switch feature/add-validation
```

Then make your changes in VS Code.

When you're done:

```bash
git add .
git commit -m "Add validation checks"
git push -u origin feature/add-validation
```

Then go to GitHub and create your Pull Request.

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
Pull Request
     ↓
Review
     ↓
Merge
```

## Quick Reference

| What you want to do       | Command                                                   |
| ------------------------- | --------------------------------------------------------- |
| Check if Git is installed | `git --version`                                           |
| Set your name             | `git config --global user.name "Your Name"`               |
| Set your email            | `git config --global user.email "your-email@example.com"` |
| Check current branch      | `git branch --show-current`                               |
| Create a branch           | `git branch <branch-name>`                                |
| Switch branches           | `git switch <branch-name>`                                |
| Stage changes             | `git add .`                                               |
| Commit changes            | `git commit -m "your message"`                            |
| Push to GitHub            | `git push -u origin <branch-name>`                        |
| Create a PR               | GitHub or VS Code                                         |
