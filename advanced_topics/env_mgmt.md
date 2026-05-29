# Environment Management
When you open GeoLab and select an environment when starting your server, you launch into a pre-configured virtual python environment. These environments typically include many commonly used geophysics tools, but we cannot include everything. Additional software and packages can be installed by users in GeoLab, with several options:

1. **Ephemeral Installation:** Due to the ephemeral nature of the JupyterHub servers, packages installed on the server will only be valid for your current session and will not persist from one session to another. You will need to reinstall them each time you log back in to GeoLab, or add them to your notebooks as magic commands. 
2. **Create a Custom Image:** If you need to use the same software repeatedly, during multiple sessions, or for multiple users, you can create your own software environment and load it at launch time. 
3. **Bring Your Own Image:** Many other organizations (e.g, NASA, NOAA) maintain their own JupyterHub compute environment images. Many of these will run in GeoLab. 

## 1. Ephemeral Installation
### Installing python packages
 Use `pip` or `conda` to install the package yourself at the beginning of your session by typing the below in a GeoLab Terminal.
```
pip install pkgname
conda install pkgname
```

Because installations are ephemeral, we recommend adding these commands to your python notebooks using a magic command. This will allow the python notebook to run the bash command, and ensure you re-install the right packages each time you return to your project without having to remember to type it in the terminal.  

You can add a magic command to the beginning of your notebook. The `%pip` / `%conda` forms here are better than their `!pip` / `!conda` counterparts to ensure that the package is installed to the correct directory.

```
%pip install pkgname
```
    
### Installing from a requirements file

If you have a `requirements.txt` or `environment.yml` file (e.g., from a cloned repository), you can install all listed packages at once.

**From a `requirements.txt` file:**
```bash
%pip install -r requirements.txt
```

**From a `environment.yml` file:**
```bash
%conda env update --file environment.yml --name base
```

Note that `--name base` installs packages into the active base environment rather than creating a new one, which is necessary in GeoLab's ephemeral setup. As with individual package installs, these will not persist after your session ends.

We have found that in GeoLab, pip is often faster, while conda's dependency solver is more robust. If you encounter dependency conflict while installing with pip, try conda.

```{{Note}} 
A Note on reproducible conda environments:
When you launch GeoLab, you are already inside a conda environment. If you create a _new_ conda environment, you can maintain the list of packages (ie, in an environment.yml), but because of the ephemeral nature of the  you will have to re-install them all in GeoLab each session. In order to use the new environment to run a notebook, you will need to install the environment to a kernel. For example, to create a kernel called 'Seis', you would need to run the following line of code in the terminal: 

`jupyter ipykernel install --user --name Seis --display-name "Seis" `

and then select the new kernel from the Kernel menu or from the dropdown of available kernels in the upper right of the notebook toolbar.

```

### Installing ubuntu packages
Some software can be installed via the terminal using `apt-get`. Please be advised that users do not have the ability to run commands as super users: `sudo` will not work and these permissions _cannot_ be granted to individual users. If you need to install something foundational, please create a custom image.

### C/C++/Fortran Compilers
Most GeoLab images have `gcc`, `g++`, and `gfortran` compilers installed. You will need to move your makefiles into your home directory and compile them within GeoLab. 

## 2. Create a Custom Image:
```{note} Section Under Development! ``
EarthScope is working to build more comprehensive documentation on custom image creation. In the mean time, please see: 

- [2i2c guide to binderhub](https://docs.2i2c.org/user/environment/dynamic-imagebuilding/#user-environment-building) - This strategy is ideal if you just want to build a reproducible environment that extends the base GeoLab image with additional software.

- If you need to make extensive software changes, you can use our existing image as a template. Instructions are located in the README.md file in the [GeoLab GitHub repository.]({{ geolab_github }}) You will need to clone the repository, modify the build files to suit your needs, build your container and upload it to a public container registry like DockerHub or AWS ECR, and then select it as 'Other' from the dropdown when you [launch your GeoLab server](../getting_started/server_launch.md). This strategy is an advanced approach for software with complex installations, and only recommended for users with some prior knowledge of building docker containers. . 

## 3. Bring Your Own Image
Many other organizations (e.g., NASA, NOAA) maintain JupyterHub images with their software installed. Many of these will run in GeoLab (but we make no guarantees that images built for other platforms will work here!) We recommend this approach for interdisciplinary research, where you may need software installed in another image, but want to run it on data stored in the NSF NGF archive. See the [Server Launch Page](../getting_started/server_launch.md#select-an-environment) for details and select the 'Other' option from the Environment dropdown selection.
 