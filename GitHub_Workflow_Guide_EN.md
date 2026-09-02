# Git & GitHub Workflow -- Complete Guide

> A practical step-by-step guide for collaborating through GitHub on
> Windows and Ubuntu/Linux using SSH, branches, pull requests, and code
> reviews.

## 1. Basic Concept

Git manages the version history **locally** on your computer. GitHub
hosts the **remote repository** that you use to collaborate with other
developers.

A typical workflow looks like this:

``` text
GitHub Repository
      │
      │ git clone / git pull
      ▼
Local Repository
      │
      ├── Edit files
      ├── git add
      ├── git commit
      └── git push
      ▼
GitHub Repository
```

For professional collaboration, the following workflow is usually
recommended:

``` text
main
 │
 └── Feature Branch
        │
        ├── Changes
        ├── Commits
        └── Push
             │
             ▼
        Pull Request
             │
             ▼
        Code Review
             │
             ▼
           Merge
             │
             ▼
            main
```

------------------------------------------------------------------------

# 2. Installing Git

## Windows

Install Git for Windows:

-   https://git-scm.com/download/win

Then verify the installation in PowerShell or Git Bash:

``` powershell
git --version
```

## Ubuntu / Linux

``` bash
sudo apt update
sudo apt install git openssh-client
```

Verify:

``` bash
git --version
ssh -V
```

------------------------------------------------------------------------

# 3. Configuring Your Git User

Git stores a name and email address with every commit.

``` bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

Display the global configuration:

``` bash
git config --global --list
```

Or check the values individually:

``` bash
git config --global user.name
git config --global user.email
```

## Changing the User

Existing values can simply be overwritten:

``` bash
git config --global user.name "New Name"
git config --global user.email "new-email@example.com"
```

Alternatively, remove them first:

``` bash
git config --global --unset user.name
git config --global --unset user.email
```

## Configuration for One Repository Only

Inside the repository directory:

``` bash
git config user.name "Name for This Project"
git config user.email "project-email@example.com"
```

Without `--global`, the setting applies only to the current repository.

## Checking Where a Setting Comes From

``` bash
git config --list --show-origin
```

This is especially useful if the same setting exists at repository,
global, or system level.

## Removing a Specific Credential Entry

Example:

``` text
credential.https://git.somecompany.at.provider=generic
```

Remove it globally:

``` bash
git config --global --unset credential.https://git.somecompany.at.provider
```

If it exists only in the current repository:

``` bash
git config --unset credential.https://git.somecompany.at.provider
```

If it is system-wide:

``` bash
sudo git config --system --unset credential.https://git.somecompany.at.provider
```

------------------------------------------------------------------------

# 4. Setting Up GitHub with SSH

GitHub no longer accepts normal account passwords for Git operations
over HTTPS. SSH is a convenient option for development computers.

## 4.1 Creating an SSH Key on Ubuntu/Linux

Check the standard SSH directory:

``` bash
ls -la ~/.ssh
```

Create it if necessary:

``` bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

Create an Ed25519 key:

``` bash
ssh-keygen -t ed25519 -C "your-email@example.com"
```

When asked:

``` text
Enter file in which to save the key (/home/user/.ssh/id_ed25519):
```

simply press **Enter**.

This creates:

``` text
~/.ssh/id_ed25519       # private key
~/.ssh/id_ed25519.pub   # public key
```

The private key must **never be shared**.

## 4.2 Starting the SSH Agent on Linux

``` bash
eval "$(ssh-agent -s)"
```

Load the key:

``` bash
ssh-add ~/.ssh/id_ed25519
```

Check loaded keys:

``` bash
ssh-add -l
```

Display the public key:

``` bash
cat ~/.ssh/id_ed25519.pub
```

Copy the complete output line.

> Note: If the SSH agent was started only for the current shell session,
> you may need to start it again and/or run `ssh-add` after closing and
> reopening the terminal.

## 4.3 Creating an SSH Key on Windows

In PowerShell:

``` powershell
ssh-keygen -t ed25519 -C "your-email@example.com"
```

Press Enter when asked for the file location. The default location is
normally:

``` text
C:\Users\<Username>\.ssh\id_ed25519
C:\Users\<Username>\.ssh\id_ed25519.pub
```

Display the public key:

``` powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub
```

Check the SSH agent:

``` powershell
Get-Service ssh-agent
```

Start it if necessary:

``` powershell
Start-Service ssh-agent
```

Add the key:

``` powershell
ssh-add $env:USERPROFILE\.ssh\id_ed25519
```

## 4.4 Adding the Public Key to GitHub

On GitHub:

1.  Open **Settings**
2.  Select **SSH and GPG keys**
3.  Click **New SSH key**
4.  Enter a descriptive name, for example `Ubuntu Development PC`
5.  Paste the contents of `id_ed25519.pub`
6.  Save the key

## 4.5 Testing the SSH Connection

``` bash
ssh -T git@github.com
```

A successful authentication produces a message similar to:

``` text
Hi USERNAME! You've successfully authenticated, but GitHub does not provide shell access.
```

During the first connection, SSH may ask whether you trust the host.
After verifying GitHub's host key, you can accept it.

------------------------------------------------------------------------

# 5. Error: Permission Denied (publickey)

If you see:

``` text
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.
```

first check:

``` bash
ssh-add -l
```

If no key is loaded:

``` bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

Then test again:

``` bash
ssh -T git@github.com
```

For detailed troubleshooting:

``` bash
ssh -vT git@github.com
```

Also check the repository remote:

``` bash
git remote -v
```

For GitHub over SSH, the address should look similar to:

``` text
git@github.com:Organization/Repository.git
```

Avoid using:

``` bash
sudo git push
```

Using `sudo` can cause Git/SSH to run as another user and therefore use
a different SSH configuration and different keys.

------------------------------------------------------------------------

# 6. Cloning a Repository for the First Time

On GitHub:

**Repository → Code → SSH**

The SSH address looks similar to:

``` text
git@github.com:Organization/Project.git
```

## Ubuntu/Linux

``` bash
cd ~/Projects
git clone git@github.com:Organization/Project.git
cd Project
```

## Windows

``` powershell
cd D:\Projects
git clone git@github.com:Organization/Project.git
cd Project
```

Then check:

``` bash
git status
git remote -v
```

------------------------------------------------------------------------

# 7. Normal Workflow on `main`

Before starting work:

``` bash
git switch main
git pull
```

Then edit your files.

Check the current status:

``` bash
git status
```

Stage all changes:

``` bash
git add .
```

Or stage specific files:

``` bash
git add src/main.py
```

Check again:

``` bash
git status
```

Create a commit:

``` bash
git commit -m "Fix communication handling"
```

Push it:

``` bash
git push
```

The complete workflow is:

``` bash
git switch main
git pull

# Edit files

git status
git add .
git status
git commit -m "Description of the changes"
git push
```

------------------------------------------------------------------------

# 8. `git commit -m` vs. `git commit -a`

## `git commit -m`

`-m` stands for **message**:

``` bash
git commit -m "Fix heartbeat timeout"
```

Only changes that have already been staged are committed.

Typical workflow:

``` bash
git add .
git commit -m "Fix heartbeat timeout"
```

## `git commit -a`

`-a` means Git automatically stages modifications and deletions of
**already tracked files**:

``` bash
git commit -a
```

New, untracked files are **not** added.

## Combining Both

``` bash
git commit -am "Fix heartbeat handling"
```

This is convenient when you only changed files that Git already tracks.

For customer or team projects, the explicit workflow is usually easier
to review:

``` bash
git status
git add .
git status
git commit -m "Description"
```

This gives you another chance to verify exactly what will be included in
the commit.

------------------------------------------------------------------------

# 9. Understanding Branches

A branch represents an independent line of development within the same
repository.

Important: **Switching branches does not change the path of the
repository on your computer.**

Example:

``` text
/home/user/Projects/Project/
```

After:

``` bash
git switch feature/new-api
```

you are still inside the same directory. Git only updates the files in
the working tree to match the selected branch.

Check the current path:

``` bash
pwd
```

Display the current branch:

``` bash
git branch
```

Example:

``` text
  main
* feature/new-api
```

The `*` indicates the active branch.

------------------------------------------------------------------------

# 10. Creating a New Branch That Does Not Yet Exist on GitHub

First update `main`:

``` bash
git switch main
git pull
```

Create a new branch and switch to it:

``` bash
git switch -c feature/new-feature
```

Check:

``` bash
git branch
```

Make your changes and commit them:

``` bash
git status
git add .
git commit -m "Implement new feature"
```

Because the branch does not yet exist on GitHub, the **first push**
should be:

``` bash
git push -u origin feature/new-feature
```

`-u`, or `--set-upstream`, links:

``` text
local:  feature/new-feature
          ↕
remote: origin/feature/new-feature
```

After that, you can simply use:

``` bash
git push
```

and:

``` bash
git pull
```

------------------------------------------------------------------------

# 11. Working with a Branch That Already Exists on GitHub

First update your remote information:

``` bash
git fetch
```

Display all branches:

``` bash
git branch -a
```

Example:

``` text
* main
  remotes/origin/main
  remotes/origin/feature/new-api
```

Switch to the existing branch:

``` bash
git switch feature/new-api
```

With a current Git version, Git will normally create a local tracking
branch for `origin/feature/new-api` automatically.

Then:

``` bash
git status
git pull
```

You can now work normally:

``` bash
git add .
git commit -m "Update API handling"
git push
```

If automatic tracking does not work:

``` bash
git switch --track origin/feature/new-api
```

------------------------------------------------------------------------

# 12. Switching Between Branches

Switch to `main`:

``` bash
git switch main
```

Switch to another branch:

``` bash
git switch feature/new-feature
```

Display local branches:

``` bash
git branch
```

Display local and remote branches:

``` bash
git branch -a
```

Display remote branches only:

``` bash
git branch -r
```

Update remote branch information:

``` bash
git fetch
```

------------------------------------------------------------------------

# 13. Before Switching Branches

Always check:

``` bash
git status
```

Ideally:

``` text
nothing to commit, working tree clean
```

Then switching branches is straightforward.

If you still have changes, you can commit them:

``` bash
git add .
git commit -m "Save current work"
```

Or temporarily store them using Git stash:

``` bash
git stash
git switch another-branch
```

Restore them later:

``` bash
git stash pop
```

------------------------------------------------------------------------

# 14. Already Made Changes on `main`? Move Them to a New Branch

If you have modified files on `main` but have **not committed them
yet**, you can normally create a new branch:

``` bash
git switch -c feature/my-change
```

Your local modifications remain in the working tree.

Then:

``` bash
git add .
git commit -m "Implement feature"
git push -u origin feature/my-change
```

This keeps the changes on the new feature branch instead of committing
them to `main`.

------------------------------------------------------------------------

# 15. Pull Request with Required Review

Assume you have already executed:

``` bash
git add .
git commit -m "Implement feature"
```

## Step 1: Push the Branch to GitHub

For the first push:

``` bash
git push -u origin feature/my-change
```

## Step 2: Create the Pull Request

On GitHub:

**Pull requests → New pull request**

Select:

``` text
base:    main
compare: feature/my-change
```

This means:

``` text
feature/my-change
        │
        ▼
       main
```

Create the pull request with a meaningful title and description.

## Step 3: Select a Reviewer

In the pull request, use the **Reviewers** section to select the
colleague who should review your changes.

The workflow then becomes:

``` text
Feature Branch
      │
      ▼
Pull Request
      │
      ▼
Code Review
      │
      ├── Changes requested
      │
      └── Approved
              │
              ▼
             Merge
              │
              ▼
             main
```

Repository rules or branch protection can be configured so that merging
is not possible until the required review has been approved.

------------------------------------------------------------------------

# 16. The Reviewer Requests Changes

You do **not** need to create a new pull request.

Continue working on the same branch:

``` bash
git switch feature/my-change
```

Make the requested changes:

``` bash
git status
git add .
git commit -m "Address review comments"
git push
```

The existing pull request is automatically updated with the new commits.

A pull request represents the difference between branches, rather than
being tied to only one individual commit.

------------------------------------------------------------------------

# 17. Merging a Pull Request

After a successful review, the pull request can be merged depending on
the repository permissions and rules.

Common GitHub options are:

### Merge commit

Preserves the branch and commit structure and creates a merge commit.

### Squash and merge

Combines multiple commits from the feature branch into one commit on
`main`.

For example:

``` text
Add heartbeat
Fix timeout
Fix typo
Address review comments
Improve logging
```

can become:

``` text
Implement frontend heartbeat handling
```

### Rebase and merge

Adds the individual commits to the target branch without creating an
additional merge commit, resulting in a more linear history.

Use the merge strategy defined by your project or team.

------------------------------------------------------------------------

# 18. Cleaning Up Locally After a Merge

After the pull request has been merged:

``` bash
git switch main
git pull
```

Your local `main` now contains the merged changes.

Delete the local feature branch:

``` bash
git branch -d feature/my-change
```

If the remote branch still exists:

``` bash
git push origin --delete feature/my-change
```

Remove stale remote references:

``` bash
git fetch --prune
```

------------------------------------------------------------------------

# 19. Recommended Collaboration Workflow

For changes that should be reviewed:

``` bash
# 1. Get the latest main
git switch main
git pull

# 2. Create a feature branch
git switch -c feature/my-change

# 3. Work on the code

# 4. Review and commit your changes
git status
git add .
git status
git commit -m "Implement feature"

# 5. Push the branch for the first time
git push -u origin feature/my-change
```

Then on GitHub:

``` text
Create Pull Request
        ↓
Select Reviewer
        ↓
Review
        ↓
Additional changes/commits if necessary
        ↓
Approval
        ↓
Merge into main
```

Then locally:

``` bash
git switch main
git pull
git branch -d feature/my-change
git fetch --prune
```

------------------------------------------------------------------------

# 20. Direct Workflow on `main`

Only use this if the repository allows direct pushes to `main` and this
is the agreed team workflow:

``` bash
git switch main
git pull

# Make changes

git status
git add .
git status
git commit -m "Description"
git push
```

For customer and team projects, feature branches and pull requests are
usually safer.

------------------------------------------------------------------------

# 21. Practical Cheat Sheet

  Task                        Command
  --------------------------- ------------------------------------------
  Clone repository            `git clone git@github.com:user/repo.git`
  Check status                `git status`
  Show remote                 `git remote -v`
  Show local branches         `git branch`
  Show all branches           `git branch -a`
  Show remote branches        `git branch -r`
  Update remote information   `git fetch`
  Update `main`               `git switch main` + `git pull`
  Create new branch           `git switch -c feature/name`
  Switch branch               `git switch branch-name`
  Track remote branch         `git switch --track origin/branch-name`
  Stage all changes           `git add .`
  Stage one file              `git add file.py`
  Create commit               `git commit -m "Message"`
  First push of new branch    `git push -u origin branch-name`
  Normal push                 `git push`
  Pull changes                `git pull`
  Temporarily store changes   `git stash`
  Restore stash               `git stash pop`
  Delete local branch         `git branch -d branch-name`
  Delete remote branch        `git push origin --delete branch-name`
  Remove stale remote refs    `git fetch --prune`
  Inspect Git configuration   `git config --list --show-origin`
  Test GitHub SSH             `ssh -T git@github.com`
  Debug GitHub SSH            `ssh -vT git@github.com`

------------------------------------------------------------------------

# 22. Recommended Routine Before Every Push

Especially in a shared repository, first check:

``` bash
git status
```

Verify that you are on the correct branch:

``` bash
git branch
```

Optionally inspect unstaged changes:

``` bash
git diff
```

Inspect staged changes:

``` bash
git diff --staged
```

Then commit and push:

``` bash
git commit -m "Meaningful commit message"
git push
```

------------------------------------------------------------------------

# 23. The Most Important Rules

1.  **Update `main` before starting new work.**

    ``` bash
    git switch main
    git pull
    ```

2.  **Use a separate branch for larger or reviewable changes.**

    ``` bash
    git switch -c feature/name
    ```

3.  **Check `git status` before using `git add .`.**

4.  **Check the staged changes before committing.**

5.  **Use meaningful commit messages.**

6.  **Use `-u` for the first push of a new branch.**

    ``` bash
    git push -u origin feature/name
    ```

7.  **Push review fixes to the same branch.** The pull request updates
    automatically.

8.  **Update your local `main` after a merge.**

    ``` bash
    git switch main
    git pull
    ```

9.  **Check `git status` before switching branches.**

10. **Never share or commit private SSH keys.**

------------------------------------------------------------------------

# 24. Example: Complete Feature Lifecycle

``` bash
# Repository has already been cloned

git switch main
git pull

git switch -c feature/heartbeat

# Edit code

git status
git add .
git status
git commit -m "Implement heartbeat handling"

git push -u origin feature/heartbeat
```

On GitHub:

``` text
feature/heartbeat
       │
       ▼
Pull Request → main
       │
       ▼
Colleague Reviews
       │
       ├── Changes requested
       │       ↓
       │   Make changes locally
       │   git add .
       │   git commit -m "Address review comments"
       │   git push
       │
       ▼
Approved
       │
       ▼
Merge
```

Then locally:

``` bash
git switch main
git pull
git branch -d feature/heartbeat
git fetch --prune
```

The complete development cycle is now finished.
