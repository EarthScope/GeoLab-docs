# GeoLab Binder

There are two ways to launch a custom environment in GeoLab. The first is to [Build a custom image](custom_image_page.md) and push it to a registry, and specify the image URL when launching GeoLab. The second, called **GeoLab Binder**, is to point GeoLab at a GitHub repository and let it build the environment for you. This guide covers how the second approach works and when to use it.

---

## What Is GeoLab Binder?

GeoLab Binder lets you launch a session from a GitHub repository instead of a prebuilt image. You put configuration files in your repo that describe what packages you need, and GeoLab builds the environment automatically when you start a session.

Behind the scenes, GeoLab reads the Dockerfile and configuration files in your repository and builds the container image for you. You write the Dockerfile (using the same template as a custom image), but you never run Docker locally or push to a registry, as GeoLab handles the build.

Both approaches run inside GeoLab, so you have the same access to EarthScope data services either way.

1. GitHub repo with config files
2. GeoLab builds image
3. Session launches in GeoLab

---

## GeoLab Binder vs. Custom Image

Both approaches use GeoLab's environment selector, but they use different options. The difference is what you select and what you paste.

| | **GeoLab Binder** | **Custom Image** |
| --- | --- | --- |
| Menu option | **Build your own image** | **Other** |
| What you paste | A GitHub repo URL | A container registry URL (`ghcr.io/...`) |
| Who builds the image | GeoLab, automatically from your repo | You, locally with Docker |
| What you need | Dockerfile + config files in a GitHub repo | Docker Desktop, local build, registry push |
| EarthScope data access | Yes | Yes |
| How updates work | Push a commit, and the next launch picks up the changes | Rebuild and push a new image tag |
| Best for | Getting started quickly, iterating, sharing a repo | Stable, versioned environments; full Docker control |

**Use GeoLab Binder when** you want to share a GitHub repository as a runnable environment without running Docker locally or managing a container registry.

**Use a custom image when** you need a fixed, versioned environment that won't change between sessions, or when you need more control than config files allow. See [Building a Custom GeoLab Image](./building_custom_images.md).

---

## Before You Start

You need:

- A **GitHub account** and a **GitHub repository** for your project.
- A GeoLab account at [GeoLab](https://geolab.earthscope.cloud).

No local Docker builds or container registry required.

---

## Step 1: Set Up Your GitHub Repository

Create a new repository on GitHub (or use an existing one). Add your notebooks and at least one configuration file:

```text
my-project/
├── Dockerfile            ← required (copy from the geolab-base template)
├── environment.yml       ← your packages (conda)
├── requirements.txt      ← PyPI-only packages (optional)
├── apt.txt               ← system software (optional, rarely needed)
├── postBuild             ← setup commands to run after install (optional)
└── my_notebook.ipynb     ← your notebooks
```

Copy the `Dockerfile` from the `geolab-base` template at [EarthScope/GeoLab](https://github.com/EarthScope/GeoLab/tree/main/geolab-base) and leave it unchanged. All your customization goes in the other files. GeoLab Binder reads everything in the repo and builds the image when you launch.

---

## Step 2: Write Your Configuration Files

### `environment.yml`: Your Main Package List

```yaml
name: my-binder-env
channels:
  - conda-forge
dependencies:
  - python=3.11
  - numpy
  - matplotlib
  - obspy
```

### `requirements.txt`: PyPI-only Packages (If Needed)

```text
some-pypi-package==1.2.3
```

### `apt.txt`: System Software (If Needed)

```text
build-essential
```

### `postBuild`: One-time Setup Commands (Optional)

If you need to run something after packages install, such as downloading a data file or enabling a Jupyter extension, create a `postBuild` file:

```bash
#!/bin/bash
set -e
jupyter labextension install my-extension
```

Before committing, mark it executable:

```bash
chmod +x postBuild
git add postBuild
git commit -m "add postBuild script"
```

These files use the exact same format as the custom image template. If you've worked through [Building a Custom GeoLab Image](./building_custom_images.md), you can reuse them directly.

Once your files are ready, commit and push them to GitHub:

```bash
git add environment.yml requirements.txt apt.txt
git commit -m "add environment configuration"
git push
```

If this is a new repository, you may need to set the upstream branch on your first push:

```bash
git push -u origin main
```

---

## Step 3: Launch from Your Repo in GeoLab

1. Go to [earthscope.org/data/geolab](https://www.earthscope.org/data/geolab/) and click **Launch GeoLab**.
2. Enter your username and password, then click **Continue**.
3. In the environment selector, choose **Build your own image**.
4. In the **Repository** field, paste the git clone URL for your repository:

   ```text
   https://github.com/your-github-username/my-project.git
   ```

5. Click **Build Image**. GeoLab will build the environment from your repository. This can take several minutes the first time.
6. When you receive a notification that the build is complete, click **Start**.

> **If the build fails:** Check the build log for error messages. Common causes are a misspelled package name, an unavailable package version, or a syntax error in `environment.yml`. Fix the file, push the commit, and try again.

---

## Step 4: Share Your Environment

To share your environment with someone else, give them the git clone URL for your repository. They follow the same steps: go to GeoLab, choose **Build your own image**, paste the URL, and launch.

Because the environment is defined by files in the repo, anyone who uses the same URL gets the same packages. If you update `environment.yml` and push, the next person to launch picks up the new version automatically.

---

## Updating Your Environment

Edit your configuration files, commit, and push. The next GeoLab launch from that repo URL will rebuild with the updated packages.

```bash
# edit environment.yml, then:
git add environment.yml
git commit -m "add pandas to environment"
git push
```

> **Note:** A new commit triggers a fresh build on the next launch. If you're changing packages frequently, expect slower first launches while GeoLab rebuilds the image.

---

## Quick Reference

| What you want to do | How |
| --- | --- |
| Define your packages | `environment.yml` (conda), `requirements.txt` (PyPI) |
| Launch from a repo | GeoLab → Start Server → Build your own image → paste git clone URL |
| Share your environment | Share the git clone URL (`https://github.com/username/repo.git`) |
| Update your environment | Edit config files, commit, and push |
| Use a fixed versioned environment instead | See [Building a Custom GeoLab Image](./building_custom_images.md) |