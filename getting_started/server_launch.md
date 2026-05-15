# Launching your Server
GeoLab offers several cloud server configurations. Use the menu options on this page to select your environment and resource configurations.

![image](../img/server_options.png)

## Select an Environment
Several pre-configured compute environments are available to choose from. Use the dropdowns below to select your initial configuration. Please see [Envrionment Management](../advanced_topics/env_mgmt.md) for more details on customizing your ephemeral server instance after launch, or bringing your own custom image.

```{dropdown} GeoLab
The default GeoLab image contains a variety of python packages to enable a broad range of geophyiscal data analysis.
This image inherits all software in [pangeo/pangeo_notebook](https://github.com/pangeo-data/pangeo-docker-images/tree/d0ea2af694e5f6a27c04abd046620fcf5bebf126/pangeo-notebook), while including additional support for the EarthScope SDK/CLI and other geophysics tools. For a complete list of packages included in this image, see the {{ geolab_github }}

```

```{dropdown} SeaFloor Geodesy
This image is specifically intended for users to process GNSS-A/Seafloor Geodesy data.  It is built on Ubuntu 20.04, and contains a set of software packages to support translation and processing of GNSS-A data collected on SV3 wavegliders.  These packages include PRIDE-PPPAR for kinematic positioning of the waveglider and GARPOS for computing seafloor positions.  It also contains an EarthScope python package es-sfgtools under development to streamline data and metadata access and drive the workflows needed to process these data.  For more information, see the es-sfgtools documentation [here.](https://es-sfgtools.readthedocs.io/en/latest/index.html)
```

```{dropdown} Jupyter
The [Jupyter SciPy](https://jupyter-docker-stacks.readthedocs.io/en/latest/using/selecting.html#jupyter-scipy-notebook) image is a minimalist default configuration available to all JupyterHubs. This Environment does not have geophysics-specific tools and offers limited scientific computing tools. This environment is appropriate for very introductory python learning modules, or as a 'clean' base to build from. 
```

```{dropdown} R Studio
Access to GeoLab in R is available using a [rocker geospatial image](https://rocker-project.org/images/versioned/binder.html). However, EarthScope does not currently provide support or geophysical package extensions for this image.
```

```{dropdown} Short Course Images
EarthScope supports several educational short courses and workshops hosted in GeoLab throughout the year. If you are using GeoLab as part of one of these workshops, select the image corresponding to your course.

Short Course Images are available on a temporary basis only and contain software configurations specific to the course. These environments are not intended for widespread public use and are not recommended for ongoing research or external projects. They may be modified, updated, or deleted without notice.

In some cases, short courses may use a custom image that is not available in the dropdown. See Other - Bring Your Own Image, below.

You can learn more about EarthScope's educational short course offerings [here.]({{ course_offerings }})
```

```{dropdown} Other...

GeoLab is compatible with many other custom compute environments that are configured to run in JupyterHub.

Custom images must be built docker containers that are available in a public image repository (e.g., aws ECR, ACR, dockerhub, quay.io)

Select 'Other' from the dropdown menu and specify the public URL of the container.

See [Environment Management](../advanced_topics/env_mgmt.md) for more details on bringing your own custom image.
```

```{dropdown} Build Your Own Image

GeoLab has a mybinder extension that allows you to specify a custom environment configuration in a github repository and build it dynamically. 

See [Environment Management](../advanced_topics/env_mgmt.md) for more instructions on building your own binder environment.
```


## Select your Resource Allocation
The Server Options page lets you allocate RAM and CPU tailored to your needs, ensuring you have the right resources for your projects.

When selecting a server for your JupyterHub environment, it is important to choose the minimum server instance size that meets your computational needs without overcommitting resources. In order to keep our operational costs low, we recommend that you start with the smallest server option available, and only move to a larger instance when you encounter performance barriers. See [User Resource Limits and Cost Controls](#user-resource-limits-and-cost-controls) for more information.

```{note}
 In many cases, you will be utilizing resources on a shared server to optimize cloud costs. You will be allocated, at minimum, the amount of RAM and CPU shown in black text. Depending on the workloads of other simultaneous users on the same server, additional CPU resources may be allocated to you automatically (up to the amount shown in gray text) and you may experience temporary boosts in performance. 
```

Larger servers for more intensive projects, as well as servers offering GPU resources, are available on a case-by-case basis. To request access to a larger server for your project, please email help@earthscope.org or fill out the {{ resource_request_form }} 


## User Resource Limits and Cost Controls
In order to be able to offer GeoLab to the geophysics research community at no cost, utilization guardrails are in place to prevent excessive or inappropriate use of these grant funds. A system is in place which allocates each user a quota limit for resource utilization credits which are refreshed on a rolling basis. In order to conserve your quota, we recommend launching the smallest server viable for your task, and stopping your server when you are done. See [Quotas](quotas.md) for more details.

### Stopping Your Server and Logging Out
For improved cost and operational efficiency, and to preserve your compute quotas, it is important to stop your server when you are done using GeoLab to release the resources that you have claimed during your session.

__Please _Stop Your Server_ before logging off.__ Logging out without first stopping your server can cause unexpected errors when re-launching GeoLab. If you experience repeated errors, try clearing browser caches/cookies or launching GeoLab in another web browser. 

1. Stop your server: 

 - Navigate to `File` --> `Hub Control Panel` (this will open in a new browser tab).
 - In the new tab, push the big red button that says 'Stop My Server.'
 - Once the button disappears, your server instance has been stopped.
 - If you need to re-launch GeoLab (i.e., with different resource allocations or environment selections), click the  'Start My Server' button to return to the Server Launch page without proceeding to steps 2 and 3. 

2. Once your server is stopped, you can Log Out by clicking `Log Out` in the upper right corner of the Server Shut Down Page, or File -> Log Out. 

3. Finally, close all browser tabs for GeoLab before logging in again.

```{note}
Servers will automatically stop after 60 minutes of user inactivity. Long-running compute processes will time out without hourly user interaction. 
```