# LibreTexts JupyterHub Default Environment
[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/LibreTexts/default-env/HEAD)

This repository contains the repo2docker config files needed to build the default Jupyter environment available at jupyter.libretexts.org, as well as serve libretexts.org directly via our CKEditor-Binder-Plugin. The goal of this environment is to contain as many of the most popular software packages needed for scientific computing from a variety of languages: Python, R, Octave, Sage, Julia, etc.

## Repo2docker Configuration Files
- [environment.yml](https://github.com/LibreTexts/default-env/blob/master/environment.yml). This file contains all the packages to be installed by conda. It is generally used for R, Python and C/C++ packages. We only install packages available from the conda-forge and default channels here.
- [apt.txt](https://github.com/LibreTexts/default-env/blob/master/apt.txt). Here we install Debian packages through `apt-get` which otherwise are not available through conda, such as texlive and some base functionality tools. We will only install from Ubuntu 18.04 APT repositories. 
- [Project.toml](https://github.com/LibreTexts/default-env/blob/master/Project.toml). This is the file where repo2docker installs Julia packages.
- [postBuild](https://github.com/LibreTexts/default-env/blob/master/postBuild). This is a shell script which runs after everything else has been built and installed. We enable JupyterLab extensions here. 

You can read more about each configuration file's purpose from the [repo2docker documentation](https://repo2docker.readthedocs.io/en/latest/config_files.html).

## How to Build a New Image

### Step 1: Pre-Setup

For JupyterTeam members, it is recommended that you use our binder.libretexts.org deployment in order to build new images. It is generally faster and easier this way. You simply need to clone this repository with `git clone https://github.com/LibreTexts/default-env.git` in order to make the changes you need.

If you plan to build the image locally, check the README on git tags <=3.1.0.

### Step 2: Making Changes

1. Before beginning, it is usually good practice to use staging (or make a new branch) and make your changes there. If you are only doing something minor, and have someone else available to immediately review, you may find it easier to make the changes directly in the master branch. Use your best judgement and ask if uncertain. 
2. Using nano, vim, or another text-editor of your choice, make changes to add/remove/alter packages within the relevant [repo2docker configuration file(s)](#repo2docker-configuration-files). For the most part you should be able to follow the intallation pattern already present within the file. For instance, all packages installed via conda (such as those for Python and R) will go into `environment.yml` under the `dependencies: ` section with a format of `  -<package-name>=<version-number`. When adding packages, be sure to add a comment in the file about what the package is and why it was included.
3. After you have made your changes, be sure to commit and push them to this repository so that binder.libretexts.org can access your changes and build the image.

### Step 3: Building and Pushing an Image

1. While you can use the web browser interface to build your image, it's suggested that you instead interact directly with binder.libretexts.org through [API calls](https://binderhub.readthedocs.io/en/latest/api.html) because the browser sometimes times out. In a terminal shell like bash, try `curl https://binder.libretexts.org/build/gh/LibreTexts/default-env/<branch>`, where `<branch>` is the branch with your changes, to build your image and recieve output in the terminal. This process is automated.
2. After the image is done building (could be upwards of 30 minutes), binder will immediatly begin to push it to our dockerhub. Once pushed, you will need to login to the JupyterTeam dockerhub (credentials in metalc-configurations) and find the repository called `libretexts/binder-dev-libretexts-2ddefault-2denv-1cb626`. You will find your image uploaded here. The images are labelled by the git hash value, which is given to each git commit and can be viewed on github or with `git log`. Usually the image most recently uploaded is the one that you are looking for.
3. After the image is done pushing, binder will try to launch the image. At this stage, you may see an `internal server error` and the image will stop launching. As of now, we don't know why these happens, but they are not a problem with the image. If you really need to open the image in binder, just retry until it works.

### Step 4: Testing the Image

Before deploying the image to the production environment at jupyter.libretexts.org, you will want to test the image on staging.jupyter.libretexts.org.
1. The image that our hubs use is specified in the helm values of their helm charts. On the development branch of the galaxy-control-repo, navigate to `galaxy-control-repo/kubernetes/staging-jhub/config.yaml`, and look for the following lines. Update the `tag: <git-hash>` value with the git hash value that corresponds to your image on dockerhub;
```
image:
    name: libretexts/binder-dev-libretexts-2ddefault-2denv-1cb626
    tag: "<git-hash>" # 3.X.X
```
Make sure to leave a comment with the default-env version number so other members can easily tell which image is being used. 

2. After making your changes, commit and push to github. Make a pull request from the development to production branch (for notification purposes), and you can immediatly merge it.
3. Once your changes are on the production branch, ssh into a management node on the cluster (such as `gravity`) and `cd ~/galaxy-control-repo/kubernetes/staging-jhub`. Run `git pull` to grab the changes made on the production branch and then run `./upgrade.sh` in this directory to apply the helm chart with your new image. This may take upwards of 10 minutes.
4. Once the upgrade is complete, you can go to staging.jupyter.libretexts.org, spawn the default environment, and you may test your image in JupyterLab.

### Step 5: Deploying to Production

If you find that the test on staging was successful, you can now deploy the image on production. 
1. You will basically repeat Step 4 except with `jhub/config.yaml` now. Be sure to leave a comment here as well about the version number and use the development branch.
2. Follow the same pull request from development to production process in galaxy-control-repo.
3. On the management node, now do `cd ~/galaxy-control-repo/kubernetes/jhub` and do git pull again. You will also run `./upgrade.sh` in this directory to apply the new helm chart.
4. Once this upgrade is complete, you should be able to verify that your new image is now deployed at jupyter.libretexts.org.

### Step 6: Committing to Github

This step may be done any time after you have verified that the image works as intended.
1. If you made your changes to default-env within a branch other than master, make a pull request to the master branch and merge it (or do so locally).
2. Tag the branch that you built the image on with your desired version number by running `git tag 3.X.X`. [Tags](https://twitter.com/kyliebytes/status/1405941706650841091) work similarly to branches and you will tag whichever commit you are currently located at. Push the tag with `git push origin 3.X.X`. If you need to update a tag, use the `-f` parameter like `git tag -f 3.X.X`. Generally, the major version number will be changed with each major release of JupyterLab, while the minor version number can be updated when new things like packages or kernels are added. The final patch number is for small fixes.

Note: It is important that you tag the commit that you actually built and tested the image on. If you tag a branch after a pull request has been made to it, this would trigger a rebuild if the tag is referenced in binder since pull requests count as new commits. Keep this in mind for CKEditor-Binder-Plugin, which references the git tag and not the exact dockerhub image git hash.

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
