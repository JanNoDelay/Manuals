# Installing Python and Setting Up a Virtual Environment

This guide explains step by step how to install Python on **Linux (Ubuntu/Debian/Raspberry Pi OS)** and **Windows**, and how to create and use a Python Virtual Environment (`venv`) inside a project directory.

---

# Table of Contents

1. [Why use a Virtual Environment?](#1-why-use-a-virtual-environment)
2. [Linux: Install Python](#2-linux-install-python)
3. [Linux: Create a Virtual Environment](#3-linux-create-a-virtual-environment)
4. [Windows: Install Python](#4-windows-install-python)
5. [Windows: Create a Virtual Environment](#5-windows-create-a-virtual-environment)
6. [Install Packages](#6-install-packages)
7. [Set Up an Existing Project](#7-set-up-an-existing-project)
8. [Set Up a GitHub Project](#8-set-up-a-github-project)
9. [Deactivate and Reactivate the Environment](#9-deactivate-and-reactivate-the-environment)
10. [`.gitignore`](#10-gitignore)
11. [Common Problems](#11-common-problems)
12. [Quick Reference](#12-quick-reference)

---

# 1. Why use a Virtual Environment?

A Virtual Environment is an isolated Python environment for a single project.

This allows different projects to use different versions of Python packages without interfering with each other.

Example:

```text
Project A
└── requests 2.31

Project B
└── requests 2.32
```

The environment is usually created directly inside the project directory:

```text
my-project/
├── .venv/
├── src/
├── requirements.txt
├── pyproject.toml
└── README.md
```

Recommended name:

```text
.venv
```

---

# 2. Linux: Install Python

These steps apply especially to:

- Ubuntu
- Debian
- Raspberry Pi OS
- Linux Mint
- other Debian-based systems

## 2.1 Check whether Python is already installed

Open a terminal and run:

```bash
python3 --version
```

Example:

```text
Python 3.12.3
```

You can also check where Python is installed:

```bash
which python3
```

Typical output:

```text
/usr/bin/python3
```

---

## 2.2 Update package lists

```bash
sudo apt update
```

Optionally update already installed packages:

```bash
sudo apt upgrade
```

---

## 2.3 Install Python, pip, and venv

```bash
sudo apt install python3 python3-pip python3-venv
```

Verify the installation:

```bash
python3 --version
```

and:

```bash
pip3 --version
```

---

# 3. Linux: Create a Virtual Environment

## 3.1 Change into the project directory

Example:

```bash
cd ~/projects/my-project
```

Show the current directory:

```bash
pwd
```

---

## 3.2 Create the Virtual Environment

```bash
python3 -m venv .venv
```

A new directory will appear inside the project:

```text
.venv/
```

---

## 3.3 Activate the Virtual Environment

```bash
source .venv/bin/activate
```

The shell prompt will usually change to something like:

```text
(.venv) user@linux:~/projects/my-project$
```

---

## 3.4 Verify which Python is being used

```bash
which python
```

The result should point to the Virtual Environment, for example:

```text
/home/user/projects/my-project/.venv/bin/python
```

Also check:

```bash
python --version
```

and:

```bash
pip --version
```

---

# 4. Windows: Install Python

## 4.1 Check whether Python is already installed

Open PowerShell or Command Prompt:

```powershell
python --version
```

Alternatively:

```powershell
py --version
```

Example:

```text
Python 3.12.7
```

If Python is not found, it needs to be installed.

---

## 4.2 Install Python

Python can be installed from the official Python website.

During installation, make sure to enable:

```text
Add Python to PATH
```

After installation, close and reopen PowerShell or Command Prompt.

Verify:

```powershell
python --version
```

or:

```powershell
py --version
```

Also check:

```powershell
pip --version
```

---

## 4.3 Optional: Use the Python Launcher

On Windows, the `py` launcher is often the most reliable way to select Python.

Examples:

```powershell
py -3 --version
```

or:

```powershell
py -3.12 --version
```

A Virtual Environment can therefore also be created with:

```powershell
py -m venv .venv
```

---

# 5. Windows: Create a Virtual Environment

## 5.1 Change into the project directory

Example:

```powershell
cd C:\Users\MyUserName\Projects\my-project
```

Show the current directory:

```powershell
Get-Location
```

In Command Prompt:

```cmd
cd
```

---

## 5.2 Create the Virtual Environment

Recommended:

```powershell
python -m venv .venv
```

Alternatively:

```powershell
py -m venv .venv
```

The project may then look like this:

```text
my-project/
├── .venv/
├── src/
├── requirements.txt
├── pyproject.toml
└── README.md
```

---

## 5.3 Activate the Virtual Environment in PowerShell

```powershell
.\.venv\Scripts\Activate.ps1
```

The prompt should then start with:

```text
(.venv)
```

Example:

```text
(.venv) PS C:\Users\MyUserName\Projects\my-project>
```

---

## 5.4 Activate the Virtual Environment in CMD

```cmd
.venv\Scripts\activate.bat
```

---

## 5.5 Activate the Virtual Environment in Git Bash

```bash
source .venv/Scripts/activate
```

---

## 5.6 PowerShell blocks `Activate.ps1`

A common error says something similar to:

```text
running scripts is disabled on this system
```

For the current user, the execution policy can be changed with:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

PowerShell will normally ask for confirmation.

Then try again:

```powershell
.\.venv\Scripts\Activate.ps1
```

This setting only applies to the current Windows user.

---

## 5.7 Verify which Python is being used

```powershell
where.exe python
```

The first result should point to the Virtual Environment, for example:

```text
C:\Users\MyUserName\Projects\my-project\.venv\Scripts\python.exe
```

Also check:

```powershell
python --version
```

and:

```powershell
pip --version
```

---

# 6. Install Packages

Once the Virtual Environment is active, first update `pip`:

```bash
python -m pip install --upgrade pip
```

This works the same way on Linux and Windows.

Install one package:

```bash
pip install requests
```

Install multiple packages:

```bash
pip install requests PyQt6 numpy
```

List installed packages:

```bash
pip list
```

---

# 7. Set Up an Existing Project

## 7.1 Project with `requirements.txt`

If the project contains:

```text
requirements.txt
```

run:

```bash
pip install -r requirements.txt
```

---

## 7.2 Project with `pyproject.toml`

If the project is structured as a Python package and contains a `pyproject.toml` file:

```bash
pip install -e .
```

The `-e` option means **editable installation**.

This means:

```text
Change source code
        ↓
no reinstall required
        ↓
changes are used immediately
```

A reinstall is usually only required when dependencies in `pyproject.toml` change.

---

## 7.3 Start the application

Single Python file:

```bash
python main.py
```

Python package:

```bash
python -m my_package
```

Example:

```bash
python -m yrail_backend
```

---

# 8. Set Up a GitHub Project

A typical workflow for a newly cloned Python project is shown below.

## Linux

```bash
cd ~/projects

git clone git@github.com:USERNAME/REPOSITORY.git

cd REPOSITORY

python3 -m venv .venv

source .venv/bin/activate

python -m pip install --upgrade pip
```

Then, depending on the project:

```bash
pip install -r requirements.txt
```

or:

```bash
pip install -e .
```

---

## Windows PowerShell

```powershell
cd C:\Users\MyUserName\Projects

git clone git@github.com:USERNAME/REPOSITORY.git

cd REPOSITORY

py -m venv .venv

.\.venv\Scripts\Activate.ps1

python -m pip install --upgrade pip
```

Then:

```powershell
pip install -r requirements.txt
```

or:

```powershell
pip install -e .
```

---

# 9. Deactivate and Reactivate the Environment

## Deactivate

On both Windows and Linux:

```bash
deactivate
```

The

```text
(.venv)
```

prefix disappears.

---

## Next time you work on the project

You do **not** need to recreate the environment.

### Linux

```bash
cd ~/projects/my-project
source .venv/bin/activate
```

### Windows PowerShell

```powershell
cd C:\Users\MyUserName\Projects\my-project
.\.venv\Scripts\Activate.ps1
```

### Windows CMD

```cmd
cd C:\Users\MyUserName\Projects\my-project
.venv\Scripts\activate.bat
```

You can then start working immediately.

---

# 10. `.gitignore`

The Virtual Environment should normally **not be committed to Git**.

Add this to `.gitignore`:

```gitignore
.venv/
```

Other common Python entries include:

```gitignore
.venv/
__pycache__/
*.py[cod]
*.egg-info/
dist/
build/
.pytest_cache/
.mypy_cache/
```

Dependencies should instead be defined using files such as:

```text
requirements.txt
```

or:

```text
pyproject.toml
```

---

# 11. Common Problems

## `python: command not found` on Linux

Try:

```bash
python3 --version
```

Outside a Virtual Environment, many Linux distributions only provide `python3`.

---

## `python is not recognized` on Windows

Possible causes:

- Python is not installed.
- Python was not added to `PATH`.
- The terminal was not reopened after installation.

Try:

```powershell
py --version
```

If `py` works, create the Virtual Environment with:

```powershell
py -m venv .venv
```

---

## `No module named venv` on Linux

Install the venv package:

```bash
sudo apt install python3-venv
```

Then:

```bash
python3 -m venv .venv
```

---

## `pip` installs packages globally

First verify that the Virtual Environment is active.

Linux:

```bash
which python
which pip
```

Windows:

```powershell
where.exe python
where.exe pip
```

Both should point to `.venv` first.

---

## PowerShell does not allow scripts to run

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Then:

```powershell
.\.venv\Scripts\Activate.ps1
```

---

## The Virtual Environment was moved

A Virtual Environment should generally **not be moved between folders or computers**.

It is better to recreate it:

```text
delete old .venv
        ↓
create a new .venv
        ↓
reinstall dependencies
```

Linux:

```bash
rm -rf .venv
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Windows PowerShell:

```powershell
Remove-Item -Recurse -Force .venv
py -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

---

# 12. Quick Reference

## Linux

Create a new environment:

```bash
cd ~/projects/my-project
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
```

Activate an existing environment:

```bash
source .venv/bin/activate
```

Deactivate:

```bash
deactivate
```

---

## Windows PowerShell

Create a new environment:

```powershell
cd C:\Users\MyUserName\Projects\my-project
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
```

Activate an existing environment:

```powershell
.\.venv\Scripts\Activate.ps1
```

Deactivate:

```powershell
deactivate
```

---

# Recommended Standard Workflow

For almost every new Python project:

```text
1. Clone the repository or create the project directory
2. Change into the project directory
3. Create .venv
4. Activate .venv
5. Update pip
6. Install dependencies
7. Start the project
```

The `.venv` remains local to each computer and is not synchronized through Git.
