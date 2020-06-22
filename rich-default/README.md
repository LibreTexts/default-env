# Rich Default Environment
This environment is based on the Jupyter Datascience Notebook and Jupyter Scipy Notebook. It builds on top of a tagged version of the Jupyter Minimal Notebook.
At the moment, this contains the Python, R, Julia, Octave, and SageMath environments. Since SageMath relies on Python 2, both Python 3 and 2 are supported.

## How to Build a New Image

### Step 1: Pre-Setup
> If you are on `Rooster`, Docker is already installed, and the repo has already been cloned, follow these steps to access, otherwise continue with the the regular process. 
1. ``cd jupyterhub/default-env/rich-default/``
2. ``vim Dockerfile``
3. To edit the ``Dockerfile``, we use vim. If you are unfamiliar with it, view this [resource](https://www.sitepoint.com/getting-started-vim/) for common commands. 

> Follow the below steps if not on `Rooster`:
1. Install Docker. The community edition should suffice.
2. Clone the repository: ``git clone https://github.com/LibreTexts/default-env.git``. The Dockerfile for the Default Environment is stored in `~/rich-default/Dockerfile`.
3. We use `vim` as our text editor, potential intallation may be necessary if not already available on your machine.  

### Step 2: Editing the Dockerfile and environment.yml
1. If this is your first time editing the `Dockerfile` and the `environment.yml` spend some time reading it over and understanding how they are formatted. 
2. If you are adding a python library, ``vim environment.yml``, and add as follows `- <library name>=<version>`. If you are adding an R package, ``vim environment.yml``, and as follows `- r-<package name>=<version>`. 
3. If you are adding something like `vim` or `latex`, edits will be needed to be made to the `Dockerfile`. If what you are adding is the last line, then it should be added as such `<package name> && \`. Otherwise it can be added as `<package name> \`. Version specification is not needed for this part. 
4. Check if there is a `jupyter labextension` for your package or library. It is critical that it is added to the `Dockerfile`. Version specification may be needed. 

### Step 3: Build and Push the Dockerfile
Once your edits have been made:
1. Build by running `docker build -t libretexts/default-test:<tagname> .` **Do not forget the ` .` at the end**. Typically for your test run, `tagname` is a relevant tag to test, and example would be like `pandas-test`.
2. Push the `Dockerfile` to DockerHub by running `docker push libretexts/<image name>:<tagname>`. Use the LibreTexts login credentials when pushing. If you're getting errors logging into DockerHub, check out this troubleshooting page.
If you go to DockerHub, you should see your image and tag pushed.

### Step 4: Testing
Test the environment out on the staging JupyterHub before deploying on the production cluster.
1. `cd jupyterhub-staging`
2. `vim staging-config.yaml`
3. Replace the tag name in the yaml file to the tag name that you assigned to the test image.
4. `./upgrade.sh`
5. Go to the staging JupyterHub: https://staging.jupyter.libretexts.org/hub/login, and test out the new feature you added.

### Step 5: Deploying to Production
Once you have found that the test on staging was successful, you are ready to push to production.
1. `docker build -t libretexts/default-env:<tagname> .` Build the same image to default-env with sequential version number as the tag. **Note: We use the image default-env to denote Docker builds intended for the Default Environment. Tags are labeled by version numbers (i.e. 1.0, 1.1, etc.).** 
2. `docker push libretexts/default-env:<tagname>`
3. `cd jupyterhub`
4. `vim config.yaml`
5. `./upgrade.sh`
6. If `./upgrade.sh` fails the first time, try to run it a second time. 

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

### RStudio
You can access RStudio through JupyterHub! Change /lab to /rstudio in the URL after spawning your server. (This issue helped install RStudio.)

### Packages
To view all current packages available on the default environment, view the [environment.yml](https://github.com/LibreTexts/default-env/blob/master/rich-default/environment.yml) file in rich-deafult.

