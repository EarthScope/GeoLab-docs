# Choosing Environments in GeoLab

When you open GeoLab and select an environment, you launch a pre-configured virtual Python environment that includes commonly used geophysics software. You can install additional software and packages in GeoLab, using one of several options: emphemeral images, building a custom image, running an image in Binder, or using a pre-built third party image.

(how-to-choose)=
## How to choose

Which environment to use depends on whether you want the environment to persist across session, installing software not available in the defautl environment, mutability of the environment, a reproducible environment, or the convenience of using a prebuilt environment.

- **[Ephemeral Installation](./creating_temporary_environments.md)** installs packages directly in a running notebook with `%pip`/`%conda`. It's the fastest way to get a package, but the install doesn't survive a new session, so it needs to be re-run each time.
- **[Build a Custom Image](./building_custom_images.md)** uses Docker to package your environment into an image you push to a repository (Docker Hub, GHCR, or ECR). GeoLab launches that exact image every time, so it's the most reproducible option, at the cost of needing Docker and a repository account.
- **[Run an Image From a GitHub Code Repository With Binder](./binder_for_images.md)** skips Docker and a repository: GeoLab reads config files from a GitHub repo and builds the image for you. It's easier to set up and share than a custom image, but GeoLab rebuilds it from scratch on every launch. Changes to the GitHub repository allows you to change the environment with having to build and host an image.
- **[Bring a JupyterHub Image](#bring-a-jupyterhub-image)** reuses a third-party image someone else already built and published, such as one maintained by NASA or NOAA. It works the same way as launching a custom image — you just didn't build the image yourself.

| Method | Persists across sessions? | What it requires | When to use it |
| --- | --- | --- | --- |
| [Ephemeral Installation](./creating_temporary_environments.md) | No — reinstalled every session | Nothing; a `%pip`/`%conda` line in a notebook | Installing software not in the default environment for a one-off task, quick test, or fix |
| [Build a Custom Image](./building_custom_images.md) | Yes — the same image every launch | Docker installed locally, and an account on an image repository (Docker Hub, GHCR, or ECR) | A reproducible, stable environment for repeated analyses or a team sharing a fixed environment |
| [Run an Image From a GitHub Code Repository With Binder](./binder_for_images.md) | Yes — same GitHub repo every launch, rebuilt each time | A GitHub repo with config files; no local Docker or registry | A mutable environment you can change by editing the repo, without building or hosting an image |
| Bring a JupyterHub Image | Yes — depends on the external image | Only the image URL | The convenience of reusing a prebuilt third-party image, e.g. from NASA or NOAA |