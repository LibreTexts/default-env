# LibreTexts JupyterHub Default Environment

This repository contains the Docker files needed to build the default Jupyter
environment available at jupyter.libretexts.org. The goal of this environment
is to contain as many of the most popular software packages needed for
scientific computing from a variety of languages: Python, R, Octave, Sage,
Julia, etc. These packages will be updated to the latest compatible versions on
a quarterly basis.

At the moment, this repository contains two Jupyter environments.

* **Minimum Default** contains all requested packages,
with only a Python environment.
* **Rich Default** contains all the requested packages,
with the Python, R, Julia, Octave, and Sage environments.

**repo2docker** contains our attempt to use repo2docker to generate
a Docker image. It failed to install packages in the root environment.
Kept as a reference for now.

# Requesting additional installed packages

In general, we will install any user requested software available in the
[Ubuntu 18.04 LTS apt repositories](https://packages.ubuntu.com/bionic/>) or
from the Conda [conda-forge channel](https://conda-forge.org/feedstocks/>) The
conda installed packages, in general, will be the default programs accessible
at the JupyterLab command line. The latest *compatible* versions of the
requested packages will be installed. To request packages you can:

1. make a pull request to this repository adding your desired packages to the
   [Dockerfile](https://github.com/LibreTexts/default-env/blob/master/rich-default/Dockerfile>),
2. open an issue on this repository with the desired packages, or
3. send an email to jupyterteam@ucdavis.edu with a list of the desired packages.

If your desired package is not available in the default apt repositories or the
Conda Forge Conda channel, you have these options for supporting your software
needs:

1. [submit the package for inclusion in Conda Forge channel](https://conda-forge.org/#contribute),
2. install the package manually after launching the default environment (you
   must ensure the package is installed in the user directory for persistence
   among sessions), or
3. create your own [repo2docker compatible
   environment](https://repo2docker.readthedocs.io/en/latest/config_files.html)
   and send us the URL of the Git repository with your configuration files.

Note that we will not install "un-released" software from arbitrary locations
in the default environment, but you may do so using options 2 and 3 above. We
will consider installing packages from PyPi, CRAN, and other language specific
package managers on a case-by-case basis for temporary use until it is added to
Conda Forge.

## Creating your own custom conda environment

### Configuring `nb_conda_kernels`
Based on the [documentation](https://zero-to-jupyterhub.readthedocs.io/en/latest/user-environment.html?highlight=conda%20environments#allow-users-to-create-their-own-conda-environments-for-notebooks)
from *Zero to JupyterHub with Kubernetes*.

1. Make sure that `nb_conda_kernels` is installed in the root conda
environment. Note that your version numbers may differ.
   ```
   $ conda list -n base | grep "nb_conda_kernels"
   nb_conda_kernels          2.2.2                    py36_0    conda-forge
   ```

1. In your home directory, run `conda config`. This will create a file called
`.condarc` in your home directory. We want the following to be inside that file:
   ```
   envs_dirs:
     - /home/jovyan/my-conda-envs/
   ```

   *Alternately*, you can do this the fast way by running the following:
   ```
   conda config
   echo "envs_dirs:" > ~/.condarc 
   echo "  - /home/jovyan/my-conda-envs/" >> ~/.condarc
   ```

### Creating your own conda environment

1. Open the Terminal from the Launcher and run the command
to create a conda environment.
   ```
   conda create -n <your_env_name> <kernel> <packages>
   ```

   For example, to create a conda environment for Python, run
`conda create -n my_python_env ipykernel`.
   If you want the `pendulum` package with Python, then run
`conda create -n my_python_env ipykernel pendulum`.

   All conda environments will be saved in a folder in
your home directory called `my-conda-envs`.

1. Activate your conda environment by running 
`source activate <your_env_name>`.

   You can install additional packages while your environment is
activated.

1. Deactivate your current conda environment by running
`deactivate` or `source deactivate`.

Your Launcher should show another Notebook labeled with your
conda environment. If you create a Notebook with this label,
the Notebook can import the packages that you installed before.

### Tips:
* You can see a list of your conda environments by running
`conda env list`.
* To add more packages to your conda environment, run
`conda install -p </path/to/env> <package>`.
