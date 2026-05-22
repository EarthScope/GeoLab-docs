# Environment Management
GeoLab images include many commonly used geophysics tools, but we cannot include everything. Additional software and packages can be installed by users in GeoLab, with two options:
1. Ephemeral Installation: Due to the ephemeral nature of the JupyterHub servers, packages installed on the server will only be valid for your current session and will not persist from one session to another. You will need to reinstall them each time you log back in to GeoLab, or add them to your notebooks as magic commands.
2. Use a Custom Image: If you need to use the same software repeatedly, during multiple sessions, or for multiple users, you can create your own software environment and load it at launch time.  

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

### Installing ubuntu packages
Some software can be installed via the terminal using `apt-get`. Please be advised that users do not have the ability to run commands as super users: `sudo` will not work and these permissions _cannot_ be granted to individual users. If you need to install something foundational, please create a custom image.

### C/C++/Fortran Compilers
Most GeoLab images have `gcc`, `g++`, and `gfortran` compilers installed. You will need to move your makefiles into your home directory and compile them within GeoLab. 

## 2. Create a Custom Image:
```{note} Section Under Development!
EarthScope is working to build more comprehensive documentation on custom image creation. In the mean time, please see:

- [2i2c guide to binderhub](https://docs.2i2c.org/user/environment/dynamic-imagebuilding/#user-environment-building) - This strategy is ideal if you want to build a reproducible environment that extends the base GeoLab image with additional software.

- [2i2c guide on building your own image from a template.](https://docs.2i2c.org/admin/howto/environment/update-community-image/). - This strategy is a last-resort to add software with complex installations and recommended for advanced users with some prior knowledge of Docker. We recommend starting with an image from [Pangeo.](https://github.com/pangeo-data/pangeo-docker-images). 
```


