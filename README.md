# LibreTexts JupyterHub Default Environment

This repository contains the repo2docker config files needed to build the default Jupyter
environment available at jupyter.libretexts.org. The goal of this environment
is to contain as many of the most popular software packages needed for
scientific computing from a variety of languages: Python, R, Octave, Sage,
Julia, etc.

# How to Build

1. Be sure to install repo2docker from source using `pip install git+https://github.com/jupyterhub/repo2docker.git`
2. Update the docker image which repo2docker builds on top of with `docker pull buildpack-deps:bionic`
3. Run `repo2docker --user-name jovyan --user-id 1000 .` in this directory to start the build. We ensure that we do not break the filesystem by specifying `jovyan` as the username, `user-id 1000` specifies a normal user account (not root), and `.` starts the build in the current directory. You may also run `repo2docker --user-name jovyan --user-id 1000 https://github.com/LibreTexts/default-env` to build an image from master.
4. Once finished, you will have an image that looks like `r2d-2e1598919088:latest`. Tag the image with your desired tag (`2.x` in this case) using `docker tag r2d-2e1598919088:latest libretexts/default-env:2.x` and push to Docker Hub using `docker push libretexts/default-env:2.x`.

# Legacy environment

Before our current setup, there has been a legacy default enviornment used before Aug. 11th, 2020. The `Dockerfile` for this image is available in the `legacy-rich-default` branch. This is kept for reference purposes.

# Requesting additional installed packages

In general, we will install any user requested software available in the
[Ubuntu 18.04 LTS apt repositories](https://packages.ubuntu.com/bionic/) or
from the [conda-forge](https://conda-forge.org/feedstocks/) Conda channel. The
conda installed packages, in general, will be the default programs accessible
at the JupyterLab command line. The latest *compatible* versions of the
requested packages will be installed. To request packages you can:

1. Make a pull request to this repository, adding your desired packages to the `environment.yml` or `apt.txt`,
2. Open an issue on this repository with the desired packages, or
3. Send an email to jupyterteam@ucdavis.edu with a list of the desired packages.

If your desired package is not available in the default apt repositories or the
Conda Forge Conda channel, you have these options for supporting your software
needs:

1. [Submit the package for inclusion in Conda Forge channel](https://conda-forge.org/#contribute),
2. Install the package manually after launching the default environment (see [our FAQ page](https://jupyter.libretexts.org/hub/faq#how-can-i-install-custom-packages) for instructions on how to install custom packages), or
3. create your own [repo2docker compatible
   environment](https://repo2docker.readthedocs.io/en/latest/config_files.html)
   and send us the URL of the Git repository with your configuration files.

Note that we will not install "un-released" software from arbitrary locations
in the default environment, but you may do so using options 2 and 3 above. We
will consider installing packages from PyPi, CRAN, and other language specific
package managers on a case-by-case basis for temporary use until it is added to
Conda Forge.
