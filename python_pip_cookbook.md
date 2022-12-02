# Python "pip" tool cookbook

## Install a package on an offline system

Download a Python wheel and all it's dependencies from an online device (here numpy package)

```bash
cd wheels-dir/
# download all wheels
pip download numpy
# or for cross version download (here target device run python 3.11), you can format request like this
pip download --python-version 311 --only-binary=:all: numpy
```

Install the package from the wheels directory (package + dependencies) to the offline device

```bash
# "--no-index" don't look at pypi index and look instead at "--find-finds" wheels directory
pip install --no-index --find-links ./wheels-dir numpy
```

## Update Windows 10 Path

Run as "admin", the GUI c:\Windows\System32\SystemPropertiesAdvanced.exe and update system variables "Path".
