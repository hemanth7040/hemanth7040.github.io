---
title: "Setting Up Python and VS Code"
date: 2026-04-19T10:00:00+05:30
draft: false
categories: ["python"]
description: "A complete beginner-friendly guide to installing Python, setting up VS Code, creating virtual environments, and running your first Python file — everything you need before writing a single line of AI code."
video_url: "https://youtu.be/ygXn5nV5qFc?si=2ai-v1UT8qYrwvev"
series: ["Python Zero to MLOps"]
series_order: 2
tags:  [ "Vscode", "Setup", "Beginners", "Ai"]
---

{{< youtube "ygXn5nV5qFc" >}}

> **Video Reference:** Watch the full course by Dave Ebbelaar on YouTube before or alongside this guide.

# Setting Up Python and VS Code

Before you write any Python code, you need two things on your computer: **Python** (the programming language) and **VS Code** (the editor where you write your code). This guide walks you through every step — from a fresh computer to running your first Python file.

---

## What You Will Have by the End

- Python installed on your computer
- VS Code installed and configured
- A working project folder
- A virtual environment set up
- Your first Python package installed
- Jupyter notebooks running

---

## Step 1 — Install Python

Python is the language your computer uses to understand and run your code. Think of it like installing a new language on your computer so it can "speak Python."

### On Windows

1. Open your browser and go to [python.org/downloads](https://www.python.org/downloads/)
2. Click the big yellow **"Download Python"** button — it picks the right version for you automatically
3. Run the downloaded `.exe` file
4. **Very important:** On the first screen, check the box that says **"Add Python to PATH"** before clicking Install

   > If you miss this step, Windows will not be able to find Python when you type commands. You would need to uninstall and reinstall.

5. Click **Install Now** and wait for it to finish
6. Click **Close** when done

**Check that it worked:** Open the Start menu, search for **Command Prompt**, and type:

```
python --version
```

You should see something like `Python 3.12.0`. If you see that, Python is installed correctly.

---

### On Mac

Mac comes with an old version of Python already installed, but you need the modern version.

1. Go to [python.org/downloads](https://www.python.org/downloads/)
2. Download the `.pkg` file for Mac
3. Open the downloaded file and follow the installer steps (Next → Agree → Install)
4. When finished, open **Terminal** (press `Cmd + Space`, type Terminal, press Enter)
5. Type this and press Enter:

```
python3 --version
```

You should see `Python 3.12.0` or similar.

> **On Mac, always use `python3` instead of `python`.** This is normal — it's how Mac separates the old built-in Python from the new one you just installed.

---

## Step 2 — Install VS Code

VS Code (Visual Studio Code) is where you will write all your Python code. It is free, very popular, and works great for AI development.

1. Go to [code.visualstudio.com](https://code.visualstudio.com/)
2. Click the big **Download** button — it detects your operating system automatically
3. Run the installer and follow the steps
4. When you reach the screen with options, check these boxes:
   - "Add to PATH"
   - "Register Code as an editor for supported file types"
5. Click **Install**, then **Finish**

After installation, open VS Code. You will see a welcome screen.

---

## Step 3 — Set Up VS Code Extensions

VS Code on its own does not know how to work with Python. You need to install **extensions** — small add-ons that give VS Code superpowers.

### Install the Python Extension

1. Inside VS Code, look at the left sidebar
2. Click the icon that looks like four squares (Extensions), or press `Ctrl+Shift+X` (Windows) / `Cmd+Shift+X` (Mac)
3. In the search box, type **Python**
4. The first result should be **"Python"** by Microsoft — click **Install**

This extension gives VS Code the ability to run Python files, show errors as you type, and offer suggestions while you code.

### Install the Pylance Extension

While you are in the Extensions panel, also search for and install **Pylance** (also by Microsoft). This adds smarter code suggestions and catches more errors before you run your code.

### Install the Jupyter Extension

Search for **Jupyter** (by Microsoft) and install it. You will need this later for running interactive notebooks.

---

## Step 4 — Customize VS Code (Optional but Recommended)

These settings make VS Code more comfortable to use every day.

### Choose a Color Theme

1. Press `Ctrl+Shift+P` (Windows) / `Cmd+Shift+P` (Mac) to open the Command Palette
2. Type **Color Theme** and press Enter
3. Browse through the themes and press Enter on one you like
4. "Dark+" and "One Dark Pro" are popular choices for coding

### Change the Font Size

If the text is too small or too large:

1. Go to **File → Preferences → Settings** (or press `Ctrl+,`)
2. Search for **Font Size**
3. Change the number to your preference (14 or 16 is comfortable for most people)

### Turn On Auto Save

This saves your file automatically every few seconds so you never lose work:

1. Go to **File → Auto Save** and click it to enable it

---

## Step 5 — Create Your First Project

A project is just a folder on your computer where all your code files live. Keeping everything organized in one folder is very important.

### Create the Folder

1. Open **File Explorer** (Windows) or **Finder** (Mac)
2. Navigate to a place you will remember — for example, your Desktop or Documents folder
3. Create a new folder and name it something like `python-ai-course`

### Open the Folder in VS Code

**Option A — From VS Code:**
1. Open VS Code
2. Click **File → Open Folder**
3. Navigate to and select your `python-ai-course` folder
4. Click **Open**

**Option B — From the terminal:**
1. Open Terminal (Mac) or Command Prompt (Windows)
2. Type `cd Desktop` (or wherever you created the folder)
3. Type `code python-ai-course`

You should now see your empty folder on the left side of VS Code.

### Create a VS Code Workspace File (Recommended)

A workspace file saves your VS Code settings specifically for this project.

1. Go to **File → Save Workspace As**
2. Save it inside your project folder with the name `python-ai-course.code-workspace`
3. Click **Save**

Next time you want to open this project, just double-click the `.code-workspace` file.

---

## Step 6 — Write and Run Your First Python File

Now for the exciting part — writing your first code.

### Create the File

1. In VS Code, look at the left panel (Explorer)
2. Click the **New File** icon (a page with a + symbol)
3. Name the file `hello.py`

> The `.py` extension tells VS Code and your computer that this is a Python file.

### Write Your First Code

Click inside the file and type:

```python
print("Hello, Python!")
print("I am learning Python for AI development.")
```

`print()` is a Python function that displays text on the screen. Whatever you put inside the quotes will appear when you run the code.

### Run the File

**Method 1 — The Play Button:**
Look at the top-right corner of VS Code. You will see a small triangle (▶) play button. Click it.

**Method 2 — The Terminal:**
1. Go to **Terminal → New Terminal** in the top menu
2. A terminal panel opens at the bottom of VS Code
3. Type this and press Enter:

```
python hello.py
```

(On Mac: `python3 hello.py`)

You should see this output:

```
Hello, Python!
I am learning Python for AI development.
```

**Congratulations — you just ran your first Python program.**

---

## Step 7 — Understand Python Environments

This is one of the most important concepts for working with Python seriously. It sounds technical, but the idea is simple.

### The Problem

Imagine you are working on two different projects:
- Project A needs version 1 of a library
- Project B needs version 2 of the same library

If you install everything in one place, the two projects will conflict and break each other.

### The Solution — Virtual Environments

A **virtual environment** is like a separate, isolated box for each project. Each project gets its own copy of Python and its own set of libraries. Changes in one box do not affect another.

Think of it like having separate toolboxes for different jobs — you don't want your kitchen tools mixed up with your garage tools.

---

## Step 8 — Create a Virtual Environment

### Open the Terminal in VS Code

Go to **Terminal → New Terminal**. Make sure the terminal is inside your project folder (you will see the folder name in the terminal prompt).

### Create the Environment

Type this command and press Enter:

```
python -m venv venv
```

(On Mac: `python3 -m venv venv`)

This creates a folder called `venv` inside your project. This folder contains its own separate Python installation.

> You will see a new `venv` folder appear in the VS Code Explorer panel on the left. Do not edit files inside this folder — it is managed automatically.

### Activate the Environment

Activation means telling your terminal "use the Python inside this venv folder."

**On Windows:**
```
venv\Scripts\activate
```

**On Mac/Linux:**
```
source venv/bin/activate
```

After activation, your terminal prompt changes to show `(venv)` at the beginning:

```
(venv) C:\Users\You\python-ai-course>
```

This tells you the virtual environment is active.

> **Every time you open a new terminal in this project, you need to activate the environment again.**

### Tell VS Code to Use This Environment

1. Press `Ctrl+Shift+P` / `Cmd+Shift+P`
2. Type **Python: Select Interpreter**
3. Press Enter
4. You will see a list — choose the one that shows `./venv/Scripts/python.exe` (Windows) or `./venv/bin/python` (Mac)

VS Code will now automatically use your virtual environment when running files.

---

## Step 9 — Install Python Packages with pip

Python's real power comes from **packages** — ready-made code written by other developers that you can use in your projects.

**pip** is the tool you use to install packages. Think of it like an app store, but for Python code.

### Install a Package

With your virtual environment activated, type:

```
pip install requests
```

`requests` is a popular package for working with APIs (you will use it a lot in AI development). You will see text scrolling by as pip downloads and installs the package.

### Check What Is Installed

```
pip list
```

This shows every package installed in your current virtual environment.

### Save Your Package List

When you share your project with someone else (or set it up on a new computer), they need to know which packages to install. The standard way to do this is a `requirements.txt` file.

Create one with:

```
pip freeze > requirements.txt
```

Open `requirements.txt` in VS Code — you will see all your installed packages listed. Another person can install everything at once with:

```
pip install -r requirements.txt
```

---

## Step 10 — Use Python Packages in Your Code

Installing a package is only the first step. You also need to **import** it in your Python file before you can use it.

Create a new file called `test_import.py` and type:

```python
import math
import random

# Use the math package
print(math.sqrt(16))     # prints 4.0
print(math.pi)           # prints 3.14159...

# Use the random package
print(random.randint(1, 10))   # prints a random number between 1 and 10
```

`math` and `random` are built into Python — you do not need to install them. Run the file and see the output.

Now try importing the package you installed:

```python
import requests

response = requests.get("https://httpbin.org/get")
print(response.status_code)   # prints 200 if successful
```

---

## Step 11 — Set Up Jupyter Notebooks

Jupyter notebooks are a special way to write Python code where you can run small pieces (called **cells**) one at a time and see the output immediately below each cell. They are very popular in AI and data science because they make it easy to experiment.

### Create a Notebook

1. In VS Code, press `Ctrl+Shift+P` / `Cmd+Shift+P`
2. Type **Create: New Jupyter Notebook** and press Enter
3. A new file opens with the `.ipynb` extension

### Run a Cell

1. Click the first cell
2. Type: `print("Hello from Jupyter!")`
3. Click the ▶ button on the left of the cell, or press `Shift+Enter`

The output appears directly below the cell.

### Add a New Cell

Press `Shift+Enter` to run the current cell and automatically move to a new one. Or click the **+ Code** button at the top.

---
### Complete Setup Checklist

{{< checklist "Python installed — `python --version` shows a version number" >}}
{{< checklist "VS Code installed and opens correctly" >}}
{{< checklist "Python extension installed in VS Code" >}}
{{< checklist "Pylance extension installed" >}}
{{< checklist "Jupyter extension installed" >}}
{{< checklist "Project folder created and opened in VS Code" >}}
{{< checklist "Virtual environment created (`venv` folder exists)" >}}
{{< checklist "Virtual environment activated (`(venv)` shows in terminal)" >}}
{{< checklist "VS Code is using the correct Python interpreter" >}}
{{< checklist "pip list works and shows packages" >}}
{{< checklist "hello.py runs and prints output" >}}
{{< checklist "Jupyter notebook opens and a cell runs successfully" >}}


---

## Common Problems and Fixes

### "python is not recognized as a command" (Windows)

This means Python was not added to PATH during installation. Uninstall Python and reinstall it, making sure to check **"Add Python to PATH"** on the first screen.

### VS Code cannot find the Python interpreter

Press `Ctrl+Shift+P`, type **Python: Select Interpreter**, and manually browse to the `python.exe` file inside your `venv` folder.

### pip install gives a permissions error (Mac)

Make sure your virtual environment is activated first. You should see `(venv)` in your terminal before running pip commands.

### The terminal does not show `(venv)` after activation

Make sure you typed the activation command correctly. On Windows use backslashes: `venv\Scripts\activate`. On Mac use forward slash: `source venv/bin/activate`.

---

## What Comes Next

Your setup is complete. You now have a professional Python development environment — the same setup used by real AI engineers. In the next section, you will start learning Python's core concepts: variables, data types, functions, and more.

> **Tip:** Keep this page bookmarked. You will need to come back to the setup steps whenever you start a new project — especially the virtual environment steps.