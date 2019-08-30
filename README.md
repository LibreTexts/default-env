# default-env

At the moment, this contains two default Jupyter environments.

* **Minimum Default** contains all requested packages,
with only a Python environment.
* **Rich Default** contains all the requested packages,
with the Python, R, Julia, Octave, and Sage environments.

**repo2docker** contains our attempt to use repo2docker to generate
a Docker image. It failed to install packages in the root environment.
Kept as a reference for now.

## Creating your own custom conda environment
### Configuring `nb_conda_kernels`
Based on the [documentation](https://zero-to-jupyterhub.readthedocs.io/en/latest/user-environment.html?highlight=conda%20environments#allow-users-to-create-their-own-conda-environments-for-notebooks)
from *Zero to JupyterHub with Kubernetes*.

Make sure that `nb_conda_kernels` is installed in the root conda
environment.
```
$ conda list -n base | grep "nb_conda_kernels"
nb_conda_kernels          2.2.2                    py36_0    conda-forge
```
Your version numbers may differ.

In your home directory, run `conda config`.
Using a text editor like `nano`, replace the contents of `~.condarc`
with the following:
```
envs_dirs:
  - /home/jovyan/my-conda-envs/
```

### Creating your own conda environment

Open the Terminal from the Launcher and run the command
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

Activate your conda environment by running 
`source activate <your_env_name>`.

You can install additional packages while your environment is
activated.

Deactivate your current conda environment by running
`deactivate` or `source deactivate`.

Your Launcher should show another Notebook labeled with your
conda environment. If you create a Notebook with this label,
the Notebook can import the packages that you installed before.

### Tips:
* You can see a list of your conda environments by running
`conda env list`.
* To add more packages to your conda environment, run
`conda install -p </path/to/env> <package>`.
