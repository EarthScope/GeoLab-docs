# Launching your Server
GeoLab offers several cloud server configurations. Use the menu options on this page to select your environment and resource allocation.

![image](../img/server_options.png)

## Select an Environment
Several pre-configured compute environments are available from the dropdown menu. Please see [Environment Management](../advanced_topics/env_mgmt.md) for details on customizing your environment after launch or bringing your own image.

```{dropdown} GeoLab
The default GeoLab image contains a variety of Python packages for a broad range of geophysical data analysis. It inherits all software in [pangeo/base_image](https://github.com/pangeo-data/pangeo-docker-images/base_image), with additional support for the EarthScope SDK/CLI and other geophysics tools. For a more thorough description of the image, see the GeoLab github {{ geolab_github }}.
```

```{dropdown} SeaFloor Geodesy
This image is intended for processing GNSS-A/Seafloor Geodesy data. It is built on Ubuntu 20.04 and includes PRIDE-PPPAR, GARPOS, and the EarthScope `es-sfgtools` package. For more information, see the [es-sfgtools documentation](https://es-sfgtools.readthedocs.io/en/latest/index.html).
```

```{dropdown} Jupyter
The [Jupyter SciPy](https://jupyter-docker-stacks.readthedocs.io/en/latest/using/selecting.html#jupyter-scipy-notebook) image is a minimalist configuration available to all JupyterHubs, without geophysics-specific tools. Appropriate for introductory Python learning or as a clean base to build from.
```

```{dropdown} R Studio
GeoLab in R is available via a [rocker geospatial image](https://rocker-project.org/images/versioned/binder.html). EarthScope does not currently provide support or geophysical package extensions for this image.
```

```{dropdown} Technical Short Course Images
The NSF NGF supports educational short courses hosted in GeoLab throughout the year. If you are attending one of these workshops, select the image for your course. Technical Short Course environments are temporary and may be modified or removed without notice. You can learn more about the NSF NGF's technical short course offerings [here]({{ course_offerings }}).
```

```{dropdown} Other...
GeoLab is compatible with custom environments configured to run in JupyterHub. Custom images must be publicly available Docker containers (e.g., on AWS ECR, DockerHub, or quay.io). Select **Other** from the dropdown and paste the public URL of your container. See [Environment Management](../advanced_topics/env_mgmt.md) for details.
```

```{dropdown} Build Your Own Image
GeoLab includes a mybinder extension for building a custom environment dynamically from a GitHub repository. See [Environment Management](../advanced_topics/env_mgmt.md) for instructions.
```

## Select your Resource Allocation
Start with the smallest server that meets your needs — larger instances consume your [resource quota](./quotas.md) more quickly.

```{note}
On shared servers, you are guaranteed at minimum the RAM and CPU shown in black text. Additional CPU may be allocated automatically when other users are idle (shown in gray text).
```

To request a larger server or GPU access, email help@earthscope.org or fill out the {{ resource_request_form }}.

## Stopping Your Server and Logging Out

**Always stop your server before closing your browser tab or logging out.** Logging out without stopping your server first can cause errors on your next login. And, running servers will deplete your compute quotas whether the browser tab is open or not. 

1. Go to **File → Hub Control Panel** and click **Stop My Server**.
2. Once the button disappears, click **Log Out** in the upper right corner.
3. Close all GeoLab browser tabs.

```{note}
Servers stop automatically after 60 minutes of inactivity. Long-running processes will time out unless you interact with the hub hourly.
```
