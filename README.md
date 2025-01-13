# pip-backtrack-bug-mre

This repo demonstrates a bug with pip backtracking.

# Trigger bug

```bash
python -m venv .venv-bug
source .venv-bug/bin/activate
pip install --upgrade pip
pip install --only-binary :all: -r requirements-bug.txt
```
