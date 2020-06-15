# Default Environment
This environment is based on the Jupyter Datascience Notebook and Jupyter
Scipy Notebook. It builds on top of a tagged version of the Jupyter Minimal
Notebook. At the moment, this contains the Python, R, Julia, Octave, and
SageMath environments. Since SageMath relies on Python 2, both Python 3 and
2 are supported.

## How to Change the Docker Image

### Making your change
Clone this repository and create another branch.
```
git clone https://github.com/LibreTexts/default-env.git
git checkout -b <your-branch-name>
```

Make the changes you want to make, commit, push your branch, then make a
pull request.
```
git commit -m "Your commit message"
git push origin <your-branch-name>
```

### Testing your image
When you create a pull request, you will trigger a Docker CI build under
the [default-env Docker
repository](https://hub.docker.com/repository/docker/libretexts/default-env).
You can check on hub.docker.com and login to our account to check the build
progress. Once the build is done, your pull request will have a build
status of pass or fail.

> Note: the tag of the image will be marked by the first 12 characters of
> the last commit SHA that you pushed up.

If it's passing, we suggest going further and testing the image on the
staging JupyterHub. The URL for the staging JupyterHub is:
https://staging.jupyter.libretexts.org/hub/login

To test, go to staging jupyter on rooster (by running `cd
~/jupyterhub-staging`), replace the tag name in the yaml file to the tag
name that you assigned to the test image, and then run ```./upgrade.sh```

> When JupyterHub upgrades, the hub will try to pull down the image on all
> nodes before marking the upgrade as complete. However, there is a current
> issue where some chicks have very low hard disk space and will become
> full, preventing the upgrade from proceeding (particularly chick11,
> chick14, chick15, and chick18).
>
> In order to get around this, I usually ssh into the chick and run `sudo
> docker container prune`. This seems to clear a lot of the space, as you
> can see if you run the command `df`.

### Pushing image to Production

After testing, merge your PR into the `master` branch. Change the Default
Environment on the production cluster by editing `~/jupyterhub/config.yaml`
and replacing the tag name with an updated version number. Then, run
`./upgrade.sh`. 
```
singleuser:
  defaultUrl: "/lab"
  ...
  image:
    name: libretexts/default-env # You shouldn't need to change the image name.
    tag: "1.7"  # Change the tag of the Docker image here!
  profileList:
    - display_name: "Default Environment"
      description: "With Python, R, Julia, Octave, and SageMath. Includes packages requested by the community."
      default: true
  ...
```

## How to Build a New Image Locally
Sometimes, you might want to build the Docker image locally, because you
want to test out larger changes without going through the effort of making
a PR, etc. Usually, this isn't necessary, but we included the steps here
anyway in case if you wish to try it out yourself.

First, install Docker. The community edition should suffice. Clone the
repository: `git clone https://github.com/LibreTexts/default-env.git`. The
Dockerfile for the Default Environment is stored in
~/rich-default/Dockerfile.

Edit the Dockerfile, then build by running docker build -t
libretexts/<image name>:<tagname> . I typically build to an image name of
default-test and a relevant tag to test first, then later build the same
image to default-env with sequential version number as the tag.

Push the Dockerfile to DockerHub by running `docker push libretexts/<image
name>:<tagname>`. Use the LibreTexts login credentials when pushing. If
you're getting errors logging into DockerHub, check out this
troubleshooting page. To push to DockerHub, run `docker push
libretexts/<image name>:<tagname>`. If you go to DockerHub, you should see
your image and tag pushed.

> Note: We use the image default-env to denote Docker builds intended for
the Default Environment. I created another Docker image repository called
`default-test` to test our own locally built versions, but you can use
whatever you wish.

## RStudio
You can access RStudio through JupyterHub! Change /lab to /rstudio in the
URL after spawning your server, or click the RStudio button inside JupyterLab.

## Packages
To view all current packages available on the default environment, view the
[environment.yml](https://github.com/LibreTexts/default-env/blob/master/rich-default/environment.yml)
file in rich-deafult.
