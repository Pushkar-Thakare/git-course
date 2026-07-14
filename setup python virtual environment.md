Below is the complete Windows + PowerShell workflow for creating a new Python project with a controlled virtual environment, VS Code, Jupyter notebooks, JupyterLab, Git, and verification at every stage.

# 1. Decide where the project will live

Example:

```text
C:\Users\pushk\Desktop\Git\demo-project
```

Open PowerShell and create it:

```powershell
cd C:\Users\pushk\Desktop\Git

mkdir demo-project
cd demo-project
```

Verify your current location:

```powershell
Get-Location
```

Expected:

```text
C:\Users\pushk\Desktop\Git\demo-project
```

Do not create the environment from `C:\Windows\System32` or some unrelated folder.

---

# 2. Make sure no other virtual environment is active

Look at the prompt.

If it begins with something like:

```text
(.venv) PS C:\...
```

then another environment is active.

Deactivate it:

```powershell
deactivate
```

Verify:

```powershell
echo $env:VIRTUAL_ENV
```

It should print nothing.

This matters because you do not want to accidentally create a new environment using another project’s environment.

---

# 3. Check which Python versions are available

Run:

```powershell
py -0p
```

Ideally, you should see something like:

```text
-V:3.13  C:\Users\pushk\AppData\Local\Programs\Python\Python313\python.exe
-V:3.12  C:\Users\pushk\AppData\Local\Programs\Python\Python312\python.exe
```

Check each version explicitly:

```powershell
py -3.13 --version
py -3.12 --version
```

Example:

```text
Python 3.13.x
Python 3.12.x
```

Also see what plain `python` currently means:

```powershell
where.exe python
python --version
```

For creating a venv, prefer the explicit `py -3.13` or `py -3.12` form instead of relying on plain `python`.

---

# 4. Create the virtual environment

Choose the Python version deliberately.

## Using Python 3.13

```powershell
py -3.13 -m venv .venv
```

## Using Python 3.12

```powershell
py -3.12 -m venv .venv
```

This creates:

```text
demo-project/
└── .venv/
```

You are not reinstalling Python itself. You are creating an isolated environment based on the chosen installed Python version.

Verify that the folder exists:

```powershell
Get-ChildItem -Force
```

You should see:

```text
.venv
```

---

# 5. Inspect which Python created the environment

Before activation, inspect the environment configuration:

```powershell
Get-Content .venv\pyvenv.cfg
```

You should see something similar to:

```text
home = C:\Users\pushk\AppData\Local\Programs\Python\Python313
include-system-site-packages = false
version = 3.13.x
executable = C:\Users\pushk\AppData\Local\Programs\Python\Python313\python.exe
```

This file is one of the best ways to confirm which base Python created the venv.

---

# 6. Activate the environment

In PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

Pay attention to the beginning:

```text
.\.venv
```

Correct:

```powershell
.\.venv\Scripts\Activate.ps1
```

Incorrect:

```powershell
\.venv\Scripts\Activate.ps1
```

After activation, your prompt should look like:

```text
(.venv) PS C:\Users\pushk\Desktop\Git\demo-project>
```

If PowerShell blocks script execution, run this once for the current terminal:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned
```

Then activate again:

```powershell
.\.venv\Scripts\Activate.ps1
```

---

# 7. Verify that the correct environment is active

Run all of these:

```powershell
echo $env:VIRTUAL_ENV
```

Expected:

```text
C:\Users\pushk\Desktop\Git\demo-project\.venv
```

Then:

```powershell
where.exe python
```

The first result must be:

```text
C:\Users\pushk\Desktop\Git\demo-project\.venv\Scripts\python.exe
```

You may see global Python installations underneath it. That is fine. Windows uses the first one.

Check the exact running interpreter:

```powershell
python -c "import sys; print(sys.executable)"
```

Expected:

```text
C:\Users\pushk\Desktop\Git\demo-project\.venv\Scripts\python.exe
```

Check the version:

```powershell
python --version
```

Check whether it is actually a venv:

```powershell
python -c "import sys; print('prefix:', sys.prefix); print('base:', sys.base_prefix)"
```

Inside a venv, these should be different:

```text
prefix: C:\Users\pushk\Desktop\Git\demo-project\.venv
base:   C:\Users\pushk\AppData\Local\Programs\Python\Python313
```

That proves:

```text
Current Python = project venv
Base Python = global installation used to create it
```

---

# 8. Verify pip belongs to this environment

Do not trust plain `pip` blindly.

Run:

```powershell
python -m pip --version
```

Expected path:

```text
...\demo-project\.venv\Lib\site-packages\pip
```

Also:

```powershell
where.exe pip
```

The first result should be:

```text
C:\Users\pushk\Desktop\Git\demo-project\.venv\Scripts\pip.exe
```

Use this form for installing packages:

```powershell
python -m pip install package-name
```

It means:

> Use the pip belonging to this exact Python interpreter.

---

# 9. Upgrade pip

```powershell
python -m pip install --upgrade pip
```

Verify:

```powershell
python -m pip --version
```

---

# 10. Install project packages

For a basic ML/data-science project:

```powershell
python -m pip install ipykernel jupyterlab numpy pandas matplotlib scikit-learn
```

Optional common packages:

```powershell
python -m pip install seaborn plotly openpyxl requests
```

Install TensorFlow, PyTorch, OpenCV, or other large packages only when the project actually needs them.

Check installed packages:

```powershell
python -m pip list
```

Check individual packages:

```powershell
python -m pip show scikit-learn
python -m pip show pandas
python -m pip show jupyterlab
```

The `Location` field should point inside:

```text
demo-project\.venv\Lib\site-packages
```

---

# 11. Verify imports directly

Run:

```powershell
python -c "import numpy; print('numpy:', numpy.__version__)"
python -c "import pandas; print('pandas:', pandas.__version__)"
python -c "import sklearn; print('sklearn:', sklearn.__version__)"
python -c "import matplotlib; print('matplotlib:', matplotlib.__version__)"
python -c "import ipykernel; print('ipykernel:', ipykernel.__version__)"
```

A single combined check:

```powershell
python -c "import sys, numpy, pandas, sklearn, matplotlib; print(sys.executable); print('numpy', numpy.__version__); print('pandas', pandas.__version__); print('sklearn', sklearn.__version__); print('matplotlib', matplotlib.__version__)"
```

If these work, the packages are installed in the selected venv.

Remember:

```text
Installation name: scikit-learn
Import name: sklearn
```

---

# 12. Create a basic project structure

For a mixed ML/software project:

```powershell
mkdir notebooks
mkdir src
mkdir data
mkdir data\raw
mkdir data\processed
mkdir tests
mkdir outputs

New-Item README.md
New-Item .gitignore
New-Item requirements.txt
```

Result:

```text
demo-project/
├── .venv/
├── notebooks/
├── src/
├── data/
│   ├── raw/
│   └── processed/
├── tests/
├── outputs/
├── README.md
├── requirements.txt
└── .gitignore
```

You can also have:

```text
html/
docs/
images/
config/
```

depending on the project.

---

# 13. Configure `.gitignore`

Open it:

```powershell
notepad .gitignore
```

Use:

```gitignore
# Python virtual environment
.venv/

# Python cache
__pycache__/
*.py[cod]

# Jupyter
.ipynb_checkpoints/

# Environment variables and secrets
.env
.env.*

# Test/tool caches
.pytest_cache/
.mypy_cache/
.ruff_cache/

# Local outputs
outputs/
*.log

# OS/editor files
Thumbs.db
.DS_Store
```

Decide separately whether to ignore datasets:

```gitignore
data/raw/
data/processed/
```

Ignore them when they are large, private, generated, or easily downloaded again.

For small course datasets, committing them may be acceptable.

Verify `.venv` is ignored:

```powershell
git check-ignore -v .venv\Scripts\python.exe
```

After Git is initialized, expected output resembles:

```text
.gitignore:2:.venv/    .venv\Scripts\python.exe
```

---

# 14. Record dependencies

After installing the packages:

```powershell
python -m pip freeze > requirements.txt
```

Inspect:

```powershell
Get-Content requirements.txt
```

This records exact installed versions.

Example:

```text
ipykernel==...
jupyterlab==...
matplotlib==...
numpy==...
pandas==...
scikit-learn==...
```

Later, the environment can be reconstructed with:

```powershell
python -m pip install -r requirements.txt
```

Do not commit `.venv`. Commit `requirements.txt`.

---

# 15. Open the project correctly in VS Code

From inside the project folder:

```powershell
code .
```

The `.` means:

> Open this folder as the VS Code workspace.

Avoid opening the parent folder containing many unrelated repositories when possible. Open the actual project root.

---

# 16. Select the correct VS Code interpreter

In VS Code:

```text
Ctrl + Shift + P
→ Python: Select Interpreter
```

Choose:

```text
C:\Users\pushk\Desktop\Git\demo-project\.venv\Scripts\python.exe
```

Do not choose a Python from another project.

You can verify the selected interpreter in the VS Code terminal:

```powershell
python -c "import sys; print(sys.executable)"
```

Expected:

```text
...\demo-project\.venv\Scripts\python.exe
```

VS Code may automatically activate the selected venv when it opens a new terminal.

Check:

```powershell
echo $env:VIRTUAL_ENV
```

---

# 17. Create and run a notebook in VS Code

Create:

```text
notebooks\01_eda.ipynb
```

At the top-right of the notebook, click:

```text
Select Kernel
```

Choose the environment pointing to:

```text
demo-project\.venv\Scripts\python.exe
```

Run this as the first cell:

```python
import sys
import numpy as np
import pandas as pd
import sklearn

print("Python executable:")
print(sys.executable)

print("\nVersions:")
print("NumPy:", np.__version__)
print("pandas:", pd.__version__)
print("scikit-learn:", sklearn.__version__)
```

The executable must be:

```text
C:\Users\pushk\Desktop\Git\demo-project\.venv\Scripts\python.exe
```

That is the definitive notebook-kernel verification.

---

# 18. Use JupyterLab with the same venv

Open PowerShell in the project:

```powershell
cd C:\Users\pushk\Desktop\Git\demo-project
```

Activate:

```powershell
.\.venv\Scripts\Activate.ps1
```

Verify:

```powershell
python -c "import sys; print(sys.executable)"
```

Launch JupyterLab:

```powershell
jupyter lab
```

Because JupyterLab was installed inside the project venv and launched from it, it should use that environment.

In a Jupyter notebook, verify:

```python
import sys
print(sys.executable)
```

Expected:

```text
...\demo-project\.venv\Scripts\python.exe
```

Stop JupyterLab from the terminal with:

```text
Ctrl + C
```

Confirm shutdown if prompted.

---

# 19. Optional: register a named Jupyter kernel

Usually, VS Code can detect `.venv` directly, and JupyterLab launched from the same venv works without global registration.

Register a named kernel only when needed:

```powershell
python -m ipykernel install --user `
  --name demo-project `
  --display-name "Python (.venv demo-project)"
```

Then notebook tools may show:

```text
Python (.venv demo-project)
```

List registered kernels:

```powershell
jupyter kernelspec list
```

Be aware that registered kernels can become stale if you delete or move the project.

Remove a stale kernel:

```powershell
jupyter kernelspec uninstall demo-project
```

For clean control, selecting the project `.venv` directly in VS Code is often simpler.

---

# 20. Initialize Git

From the project root:

```powershell
git init
```

Check:

```powershell
git status
```

Verify `.venv` is not listed among files to be committed.

Check specifically:

```powershell
git status --ignored
```

You should see `.venv/` under ignored files, not untracked files.

First commit:

```powershell
git add .
git status
git commit -m "Set up project structure and Python environment"
```

Remember: `.venv` is ignored, so `git add .` should not add it.

---

# 21. Connect to GitHub

After creating an empty GitHub repository:

```powershell
git remote add origin https://github.com/YOUR_USERNAME/demo-project.git
git branch -M main
git push -u origin main
```

Verify:

```powershell
git remote -v
git branch -vv
```

---

# 22. Normal workflow when reopening the project

Open PowerShell:

```powershell
cd C:\Users\pushk\Desktop\Git\demo-project
```

Activate:

```powershell
.\.venv\Scripts\Activate.ps1
```

Verify quickly:

```powershell
python --version
python -c "import sys; print(sys.executable)"
```

Open VS Code:

```powershell
code .
```

Or launch JupyterLab:

```powershell
jupyter lab
```

When finished:

```powershell
deactivate
```

---

# 23. Quick verification checklist

Whenever something feels wrong, run:

```powershell
echo $env:VIRTUAL_ENV
where.exe python
where.exe pip
python --version
python -c "import sys; print(sys.executable)"
python -m pip --version
Get-Content .venv\pyvenv.cfg
```

What each tells you:

| Command                   | What it answers                                              |
| ------------------------- | ------------------------------------------------------------ |
| `echo $env:VIRTUAL_ENV`   | Which venv is active?                                        |
| `where.exe python`        | Which Python executables can Windows see, in priority order? |
| `python --version`        | Which Python version is currently running?                   |
| `sys.executable`          | Exact path of the active interpreter                         |
| `python -m pip --version` | Which pip belongs to the active Python?                      |
| `pyvenv.cfg`              | Which base Python created this venv?                         |

Inside a notebook:

```python
import sys
print(sys.executable)
```

That is the notebook equivalent of checking `where.exe python`.

---

# 24. Changing the Python version later

A venv is tied to the Python version that created it.

Do not try to convert a 3.12 venv into a 3.13 venv in place.

First save dependencies:

```powershell
python -m pip freeze > requirements.txt
```

Deactivate:

```powershell
deactivate
```

Delete the venv:

```powershell
Remove-Item -Recurse -Force .venv
```

Recreate with another version:

```powershell
py -3.12 -m venv .venv
```

or:

```powershell
py -3.13 -m venv .venv
```

Activate:

```powershell
.\.venv\Scripts\Activate.ps1
```

Reinstall:

```powershell
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Verify:

```powershell
python --version
Get-Content .venv\pyvenv.cfg
python -c "import sys; print(sys.executable)"
```

---

# 25. Recreating a cloned project

When you clone a project from GitHub, `.venv` should not be included.

Clone:

```powershell
git clone https://github.com/USERNAME/project.git
cd project
```

Create environment:

```powershell
py -3.13 -m venv .venv
```

Activate:

```powershell
.\.venv\Scripts\Activate.ps1
```

Install recorded packages:

```powershell
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Verify:

```powershell
python -c "import sys; print(sys.executable)"
python -m pip list
```

---

# 26. Common mistakes

## Creating a venv while another venv is active

Check first:

```powershell
echo $env:VIRTUAL_ENV
```

Deactivate before creating a new one:

```powershell
deactivate
```

## Using the wrong activation path

Correct:

```powershell
.\.venv\Scripts\Activate.ps1
```

## Installing with a random pip

Avoid:

```powershell
pip install scikit-learn
```

Prefer:

```powershell
python -m pip install scikit-learn
```

## Selecting the wrong VS Code kernel

Verify inside the notebook:

```python
import sys
print(sys.executable)
```

## Committing `.venv`

Make sure `.gitignore` contains:

```gitignore
.venv/
```

Verify:

```powershell
git check-ignore -v .venv\Scripts\python.exe
```

## Assuming `(.venv)` means the current project’s venv

It might be another project’s venv.

Check:

```powershell
echo $env:VIRTUAL_ENV
```

## Opening the entire parent directory in VS Code

This can make VS Code auto-select another project’s interpreter.

Open the project itself:

```powershell
cd C:\Users\pushk\Desktop\Git\demo-project
code .
```

---

# Compact reusable setup

For a new Python 3.13 ML project:

```powershell
cd C:\Users\pushk\Desktop\Git
mkdir demo-project
cd demo-project

git init

py -3.13 -m venv .venv
.\.venv\Scripts\Activate.ps1

python --version
python -c "import sys; print(sys.executable)"
python -m pip --version

python -m pip install --upgrade pip
python -m pip install ipykernel jupyterlab numpy pandas matplotlib scikit-learn

mkdir notebooks
mkdir src
mkdir data
mkdir tests
mkdir outputs

New-Item README.md
New-Item .gitignore
New-Item requirements.txt

python -m pip freeze > requirements.txt

code .
```

Then add this to `.gitignore`:

```gitignore
.venv/
.ipynb_checkpoints/
__pycache__/
*.pyc
.env
outputs/
```

The central rule is:

```text
Choose the Python version explicitly when creating .venv.
Activate that .venv.
Install packages through that venv’s Python.
Make VS Code and Jupyter use that same .venv.
Verify using sys.executable rather than guessing.
```
