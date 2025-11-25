---
tags:
  - git
published: true
---
# Git Allgemein Infos

Begriffe

- Git directory = Lokaler clone des Repos
- Git working tree = pull vom Git directory
- Git Stag = Lokalle änderungen

**Anpassen einer Fehlerhaften commit Massage vor dem  push zum  remote System.**

Hier bietet sich `--amend` an
Syntax: `git commit --amend`
Beispiel: `git commit --amend -m "New commit message"`

**Git remote set-url wird benutzt um die remote url zu ändern**

`git remote set-url origin https://github.com/USERNAME/REPOSITORY_xyz.git`

Dabei ist https://github.com/USERNAME/REPOSITORY_xyz.git das neue Remote repo

Zum Überprüfen kann `git remote -v` benutzt werden.

**Kopieren von einer oder zwei Datein von einem branch zum andern. Hier von _dev_.**

`git checkout dev -- path/to/your/file.`

**Kopiren von Datein und Verzeichnissen von einem commit hash zu einem anderen branch.**

`git checkout <commit_hash> <relative_path_to_file_or_dir>`

Zum Überprüfen kann `git remote -v` benutzt werden.

**Kopieren einer Datei oder Verzeichnisses von einem branch zu einem anderen.**

`git checkout dev -- path/to/your/file.`

**Kopieren von einem Verzeichnis im Bransch zu einen anderen Branch.**

`git checkout dev -- path/to/your/folder`

**Kopieren von Datein und Verzeichnissen von einem commit hash zu einem anderen branch.**

`git checkout <commit_hash> <relative_path_to_file_or_dir>`

## Eintragungen bereit gesteller Änderungen Herausnehmen

`git reset HEAD DATEI`

## Git Konfiguration

## Git bransh

**Pullen von einem branch.**

`git pull <remote> <branch>`

**Bransh festlegen.**

`git branch --set-upstream-to=origin/<branch> test`

**Git Author reset**

```sh
git config --global --edit
git commit --amend --reset-author
```

[[git-submodules]]

## Git sed

* [](https://mehl.mx/blog/2020/the-power-of-git-sed/)

# Clear Git Cache

Managing Git repositories efficiently often involves clearing the cache to ensure that changes in tracked files or directory structures are correctly recognized. The Git cache can sometimes retain outdated information, leading to inconsistencies in your repository. Clearing the Git cache is a straightforward process that helps maintain an accurate and up-to-date working directory.

This guide will demonstrate how to clear the Git cache using the command-line terminal. Whether you’re addressing issues with ignored files, updating the repository structure, or ensuring accurate tracking of changes, these steps will help you manage your Git cache effectively.

## Commands to Clear the Entire Git Cache

Clearing the Git cache is vital when .gitignore seems to ignore changes or when you need to refresh the index to accurately reflect the current state of tracked and untracked files. This action forces Git to reevaluate your .gitignore settings, thereby ignoring files that should not be tracked.

### Navigate to Your Repository

Start by opening a terminal. To access your project’s directory, use the cd command followed by the path to your Git repository. This step ensures you’re working within the correct context for the Git commands you’ll execute.

```bash
cd ~/your-git-repository
```

### Remove Cached Files

Next, clear out the Git cache. This step doesn’t affect your local files but removes all files from Git’s index. The command git rm -r –cached recursively removes files from the cache, preparing the stage for a fresh start.

```bash
git rm -r --cached .
```

After executing this command, Git’s index is empty, but your files remain untouched on your local disk.

### Reset Git Index

Resetting the Git index ensures that your next commit accurately reflects the current state of your project, minus the files you’ve intended to ignore.

```bash
git reset .
```

This command refreshes the staging area, effectively syncing it with the last commit while adhering to the .gitignore rules.

### Verify the Changes

It is crucial to check the status of your repository. This command provides a snapshot of the current state, showing which files are untracked, modified, or ready to be committed.

```bash
git status
```

### Re-add Files

To re-add your files to the Git index (this time excluding files specified in .gitignore), use the following command. It respects your .gitignore settings, only adding files that should be tracked.

```bash
git add .
```

### Commit the Cache Clear Changes

To finalize the process, commit the changes. This step records the cache reset in your repository’s history, ensuring that the cache clearing has a point of reference.

```bash
git commit -am 'Reset the entire repository cache.'
```

This command commits all current changes, embedding the reset of the Git cache within your project’s commit history.

## Commands to Clear Git Cache for Specific Files

Follow these steps to selectively remove files or directories from the Git cache without clearing the entire cache. This method is beneficial for correcting tracking errors on a smaller scale.

### Remove a Single File from Git Cache

If you need to untrack a single file mistakenly added to the Git repository, you can remove it from the cache without deleting it from your local filesystem.

```bash
git rm --cached your-file-here.txt
```

This command updates the Git index so that the specified file can no longer be tracked while the file remains in your working directory.

### Clear Git Cache for a Directory

For directories, the process is similar. By using the -r (recursive) option, you can remove an entire directory from the Git cache.

```bash
git rm -r --cached ./your/directory/here
```

This effectively stops tracking the directory and its contents, adhering to any updates in your .gitignore without affecting the local copies of those files or directories.

### Verify and Commit Changes

After removing specific items from the cache, it’s important to verify the changes with git status. This shows the current tracking status and any files no longer being tracked.

```bash
git status
```

Then, commit your changes to ensure that the removal of specific files or directories from the cache is recorded in the repository’s history.

```bash
git commit -am 'Removed specific items from the cache.'
```

## Clearing Git Cached Credentials

Managing cached credentials securely is critical, especially on shared systems where leaving your credentials cached could pose a security risk.

### Commands to Clear Git Credentials

The first step is to navigate to your repository. From there, you can clear your cached credentials using Git’s built-in tools, which vary depending on how you’ve configured Git to handle credentials.

If you’re using Git’s credential cache, you can clear it with:

```bash
git credential-cache exit
```

Alternatively, if your credentials are stored more permanently, you might need to directly edit your .gitconfig or use the git config command to unset the credential helper:

```bash
git config --global --unset credential.helper
```

These commands help ensure that your credentials are not stored longer than necessary, safeguarding your repositories.

## Additional Commands for Git Repository Management

Beyond the core commands for clearing the Git cache and managing credentials, there are additional practices that can enhance your Git experience and ensure your repository remains clean and efficient.

### Check .gitignore Effectiveness

After making changes to your .gitignore file or clearing your cache, it’s wise to verify that .gitignore is functioning as expected. Git provides a tool for this exact purpose:

```bash
git check-ignore -v PATH_TO_FILE
```

This command not only tells you if a file is ignored but also specifies which .gitignore rule is responsible for the behavior. It’s a great way to debug and confirm that your .gitignore rules are applied correctly.

### Use a Global .gitignore for Personal Files

Developers often work with tools that generate local files you don’t want to track in every project (like editor configurations or OS-specific files). Instead of adding these to every project’s .gitignore, you can create a global .gitignore file:

```bash
git config --global core.excludesfile '~/.gitignore_global'
```

### Regularly Prune Your Repository

Git stores references to objects (commits, trees, blobs, etc.) that can become obsolete over time. Pruning these objects helps reduce clutter and can improve performance:

```bash
git gc --prune=now
```

This command cleans up unnecessary files and optimizes your repository’s storage.

### Leverage Git Hooks for Automation

Git hooks allow you to automate specific actions based on Git events, such as pre-commit checks, automatic testing, or linting before allowing a commit:

```bash
cd .git/hooks
```

Explore this directory to see sample hooks Git provides. Renaming a sample (by removing .sample from the file name) and customizing it allows you to automate various tasks, enhancing your workflow.

### Keep Your Branches Organized

As projects grow, so does the number of branches. Regularly cleaning up merged or stale branches helps keep your repository navigable:

```bash
git branch --merged | egrep -v "(^\*|master|main)" | xargs git branch -d
```

This command lists branches merged into your current branch, excluding master or main, and deletes them. It’s a quick way to clean up after feature development or bug fixes.

# Undo Last Git Commit

Undoing the last Git commit is a common task for developers needing to revert changes or correct mistakes. Git provides several methods to undo the most recent commit, allowing you to maintain a clean and accurate project history. These methods include resetting, reverting, or amending commits, each serving different purposes depending on whether you want to preserve or discard the changes.

This guide will demonstrate how to undo the last Git commit using practical examples. By mastering these techniques, you can efficiently manage your Git history and ensure your project remains well-organized and error-free.

## Understanding Git Commit

Before we explore how to undo a Git commit, it’s crucial to comprehend what a commit signifies in Git. A commit represents a snapshot of your work at a specific moment. It encapsulates all the modifications you’ve made since the last commit. Each commit is uniquely identified by a SHA-1 hash, enabling Git to maintain a history of changes.

## Reversing a Git Commit

There are two primary techniques to reverse a Git commit: git reset and git revert. We will discuss each method in detail, including their use cases and implications.

```bash
git reset --soft HEAD~1
```

### Using git reset

The git reset command is a potent tool that allows you to move or “reset” your current HEAD to a specified state. Here’s how to use it to undo your last commit:

```bash
git reset --soft HEAD~1
```

In this command, –soft ensures that your changes are preserved in the staging area, and HEAD~1 refers to the commit before the current one. The changes from the undone commit will remain in your working directory and the staging area.

Here’s an example output:

```bash
Unstaged changes after reset:
M    file1.txt
M    file2.txt
```

This indicates that the changes in file1.txt and file2.txt are now unstaged.

If you wish to completely discard the commit and all associated changes, you can use the –hard option:

```bash
git reset --hard HEAD~1
```

Exercise caution with this command, as it permanently discards all changes from the last commit.

### Utilizing git revert

The git revert command creates a new commit that undoes the changes made in a previous commit. This is a safe way to undo changes, as it doesn’t alter the existing commit history. Here’s how you can use it:

```bash
git revert HEAD
```

This command will create a new commit that undoes the changes made in the last commit.

Here’s an example output:

```bash
[master 0a9b2aa] Revert "Add new feature"
 1 file changed, 1 insertion(+), 2 deletions(-)
```

This indicates that a new commit has been created that reverts the changes from the previous commit.

## Undoing a Git Commit: Expanded Examples

### Using git reset: More Examples

#### Resetting to a Specific Commit

If you want to undo not just the last commit but several commits, you can do so by specifying the commit hash instead of `HEAD~1`. For example:

```bash
git reset --soft 9fceb02
```

This command will move the HEAD to the commit specified by the hash 9fceb02, and the changes from all commits after 9fceb02 will be kept in the staging area.

#### Resetting Unstaged Changes

If you have made changes but haven’t committed them yet and you want to undo these changes, you can use git reset without any commit reference:

```bash
git reset --hard
```

This command will undo all unstaged changes, reverting your working directory to the state of the last commit.

### Using git revert: More Examples

#### Reverting a Range of Commits

If you want to undo a range of commits, you can specify the range using the git revert command. For example:

```bash
git revert HEAD~3..HEAD
```

This command will create new commits that undo the changes made in the last three commits.

#### Reverting a Specific Commit

If you want to undo a specific commit that’s not the most recent one, you can do so by specifying the commit hash:

```bash
git revert 9fceb02
```

This command will create a new commit that undoes the changes made in the commit specified by the hash `9fceb02`.