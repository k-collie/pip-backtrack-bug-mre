# pip-backtrack-bug-mre

This is a toy repository used to reproduce a bug with pip backtracking. Tested
with `pip-24.3.1` and commit
[a84a9403a493f46ec8c02975d9f832b2f19dc24d](https://github.com/pypa/pip/tree/a84a9403a493f46ec8c02975d9f832b2f19dc24d)
from main.

# Bug

This involves trying to install an unsolveable set of packages. `numpy>=2` is
specified however the `pip-backtrack-bug-mre` toy repository depends on
`numpy<2`. The `--only-binary :all:` argument is not related, it just prevents
attempting to
compile many versions of `scipy`.

Create venv with updated pip:

```bash
python -m venv .venv-bug
source .venv-bug/bin/activate
pip install --upgrade pip
```

attempt install:

```
pip install --only-binary :all: "numpy>=2" scipy git+https://github.com/k-collie/pip-backtrack-bug-mre.git
```

results in:

```console
Collecting git+https://github.com/k-collie/pip-backtrack-bug-mre.git
  Cloning https://github.com/k-collie/pip-backtrack-bug-mre.git to /tmp/pip-req-build-rb_ft5g4
  Running command git clone --filter=blob:none --quiet https://github.com/k-collie/pip-backtrack-bug-mre.git /tmp/pip-req-build-rb_ft5g4
  Resolved https://github.com/k-collie/pip-backtrack-bug-mre.git to commit bd7ed6dd3afbaba593f2364c9cafcc2073d96cef
  Installing build dependencies ... done
  Getting requirements to build wheel ... done
  Preparing metadata (pyproject.toml) ... done
Collecting numpy>=2
  Using cached numpy-2.2.1-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (62 kB)
Collecting scipy
  Using cached scipy-1.15.1-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (61 kB)
INFO: pip is looking at multiple versions of pip-backtrack-bug-mre to determine which version is compatible with other requirements. This could take a while.
  Using cached scipy-1.15.0-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (61 kB)
  Using cached scipy-1.14.1-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (60 kB)
  Using cached scipy-1.14.0-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (60 kB)
  Using cached scipy-1.13.1-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (60 kB)
  Using cached scipy-1.13.0-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (60 kB)
  Using cached scipy-1.12.0-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (60 kB)
INFO: pip is looking at multiple versions of scipy to determine which version is compatible with other requirements. This could take a while.
  Using cached scipy-1.11.4-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (60 kB)
  Using cached scipy-1.11.3-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (60 kB)
  Using cached scipy-1.11.2-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (59 kB)
ERROR: Cannot install numpy>=2, scipy==1.11.2, scipy==1.11.3, scipy==1.11.4 and scipy==1.12.0 because these package versions have conflicting dependencies.

The conflict is caused by:
    The user requested numpy>=2
    scipy 1.12.0 depends on numpy<1.29.0 and >=1.22.4
    The user requested numpy>=2
    scipy 1.11.4 depends on numpy<1.28.0 and >=1.21.6
    The user requested numpy>=2
    scipy 1.11.3 depends on numpy<1.28.0 and >=1.21.6
    The user requested numpy>=2
    scipy 1.11.2 depends on numpy<1.28.0 and >=1.21.6

To fix this you could try to:
1. loosen the range of package versions you've specified
2. remove package versions to allow pip to attempt to solve the dependency conflict

ERROR: ResolutionImpossible: for help visit https://pip.pypa.io/en/latest/topics/dependency-resolution/#dealing-with-dependency-conflicts
```

The bug is that pip attempts to backtrack `scipy`, despite the first
`scipy-1.15.1` version being compatible with `numpy>=2`. The error message then
implies a conflict between `scipy` and `numpy>=2`, despite the true source of
the conflict being between `numpy>=2` and the toy repository.


# Working examples

## Reordering

Moving `scipy` to the end of the line produces a correct error message and no
strange backtracking behaviour:

```bash
pip install --only-binary :all: "numpy>=2" git+https://github.com/k-collie/pip-backtrack-bug-mre.git scipy
```

```console
Collecting git+https://github.com/k-collie/pip-backtrack-bug-mre.git
  Cloning https://github.com/k-collie/pip-backtrack-bug-mre.git to /tmp/pip-req-build-kysvnsef
  Running command git clone --filter=blob:none --quiet https://github.com/k-collie/pip-backtrack-bug-mre.git /tmp/pip-req-build-kysvnsef
  Resolved https://github.com/k-collie/pip-backtrack-bug-mre.git to commit bd7ed6dd3afbaba593f2364c9cafcc2073d96cef
  Installing build dependencies ... done
  Getting requirements to build wheel ... done
  Preparing metadata (pyproject.toml) ... done
Collecting numpy>=2
  Using cached numpy-2.2.1-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (62 kB)
Collecting scipy
  Using cached scipy-1.15.1-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (61 kB)
INFO: pip is looking at multiple versions of pip-backtrack-bug-mre to determine which version is compatible with other requirements. This could take a while.
ERROR: Cannot install numpy>=2 and pip-backtrack-bug-mre==0.1.0 because these package versions have conflicting dependencies.

The conflict is caused by:
    The user requested numpy>=2
    pip-backtrack-bug-mre 0.1.0 depends on numpy<2.0

To fix this you could try to:
1. loosen the range of package versions you've specified
2. remove package versions to allow pip to attempt to solve the dependency conflict

ERROR: ResolutionImpossible: for help visit https://pip.pypa.io/en/latest/topics/dependency-resolution/#dealing-with-dependency-conflicts
```

## Removing toy repo

Removing the toy repository allows pip to solve:

```bash
pip install --only-binary :all: "numpy>=2" scipy
```
