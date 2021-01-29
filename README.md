# LibreTexts JupyterHub Default Environment
[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/LibreTexts/default-env/HEAD)

This repository contains the repo2docker config files needed to build the default Jupyter
environment available at jupyter.libretexts.org. The goal of this environment
is to contain as many of the most popular software packages needed for
scientific computing from a variety of languages: Python, R, Octave, Sage,
Julia, etc.

## Repo2docker Configuration Files
- [environment.yml](https://github.com/LibreTexts/default-env/blob/master/environment.yml). This file contains all the packages to be installed by conda. It is generally used for R, Python and C/C++ packages. We only install packages available from the conda-forge and default channels here.
- [apt.txt](https://github.com/LibreTexts/default-env/blob/master/apt.txt). Here we install Debian packages through `apt-get` which otherwise are not available through conda, such as texlive and some base functionality tools. We will only install from Ubuntu 18.04 APT repositories. 
- [Project.toml](https://github.com/LibreTexts/default-env/blob/master/Project.toml). This is the file where repo2docker installs Julia packages.
- [postBuild](https://github.com/LibreTexts/default-env/blob/master/postBuild). This is a shell script which runs after everything else has been built and installed. We enable JupyterLab extensions here. 

You can read more about each configuration file's purpose from the [repo2docker documentation](https://repo2docker.readthedocs.io/en/latest/config_files.html).

## How to Build a New Image

### Step 1: Pre-Setup

1. If you plan to build the image using binder.libretexts.org, you simply need to clone this repository with `git clone https://github.com/LibreTexts/default-env.git` in order to make the changes you need. If you are part of JupyterTeam, it is recommended that you do this as it is much faster and easier.

If you plan to build the image locally, follow these next steps;
1. Install repo2docker from source using `pip install git+https://github.com/jupyterhub/repo2docker.git`. The stable releases on PyPI are often out of date and the developers themselves have [suggested to install from source](https://github.com/jupyterhub/repo2docker/pull/855). 
2. [Install and enable Docker](https://docs.docker.com/get-docker/) if you have not already. Repo2docker requires Docker to be running in order to function.
2. Update the Docker image which repo2docker builds on top of with `docker pull buildpack-deps:bionic`.
3. Clone the `default-env` repository with `git clone https://github.com/LibreTexts/default-env.git`. You may want do this on one of the test servers we have available so that you can take advantage of *FAST* internet speeds for image builds and pushes.

### Step 2: Making Changes

From now on, follow the steps whether you are building locally or with binder.libretexts.org unless it is otherwise specified.
1. Before beginning, it is usually good practice to create a new git branch and make your changes in there. If you are only doing something minor, and have someone else available to immediately review, you may find it easier to make the changes directly in the master branch. Use your best judgement and ask if uncertain. 
2. Using nano, vim, or another text-editor of your choice, make changes to add/remove/alter packages within the relevant [repo2docker configuration file(s)](#repo2docker-configuration-files). For the most part you should be able to follow the intallation pattern already present within the file. For instance, all packages installed via conda (such as those for Python and R) will go into `environment.yml` under the `dependencies: ` section with a format of `  -<package-name>=<version-number`. When adding packages, be sure to add a comment in the file about what the package is and why it was included.
3. If the package requires a `jupyter labextension`, then be sure to place it in `postBuild`. Don't forget to add `--no-build` at the end of the command or else JupyterHub will try to rebuild itself after each `jupyter labextension` command. 

### Step 3: Building, Tagging and Pushing a Test Image

1. If you are building using binder.libretexts.org, you will want to use the `docker-retag` script found in metalc-configurations. There is a markdown (`.md`) file that explains how to use it. You will need to login to the JupyterTeam dockerhub online to find the repository that begins with `libretexts/binder-dev-libretexts`. You will find your image uploaded here after it has been built and updated by binder.libretexts.org.

Follow the below steps if you are trying to build the image locally;
1. After your changes have been made, run `repo2docker --user-name jovyan --user-id 1000 .` in the `default-env/` directory to start the build. We ensure that we do not break the filesystem by specifying `jovyan` as the username, `--user-id 1000` specifies a normal user account (not root), and `.` starts the build in the current working directory.  
2. Once finished, you will have an image that looks like `r2d-<string>:latest`, where `<string>` could look something like `2e1598919088`. If you do not see this, look for it with `docker images`. You will want to tag this image with `docker tag r2d-<string>:latest libretexts/default-test:<tagname>` so that you can later push this to our testing dockerhub repository . Be sure to fill in the `<string>` and `<tagname>` fields with the actual string value given by repo2docker and the tagname that you desire. An example testing tagname would be something like `pandas-test`.
3. After the image has been tagged, push it to the testing dockerhub repository wth `docker push libretexts/default-test:<tagname>`. Use the Libretexts dockerhub login credentials when pushing. 

### Step 4: Testing the Image

Before deploying the image to the production environment at jupyter.libretexts.org, you will want to test the image on staging.jupyter.libretexts.org.
1. ssh into a management node on the cluster and edit the helm values for staging.jupyter.libretexts.org located at `~/galaxy-control-repo/kubernetes/staging-jhub/staging-config.yaml`. Look for the following lines and update the `tag: <tagname>` value with your chosen tagname;
```
image:
  name: libretexts/default-test
  tag: "<tagname>"
```
2. In order to apply your helm value changes, you will need to upgrade the helm chart for `staging-jhub` with these new values. Navigate to `~/galaxy-control-repo/kubernetes/staging-jhub/` and use the command `./upgrade.sh` to run the upgrade shell script.
3. After the upgrade is complete, you can go to staging.jupyter.libretexts.org, spawn the environment selected by default, and you should now be able to test your image in JupyterLab.

### Step 5: Deploying to Production

If you find that the test on staging was successful, you can now deploy the image on production. 
1. Retag the image with your desired tag (`2.x` in this case) using `docker tag libretexts/default-test:<tagname> libretexts/default-env:2.x`. Starting with our images built using repo2docker, we are using `2.x` (where x is some number) notation to denote the version number.  
2. Push this image to the default-env dockerhub repository using `docker push libretexts/default-env:2.x`.
3. ssh into a management node on the cluster and edit the helm values for jupyter.libretexts.org located at `~/galaxy-control-repo/kubernetes/jhub/config.yaml`. Look for a similar `tag: 2.x` line as you did in [Step 4](#step-4-testing-the-image), and change it to your new `2.x` value. 
```
image:
  name: libretexts/default-env
  tag: "2.x"
```
4. Before upgrading, check if there are any users of the Hub with `kubectl get pods -n jhub` before running the upgrade script, as it could cause problems for active users. Get confirmation from another team member to be entirely certain of this before upgrading. 
5. Within the `~/galaxy-control-repo/kubernetes/jhub/` directory, run `./upgrade.sh` to apply the helm values for `jhub`, similar to how you did before on `staging-jhub`. 
6. Once the upgrade is complete, your new image will be deployed and available at `jupyter.libretexts.org`. Spawn the environment selected by default to ensure it deployed properly. Great work!

### Step 6: Committing to Github

1. If you made your changes within a branch other than master, be sure to commit your changes made within `~/jupyterhub/config.yaml` and `~/jupyterhub-staging/staging-config.yaml` and then push and make a pull request on the [default-env repository](https://github.com/LibreTexts/default-env). Otherwise, simply push straight to the master branch. 
2. Tag the image by now running [`git tag 2.x`](https://www.freecodecamp.org/news/git-tag-explained-how-to-add-remove/). Tags work similar to branches and you will tag whichever commit you are currently located at. Push the tag with `git push origin 2.x`. If you need to update a tag, use the `-f` parameter like `git tag -f 2.x`.

## Legacy environment

Before our current setup, there has been a legacy default enviornment used before Aug. 11th, 2020. The `Dockerfile` for this image is available in the `legacy-rich-default` branch. This is kept for reference purposes.

## Requesting additional installed packages

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
