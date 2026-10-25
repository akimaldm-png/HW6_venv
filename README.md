# HW6: Running `pain.py`

Instructions for setting up an environment in which `pain.py` runs without modifying the script.

## System

The solution was tested on:

- **OS:** macOS (Apple Silicon, `osx-arm64`)
- **Package manager:** conda 26.9.4 + mamba 2.9.0

## Why Python 3.14

`pain.py` uses language and standard library features that appeared only in recent Python versions:

| Line | Feature | Minimum Python |
|---|---|---|
| 4, 28 | `typing.override` | 3.12 |
| 12 | nested f-strings with the same quotes | 3.12 |
| 7 | `glob.translate` | 3.13 |
| 24 | `typing.TypeIs` | 3.13 |
| 15 | `annotationlib` module | 3.14 |

The strictest requirement is `annotationlib`, so Python 3.14 is needed. This also determines the rest of the setup: the old versions of Biopython and pandas that the script was written for are not available for Python 3.14 (see steps 2 and 3).

## Requirements

- [Miniconda](https://docs.conda.io/en/latest/miniconda.html) or Anaconda
- `mamba` (installed into the `base` environment):

```bash
conda install -n base -c conda-forge mamba
```

## Steps

All commands are run from the root of this repository.

### 1. Create and activate the environment

```bash
mamba env create -f environment.yaml
conda activate hw6
```

`conda activate` is used instead of `mamba activate`, because the latter requires a separate shell initialization.

Check that the environment is active:

```bash
python --version   # Python 3.14.9
```

### 2. Restore the `Bio.SubsMat` module

`pain.py` imports `Bio.SubsMat`, which was removed from Biopython after version 1.79. Biopython 1.79 itself cannot be installed, because it supports only Python < 3.10. So the module is copied from the Biopython 1.79 source code into the installed Biopython 1.88:

```bash
curl -L -o /tmp/biopython-179.tar.gz https://github.com/biopython/biopython/archive/refs/tags/biopython-179.tar.gz
tar -xzf /tmp/biopython-179.tar.gz -C /tmp
cp -r /tmp/biopython-biopython-179/Bio/SubsMat "$CONDA_PREFIX/lib/python3.14/site-packages/Bio/"
```

`$CONDA_PREFIX` is the path to the active environment, so the command works regardless of where conda is installed.

### 3. Patch pandas

Since pandas 1.5, a `DataFrame` cannot be created with a `set` as its index (line 46 of `pain.py`). pandas < 1.5 is not available for Python 3.14, so the check in the installed pandas is changed: instead of raising an error, the set is converted to a list, as in older pandas versions.

```bash
sed -i.bak 's/raise ValueError("index cannot be a set")/index = list(index)/' "$CONDA_PREFIX/lib/python3.14/site-packages/pandas/core/frame.py"
```

A backup of the original file is saved as `frame.py.bak`. The patch relies on the exact text of pandas 3.1.0, which is why the version is pinned in `environment.yaml`.

Check the result (the line after `if isinstance(index, set):` should now be `index = list(index)`):

```bash
grep -n -A1 "isinstance(index, set)" "$CONDA_PREFIX/lib/python3.14/site-packages/pandas/core/frame.py"
```

### 4. Run the script

```bash
python pain.py
```

Expected output:

```
✨❤ Всё работает, ты больш(ой/ая) молодец!!! ✨❤
```

## Why these packages

| Package | Version | Why it is needed |
|---|---|---|
| `python` | 3.14.9 | see [Why Python 3.14](#why-python-314) |
| `pandas` | 3.1.0 | `import pandas` |
| `pyarrow` | 25.0.0 | `dtype="float32[pyarrow]"` requires `pyarrow>=16` |
| `protobuf` | 7.35.1 | provides the `google` namespace package (`import google`) |
| `opencv` | 5.0.0 | provides the `cv2` module |
| `biopython` | 1.88 | provides `Bio` and `Bio.pairwise2` |

## Part 2

Here is how I rested this week:

![A couple of episodes](img/housemd.jpg)
![Coffee and buns under the autumn sun](img/coffee.jpg)
