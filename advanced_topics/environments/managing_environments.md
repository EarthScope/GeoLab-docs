# Creating Your Own Python Environment in GeoLab

Python projects rely on add-on packages, which are collections of code that do useful things like process data, make maps, or analyze signals. Different projects often need different packages, or even different versions of the same package. An **environment** is a way to keep all of that organized in one tidy, self-contained workspace.

This guide shows you how to create your own environment in GeoLab.

---

## What Is a Package Manager?

GeoLab uses a tool called **conda** to install and manage packages. Think of conda like an app store for Python packages: you tell it what you want, it figures out what else is needed to make it work, and installs everything together.

> **Heads up:** GeoLab resets when you sign out, so any packages you installed during a session won't be there next time. The solution is to define your environment in a file (explained below) so you can recreate it anytime.

---

## See What's Already Installed

Open the **terminal** in GeoLab and try these commands to get your bearings.

**See all available environments:**

```bash
conda env list
```

The one with a `*` is the currently active environment:

```text
# conda environments:
#
base                  *  /srv/conda
notebook                 /srv/conda/envs/notebook
custom_environment       /home/jovyan/.conda/envs/custom_environment
```

**See all packages in the current environment:**

```bash
conda list
```

This prints a long list. Each row shows a package name, its version, and where it came from:

```text
# Name                    Version                   Build  Channel
cartopy                   0.24.1          py312h78ddc71_0    conda-forge
numpy                     2.2.4           py312h7e3fe57_0    conda-forge
obspy                     1.4.1           py312h7b8d3f4_0    conda-forge
xarray                    2025.3.0           pyhd8ed1ab_0    conda-forge
…
```

---

## Installing a Package Without Rebuilding

If you just need to add one package quickly, you can install it directly from inside a notebook cell using a special `%` command. This installs into the notebook's active environment:

```python
%conda install pandas
```

Or, for packages only available on PyPI (a different package source):

```python
%pip install some-package
```

> **Warning:** Installing with `%conda` inside a notebook can use a lot of memory. If your notebook becomes unresponsive or the kernel crashes, use the `environment.yml` approach instead, which is more reliable for larger installs.

---

## Create Your Own Environment

The best way to create a custom environment is to write a short file that lists everything you need. Conda reads the file and builds the environment from it. This also means you can recreate the exact same environment later, or share it with a teammate.

### Step 1: Write an `environment.yml` File

Create a new file called `environment.yml` and paste in something like this:

```yml
name: my_environment
channels:
  - conda-forge
dependencies:
  - python=3.11
  - ipykernel
  - numpy
  - matplotlib
```

Here's what each part means:

- **name**: What you want to call your environment.
- **channels**: Where to download packages from (`conda-forge` is a large, reliable source).
- **dependencies**: The packages you want installed.
- `ipykernel` is required so your environment can be used as a notebook kernel, so always include it.

Replace `numpy` and `matplotlib` with whatever packages your project actually needs.

### Step 2: Build the Environment

In the **terminal**, run:

```bash
conda env create -f environment.yml
```

Conda will figure out which versions of everything are compatible and download them. This can take a few minutes, which is normal.

> **If it fails:** Read the error message. Conda usually names the package that's causing the problem. Try removing it from the file or changing its version.

### Step 3: Activate the Environment

```bash
conda activate my_environment
```

Activating an environment switches your terminal into that workspace, so any Python commands you run use that environment's packages. Your terminal prompt will update to show the environment name, confirming it worked.

### Step 4: Register It as a Notebook Kernel

A new environment isn't automatically available in JupyterLab. Run this command **in the terminal** to register it:

```bash
python -m ipykernel install --user --name my_environment --display-name "Python (my_environment)"
```

### Step 5: Switch to Your Environment in a Notebook

1. Open a notebook and go to **Kernel > Change Kernel…**

   ![Kernel menu showing the Change Kernel option](./images/select_kernel.png)

2. Select **Python (my_environment)**.

   ![Kernel selector dialog with custom environment listed](./images/select_custom.png)

3. Check the upper-right corner of the notebook to confirm the kernel changed.

   ![Notebook header showing the active custom environment kernel](./images/custom_env.png)

> **Note:** You have to do this for each notebook separately. There's no way to set it as the default for everything.

---

## Quick Reference

| What you want to do | Command |
| --- | --- |
| See all environments | `conda env list` |
| See installed packages | `conda list` |
| Activate an environment | `conda activate <name>` |
| Leave an environment | `conda deactivate` |
| Build from a file | `conda env create -f environment.yml` |
| Register as a kernel | `python -m ipykernel install --user --name <name> --display-name "<label>"` |
| Install a package (in notebook) | `%conda install <package>` |
| Install from PyPI (in notebook) | `%pip install <package>` |
