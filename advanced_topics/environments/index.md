# Managing Environments in GeoLab

When you open GeoLab and select an environment, you launch a pre-configured virtual Python environment that includes commonly used geophysics software. Additional software and packages can be installed by users in GeoLab, with several options:

## [Ephemeral Installation:](#ephemeral-installation)

Environments in JupyterHub are temporary. Packages installed while using GeoLab are only valid during current session and will not persist from one session to another. Packages must be reinstalled every time a new GeoLab instance is launched. For ephemeral installs, add packages using [line magics](https://ipython.readthedocs.io/en/8.24.0/interactive/magics.html#line-magics) commands.

Because installations are temporary, we recommend adding these commands to the beginning of a notebook. This ensures that the required packages are installed and available each time when the notebook runs.

Use the `%pip` / `%conda` magic commands because they execute in the active Python notebook kernel. The `!` magic command executes in the system shell and is best used for running CLI tools or file-system checks.

```ipynb
%pip install pkgname
```

For more information about managing environments, go to [`Managing Environments`](./managing_environments.md) for detailed instructions.

## [Build a Custom Image:](#build-a-custom-image)

If your task or analysis requires the same software over multiple sessions or for multiple users, building a custom image with the required packages is best practice.

The process for building a custom image requires software such as Docker to build images. GeoLab can use custom images available on public image repositories such as Docker Hub, GitHub Container Repository (ghcr), or AWS Elastic Container Repository (ECR).

For more information about build a custom image, go to [`Building Custom Images`](./building_custom_images.md) for detailed instructions for building and pushing images to an image repository.

## [Run an Image From a GitHub Code Repository With Binder](#run-an-image-from-a-github-code-repository-with-binder)

GeoLab can run environments from GitHub repository using [Binder](https://mybinder.readthedocs.io/en/latest/). The process is similar to building a custom image but instead of building and storing the image with Docker, you provide the URL of the GitHub repository to GeoLab and the image will be dynamically built and opened.

While this is the simplest method for building and running a custom image, GeoLab builds the image every time. The build process can take some time and is not ideal for multiple users. An image in a repository is pulled directly into JupyterHub and started without waiting for it to be built.

For more information about using Binder, go to [`Binder for Images`](./binder_for_images.md) for detailed instructions for building a repository that works with Binder.

## [Bring a JupyterHub Image](#bring-a-jupyterhub-image)

Other organizations, such as NASA or NOAA maintain their own JupyterHub compute environment images that run in GeoLab. Using an image from another organization uses the same process as a custom image. Select `Other...` in the Environment pull down menu, enter the image URL, select the `Resource Allocation`, and select `Start`.
