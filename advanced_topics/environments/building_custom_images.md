# Building a Custom GeoLab Image

A GeoLab image is a complete, prepackaged computing environment, including Python, JupyterLab, and all the scientific packages your project needs, bundled together so anyone can run them. When you start a GeoLab session, it launches from an image. This guide walks you through building your own.

---

## How It Works

Think of an **image** as a recipe with each Python package as an ingredient. An **image** is the meal created from the recipe. Finally, a **container** is the meal served on a plate. The recipe doesn't change and you can make the same meal over and over. GeoLab does the same thing: it takes your image and launches a fresh session from it every time.

You define the image by editing a few plaintext files that list what you want installed. A tool called Docker reads those files and builds the image. You then publish it so GeoLab can access it.

```text
Config files  →  Docker builds  →  Image  →  GeoLab runs it
```

---

## Before You Start

> **Operating-system note:** The commands below use a macOS/Linux shell. Windows users should run them from a WSL 2 terminal with Docker Desktop’s WSL integration enabled. PowerShell equivalents may differ.


You need two things installed on **your computer**:

1. **Docker Desktop**, download it at [docker.com](https://www.docker.com/products/docker-desktop/). Install it on your computer, open it and leave it running in the background.
2. **Git**, to download the template.

You also need a **GitHub** account at [github.com](https://github.com), where you'll publish your image using the GitHub Container Registry (`ghcr.io`). If you already use GitHub for code, you can use the same account.

Before you can push images, you need a **Personal Access Token (PAT)** with package permissions:

1. Go to **GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)**.
2. Click **Generate new token (classic)**.
3. Give it a name (e.g. `geolab-image`), set an expiration, and check the **`write:packages`** scope.
4. Click **Generate token** and copy it, as you won't be able to see it again.

Save your token somewhere safe (a password manager works well). You'll use it to log in to the registry in Step 4.

Verify Docker is working by opening a terminal and running:

```bash
docker --version
```

If it prints a version number, you're good to go.

---

## Step 1: Get the Template

EarthScope provides a starter template. Download it and set up your working folder:

```bash
git clone --depth 1 https://github.com/EarthScope/GeoLab.git
cp -R GeoLab/geolab-base my-geolab-image
cd my-geolab-image
```

Your `my-geolab-image` folder contains these files:

```text
my-geolab-image/
├── Dockerfile          ← do not edit this
├── environment.yml     ← add your conda packages here
├── requirements.txt    ← add PyPI-only packages here
├── apt.txt             ← add system software here (rarely needed)
├── start               ← do not edit this
├── test_packages.py    ← smoke test for installed packages
└── test_notebook.ipynb ← interactive version of the smoke test
```

> **The only files you need to edit are `environment.yml`, `requirements.txt`, and `apt.txt`.** Everything else is set up for you.

---

## Step 2: Add Your Packages

### `environment.yml`: Your Main Package List

This is where you add Python packages. Open the file and add packages under the `dependencies` section:

> **ALERT:** Only add packages that are not already listed. Do not replace the existing file.

```yaml
channels:
  - conda-forge
  - nodefaults
dependencies:
  - python=3.12
  # --- Geophysics ---
  - obspy
  - pygmt
  # --- Geospatial ---
  - cartopy
  - geopandas
  # add your packages below:
  - my-package-name
```

Use conda packages whenever you can, as conda checks that everything works together before installing.

### `requirements.txt`: Packages Only on PyPI

Some packages aren't available through conda and must be installed from PyPI. Add them here, one per line:

> **ALERT:** Keep the EarthScope SDK version already provided by the current template. Add only your additional PyPI package below it.

```text
earthscope-sdk
my-pypi-package
```

### `apt.txt`: System Software (Rarely Needed)

Most scientific packages go in `environment.yml`. Only use `apt.txt` for low-level system tools that can't be installed any other way:

```text
build-essential
git
```

---

## Step 3: Build and Test Locally

Before publishing, build the image on your own computer and make sure everything works.

**Build the image:**

```bash
docker build -f Dockerfile --tag my-geolab-image:0.1.0 .
```

This reads your config files and assembles the image. It can take several minutes the first time.

**Run it:**

```bash
docker run --rm -p 8888:8888 my-geolab-image:0.1.0
```

Look in the output for a line like:

```text
http://127.0.0.1:8888/lab?token=...
```

Copy that URL into a browser. You'll see a JupyterLab session running from your image.

**Test that your packages installed correctly:**

**test_notebook.ipynb** checks to see if base packages were installed correctly. It is recommended that you write a test for any packages you install. Add a new cell before **Summary** for you tests.

There are two test functions:

- `py` imports the package and optionally runs a test as a sanity check.
- `cli` verifies the client package is available and responds to a version flag.

This is an example of a `py` test for gnss-lib-py:

```python
py('gnss_lib_py')
```

This is an example of a `cli` test using the earthscope-cli:

```bash
cli(es, version_flag='--version')
```

Each package gets a pass or fail which is display in the **Summary** cell of the notebook. If something fails, it usually means a package name is misspelled or a version is unavailable, so go back to `environment.yml` or `requirements.txt` and fix it, then rebuild.

---

## Step 4: Publish Your Image

Once the local test passes, rebuild the image for GeoLab's platform and push it to the GitHub Container Registry.

**Rebuild for GeoLab's platform:**

```bash
docker build --no-cache -f Dockerfile \
  --platform linux/amd64 \
  --tag ghcr.io/your-github-username/my-geolab-image:0.1.0 .
```

Replace `your-github-username` with your GitHub username.

> **Why `--platform linux/amd64`?** GeoLab runs on Linux. If you're on a Mac with Apple Silicon, your local machine uses a different architecture. This flag ensures the image works on GeoLab regardless of what you built it on.

**Log in to the GitHub Container Registry and push:**

```bash
echo YOUR_GITHUB_TOKEN | docker login ghcr.io -u your-github-username --password-stdin
docker push ghcr.io/your-github-username/my-geolab-image:0.1.0
```

Replace `YOUR_GITHUB_TOKEN` with the token you created in the prerequisites and `your-github-username` with your GitHub username.

**Make the image public:**

By default, images published to the GitHub Container Registry are private. GeoLab needs to be able to pull it, so you must make it public:

1. Go to your GitHub profile and click **Packages**.
2. Click on `my-geolab-image`.
3. Click **Package settings** (bottom of the right sidebar).
4. Scroll to the **Danger Zone** and click **Change visibility → Public**.

---

## Step 5: Launch It in GeoLab

1. Go to [earthscope.org/data/geolab](https://www.earthscope.org/data/geolab/) and click **Launch GeoLab**.
2. Enter your username and password, then click **Continue**.
3. If a **Stop My Server** button appears, click it first.
4. Click **Start My Server**.
5. Under **Environment**, choose **Other**.
6. In the **Custom image** field, enter your image name, e.g. `ghcr.io/your-github-username/my-geolab-image:0.1.0`.
7. Click **Start**.

GeoLab will pull your image and launch a session from it. The first launch takes a minute while it downloads; after that it's cached and starts quickly.

---

## Making Changes Later

Edit your config files, then rebuild and push with a new version number:

```bash
docker build --no-cache -f Dockerfile \
  --platform linux/amd64 \
  --tag ghcr.io/your-github-username/my-geolab-image:0.1.1 .

docker push ghcr.io/your-github-username/my-geolab-image:0.1.1
```

> **Always use a new version number** (`0.1.1`, `0.1.2`, etc.) when you rebuild. If you reuse the same tag, GeoLab may load the old cached version instead of your new one.

## Troubleshooting Package Installation

In general, it's best practice to install packages using the conda package manager for the GeoLab image. Conda checks packages for dependencies which ensures that dependency conflicts are resolved in the environment. Conda has a search function to discover packages.

Some packages are only available in PyPI and are installed with the pip package manager. Pip also has a search function to find packages in PyPI.

> **Tip:** Keep mind that installation name and import name can be different (such as, scikit-learn vs sklearn).

### Find If a Conda Package Is Available

To find if a package is available through the conda package manager, you can use `conda search`:

```bash
conda search -c conda-forge #PackageName(e.g., seisbench)
```

### Find If a Package Is Available On PyPI

To find if a package is available through the pip package manager, you can use `the following command.

```bash
python -m pip index versions #PackageName(e.g., seisbench)
```

---

## Quick Reference

| What you want to do | Where to do it |
| --- | --- |
| Add a Python package | `environment.yml` under `dependencies` |
| Add a PyPI-only package | `requirements.txt` |
| Add a system tool | `apt.txt` |
| Build locally for testing | `docker build --tag my-geolab-image:0.1.0 .` |
| Run locally | `docker run --rm -p 8888:8888 my-geolab-image:0.1.0` |
| Test packages | `pytest test_packages.py -v` in the container terminal |
| Build for GeoLab | `docker build --no-cache --platform linux/amd64 --tag ghcr.io/username/image:version .` |
| Publish | `docker push ghcr.io/username/my-geolab-image:0.1.0` |
