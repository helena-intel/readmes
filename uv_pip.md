# `uv pip install` Python packages

Installing packages with `pip install` can be slow,especially on Windows. [uv](https://docs.astral.sh/uv/) massively improves this.
`uv` is a powerful tool, but you do not need to understand or use all of it to leverage the benefits of faster `pip install`.

Another benefit of `uv` is that it makes it very easy to use different versions of Python.

This README explains how to use `uv pip install` as a drop in replacement for `pip install`.

## Install

There are many ways to install `uv`. See the documentation at https://docs.astral.sh/uv/getting-started/installation/

Below are two easy ways using package managers.

### Windows

```
winget install astral-sh.uv
```

### Ubuntu

```
sudo snap install astral-uv --classic
```

## Usage

### Create a virtual environment

You create a venv with `uv venv`. By default this creates a virtualenv in directory `.venv` as a subdirectory of the current directory, and activating the .venv shows the directory name.
You can also optionally specify a directory name for the virtualenv. In that case that name will be shown when you activate the venv. A very useful feature is that you can optionally specify the Python version.

This creates a venv in a directory `venv_310` using Python 3.10:

```
uv venv --python 3.10 venv_310
```

After creating the .venv, `uv` helpfully prints a message with the command line to activate it. This is the same as for virtualenvs
created with `python -m venv`. `venv_310\Scripts\activate` on Windows, `source venv_310/bin/activate` on Linux.

### Install packages

By default the `pip` executable is not installed in the virtual environment. You can optionally install it with `uv pip install pip`. This is primarily useful when you use other tools that hardcode `pip install`.

The regular way to install packages in a `uv` virtual environment is to use `uv pip install`. This is works almost always exactly the way as `pip install` but much faster.

You can use the same command line arguments. If something does not work, you'll get a clear error message with usually a resolution.

Examples:

```
uv pip install openvino
uv pip install torch==2.8.* --extra-index-url https://download.pytorch.org/whl/cpu
uv pip install -r requirements.txt
```

#### Differences from regular `pip install`

Generally, you can just use `uv pip` instead of `pip`, but there are some subtle differences.

- `uv pip uninstall` does not ask for confirmation before uninstalling. `uv pip uninstall -y` raises an error
- `uv` is sometimes a bit more strict than `pip`. For example, it refuses to install a package by default when it can find the package
in `extra-index-url` with the wrong version, and correct version exists on PyPI. In that case it usually tells you how to solve it. In
this case: `uv pip install -r requirements.txt  --index-strategy unsafe-best-match`
- `uv pip install --upgrade` upgrades dependencies too (similar to `pip install --upgrade --upgrade-strategy eager`). To upgrade a package, for example `openvino`, without dependencies, use `uv pip install --upgrade-package openvino openvino` or `uv pip install --upgrade openvino --no-deps`. ([GitHub issue](https://github.com/astral-sh/uv/issues/4779))

See `uv pip install --help` for an overview of all options.

### The end

That's it! You now improved `pip install` speed by a lot.
