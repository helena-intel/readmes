This guide walks you through creating a virtual environment in Python. There are many tools that can also do this, but in this guide
we use the built-in `venv` module in Python.

# Windows

## Prerequisites

Use a Python installer from python.org https://www.python.org/downloads/windows/. Do not use Python from Windows store. 

## Create a virtual environment

`python -m venv c:\path\to\venv`

## Activate the virtual environment:

`\path\to\venv\Scripts\activate`

## Upgrade pip (highly recommended)

`python -m pip install --upgrade pip`

## Different Python versions

The easiest way to handle multiple Python versions is just to install multiple versions of Python with the Windows installer 
from python.org. You will automatically get the `py` executable which let's you pick the Python version. e.g.
: `py -3.8 -m venv \path\to\venv`. That creates  a virtual environment with Python 3.8.

# Ubuntu/Debian

## Prerequisites

On a fresh Ubuntu install, you may have only a minimum Python install. Make sure to install `python3-venv`. If you're on
a new Ubuntu system that doesn't have `python3-venv` yet, you'll usually also want to install `python3-dev`: `sudo apt install 
python3-venv python3-dev`.

## Create a virtual environment

`python3 -m venv /path/to/venv`

## Activate the virtual environment:

`source /path/to/venv/bin/activate`

## Upgrade pip (highly recommended)

`pip install --upgrade pip`

## Different Python versions

The easiest way to handle multiple Python versions on Ubuntu is to add the deadsnakes repository:

```
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
```

After doing that once, you can install different Python versions with: `sudo apt install python3.11 python3.11-venv python3.11-dev`.

To create a virtual environment with this Python version, use: `python3.11 -m venv /path/to/venv`. 

> Note that deadsnakes is not recommended for high-security production environments.
