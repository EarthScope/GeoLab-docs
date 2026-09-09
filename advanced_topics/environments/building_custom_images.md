# Building a Custom GeoLab Image

A GeoLab image is a complete, prepackaged computing environment that runs in JupyterLab. It includes common geophysics Python and scientific packages bundled together. Starting a GeoLab session launches an image. This step-by-step guide walks through building an image with a customized environment.

---

(how-it-works)=
## How It Works

Think of an **image** as a recipe with each Python package as an ingredient. An **image** is the meal created from the recipe. Finally, a **container** is the meal served on a plate. The recipe doesn't change and you can make the same meal over and over. GeoLab does the same thing: it takes your image and launches a fresh session from it every time.

Install Python packages in an image by editing plaintext files that list the required software. A tool called `Docker` reads those files and builds the image. The image must be published in an image repository so GeoLab can access it. Here is the process:

```text
Edit config files  →  Docker builds  →  Image  →  Push image to repository  →  GeoLab runs it
```

---

(before-starting)=
## Before Starting

Two pieces of software must be installed on **your computer**:

1. **Docker Desktop.** Download it at [docker.com](https://www.docker.com/products/docker-desktop/). Install it on your computer, open it, and leave it running in the background.
2. **Git client**, to download the GeoLab Dockerfile template. Use the operating system's package manager to install a git client.

Verify Docker and git are installed and working by opening a terminal and running:

```bash
docker --version
git --version
```

If they print a version number, they are installed and working.

In addition to the required software, a **GitHub** account at [github.com](https://github.com), a Docker account, or an AWS account is needed for publishing the image and making it available for GeoLab.

---

(step-1-get-the-template)=
## Step 1: Get the Template

EarthScope provides a starter template. Download it using git to set up a working folder:

```bash
git clone --depth 1 https://github.com/EarthScope/GeoLab.git
cp -R GeoLab/geolab-base my-geolab-image
cd my-geolab-image
```

The `my-geolab-image` folder contains these files:

```text
my-geolab-image/
├── Dockerfile          ← do not edit this
├── environment.yml     ← add your conda packages here
├── requirements.txt    ← add PyPI-only packages here
├── apt.txt             ← add system software here (rarely needed)
├── start               ← do not edit this
├── test_helpers.py     ← Python module with testing functions for packages
└── test_notebook.ipynb ← interactive version of the smoke test
```

> **The only files to edit are `environment.yml`, `requirements.txt`, and `apt.txt`.** Everything else is set up for you.

---

(step-2-add-your-packages)=
## Step 2: Add Your Packages

(custom-image-environmentyml-your-main-package-list)=
### `environment.yml`: Your Main Package List

Add conda Python packages from `conda-forge` here. Open the file and add packages under the `dependencies` section:

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

Conda packages are preferred, because conda checks that everything works together before installing and reduces the possibility of dependency conflicts among packages.

(requirementstxt-packages-only-on-pypi)=
### `requirements.txt`: Packages Only on PyPI

Some packages aren't available through conda-forge and must be installed from PyPI. Add them here, one per line:

```text
earthscope-sdk==1.4.1
seisbench
```

(apttxt-system-software-rarely-needed)=
### `apt.txt`: System Software (Rarely Needed)

Most scientific packages go in `environment.yml`. Only use `apt.txt` for low-level system tools that can't be installed any other way:

```text
build-essential
git
```

---

(step-3-build-and-test-locally)=
## Step 3: Build and Test Locally

Before publishing, build the image on your computer and make sure everything works.

**Build the image:**

```bash
docker build -f Dockerfile --tag my-geolab-image:0.1.0 .
```

This reads the config files and assembles the image. It can take several minutes the first time.

**Run it locally:**

```bash
docker run --rm -p 8888:8888 my-geolab-image:0.1.0
```

Look in the output for a line like:

```text
http://127.0.0.1:8888/lab?token=...
```

Copy that URL into a browser and a JupyterLab session will open.

**Optional: Test that your packages installed correctly:**

In the JupyterLab session, test if a package was installed and functions as expected. A simple version check is often sufficient to determine if a package has been installed and is working.

To test a package in a notebook environment, open a notebook and add this code to a cell:

```python
import importlib
import shutil
import subprocess
import sys

RESULTS = []


def py(modname, alias=None, smoke=None):
    """Import `modname` and optionally run `smoke(mod)` as a sanity check."""
    label = alias or modname
    try:
        mod = importlib.import_module(modname)
        if smoke is not None:
            smoke(mod)
        version = getattr(mod, '__version__', '')
        RESULTS.append((label, 'OK', str(version), ''))
    except Exception as exc:
        RESULTS.append((label, 'FAIL', '', f'{type(exc).__name__}: {exc}'))


def cli(cmd, version_flag='--version'):
    """Verify `cmd` is on $PATH and responds to a version flag."""
    path = shutil.which(cmd)
    if not path:
        RESULTS.append((cmd, 'FAIL', '', 'not on $PATH'))
        return
    try:
        r = subprocess.run([cmd, version_flag],
                           capture_output=True, text=True, timeout=10)
        line = (r.stdout or r.stderr).strip().splitlines()
        version = line[0] if line else 'on PATH'
        RESULTS.append((cmd, 'OK', version[:80], ''))
    except Exception as exc:
        RESULTS.append((cmd, 'OK', 'on PATH', f'{type(exc).__name__}'))


print(f'Python {sys.version}')
print(f'sys.prefix: {sys.prefix}')
```

Open another cell and use either the `py` or `cli` function to test a package. For example:

```python
py('earthscope_sdk', alias='earthscope-sdk')
cli('es') # earthscope-cli entry point
```

Open a new cell and add the following code to summarize the results of the test:

```python
import pandas as pd
from IPython.display import display

df = pd.DataFrame(RESULTS,
                  columns=['package', 'status', 'version', 'error'])

passed = int((df['status'] == 'OK').sum())
total = len(df)
failed = total - passed

print(f'Results: {passed}/{total} OK, {failed} failed')
if failed:
    print('\nFailures:')
    for _, row in df[df['status'] == 'FAIL'].iterrows():
        print(f"  {row['package']:35s}  {row['error']}")

df.style.map(
    lambda v: ('color: red; font-weight: bold' if v == 'FAIL'
               else 'color: green'),
    subset=['status']
)
```

Each package gets a pass or fail which is display in the **Summary** cell of the notebook. If something fails, it usually means a package name is misspelled or a version is unavailable, so go back to `environment.yml` or `requirements.txt` and fix it, then rebuild.

---

(step-4-publish-your-image-to-docker-hub)=
## Step 4: Publish Your Image to Docker Hub

Once the local test passes, rebuild the image for GeoLab's platform and push it to a container registry.

**Rebuild for GeoLab's platform:**

GeoLab uses linux/amd64 images. Depending on your computer operating system (Windows or macOS), you may have to rebuild the image for the linux/amd64 platform. In addition, the image must be published in an image repository that is accessible to GeoLab.

If you created an account using Docker Desktop, pushing an image to Docker Hub (the Docker image repository) does not require additional authentication. `Tag` or name the image with your Docker username and the name of the image.

```bash
docker build --no-cache -f Dockerfile \
  --platform linux/amd64 \
  --tag your-docker-username/my-geolab-image:0.1.0 .
```

> **Why `--platform linux/amd64`?** GeoLab runs on Linux. If you're on a Mac with Apple Silicon, your local machine uses a different architecture. This flag ensures the image works on GeoLab regardless of what you built it on.

**Push the Image to Docker Hub:**

```bash
docker push your-docker-username/my-geolab-image:0.1.0
```

By default, images published to Docker Hub are public and available for use with GeoLab.

(publishing-to-github-or-aws-image-repositories)=
### Publishing to GitHub or AWS Image Repositories

Alternatives to Docker Hub include GitHub Container Registry (ghcr) or AWS Elastic Container Registry (ECR). Choosing an image repository depends on user requirements. GitHub features a tight integration with CI (Continuous Integration) through GitHub Actions that can trigger an image build and push to ghcr. This automates the process of building and pushing an image through a `pull request`. AWS ECR offers cloud-scale uploads and downloads to support multiple instances of GeoLab requested by hundreds of users or more.

Both ghcr and ECR have more stringent authorization practices and controls over publicly available images. For a step-by-step walkthrough for pushing images to either repository, go to [Pushing Images to GitHub or AWS ECR](./pushing_to_ghcr_ecr.md) for detailed instructions.

---

(step-5-launch-it-in-geolab)=
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

(making-changes-later)=
## Making Changes Later

Edit your config files, then rebuild and push with a new version number:

```bash
docker build --no-cache -f Dockerfile \
  --platform linux/amd64 \
  --tag ghcr.io/your-github-username/my-geolab-image:0.1.1 .

docker push ghcr.io/your-github-username/my-geolab-image:0.1.1
```

> **Always use a new version number** (`0.1.1`, `0.1.2`, etc.) when you rebuild. If you reuse the same tag, GeoLab may load the old cached version instead of your new one.

(troubleshooting-package-installation)=
## Troubleshooting Package Installation

In general, it's best practice to install packages using the conda package manager for the GeoLab image. Conda checks packages for dependencies which ensures that dependency conflicts are resolved in the environment. Conda has a search function to discover packages.

Some packages are only available in PyPI and are installed with the pip package manager. Pip also has a search function to find packages in PyPI.

> **Tip:** Keep mind that installation name and import name can be different (such as, scikit-learn vs sklearn).

(find-if-a-conda-package-is-available)=
### Find If a Conda Package Is Available

To find if a package is available through the conda package manager, you can use `conda search`:

```bash
conda search -c conda-forge #PackageName(e.g., seisbench)
```

(find-if-a-package-is-available-on-pypi)=
### Find If a Package Is Available On PyPI

To find if a package is available through the pip package manager, you can use `the following command.

```bash
python -m pip index versions #PackageName(e.g., seisbench)
```

---

(custom-image-quick-reference)=
## Quick Reference

| What you want to do | Where to do it |
| --- | --- |
| Add a Python package | `environment.yml` under `dependencies` |
| Add a PyPI-only package | `requirements.txt` |
| Add a system tool | `apt.txt` |
| Build locally for testing | `docker build --tag my-geolab-image:0.1.0 .` |
| Run locally | `docker run --rm -p 8888:8888 my-geolab-image:0.1.0` |
| Test packages | Run the test cells in a notebook (see Step 3) |
| Build for GeoLab | `docker build --no-cache --platform linux/amd64 --tag ghcr.io/username/image:version .` |
| Publish | `docker push ghcr.io/username/my-geolab-image:0.1.0` |

---

(getting-a-personal-access-token-for-ghcr)=
## Getting a Personal Access Token (for GHCR)

Before you can push images to GHCR, you need a **Personal Access Token (PAT)** with package permissions:

1. Go to **GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)**.
2. Click **Generate new token (classic)**.
3. Give it a name (e.g. `geolab-image`), set an expiration, and check the **`write:packages`** scope.
4. Click **Generate token** and copy it, as you won't be able to see it again.

Save your token somewhere safe (a password manager works well). You'll use it to log in to the registry in Step 4.