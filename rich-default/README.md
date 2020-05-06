# Rich Default Environment
This environment is based on the Jupyter Datascience Notebook and Jupyter Scipy Notebook. It builds on top of a tagged version of the Jupyter Minimal Notebook.
At the moment, this contains the Python, R, Julia, Octave, and SageMath environments. Since SageMath relies on Python 2, both Python 3 and 2 are supported.

## How to Build a New Image
First, install Docker. The community edition should suffice.
Clone the repository: git clone https://github.com/LibreTexts/default-env.git. The Dockerfile for the Default Environment is stored in ~/rich-default/Dockerfile.
Edit the Dockerfile, then build by running docker build -t libretexts/<image name>:<tagname> . I typically build to an image name of default-test and a relevant tag to test first, then later build the same image to default-env with sequential version number as the tag.
Push the Dockerfile to DockerHub by running docker push libretexts/<image name>:<tagname>. Use the LibreTexts login credentials when pushing. If you're getting errors logging into DockerHub, check out this troubleshooting page.
To push to DockerHub, run docker push libretexts/<image name>:<tagname>. If you go to DockerHub, you should see your image and tag pushed.

Note: We use the image default-env to denote Docker builds intended for the Default Environment. Tags are labeled by version numbers (i.e. 1.0, 1.1, etc.).
The image default-test is used for temporary test Docker builds. I use relevant, memorable tags. 

## Testing
I suggest testing the environment out on the staging JupyterHub before deploying on the production cluster. The URL for the staging JupyterHub is: https://staging.jupyter.libretexts.org/hub/login
To test, go to staging jupyter on rooster, replace the tag name in the yaml file to the tag name that you assigned to the test image, and then run ```./upgrade.sh```

## Production
After testing, change the Default Environment on the production cluster by editing config.yaml and replacing the tag name with an updated version number. 

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

