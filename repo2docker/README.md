# How to Build

1. Be sure to install repo2docker from source using `pip install git+https://github.com/jupyterhub/repo2docker.git`
2. Update the docker image which repo2docker builds on top of with `docker pull buildpack-deps:bionic`
3. Navigate to `/default-env/repo2docker/` and run `repo2docker --user-name jovyan --user-id 1000 .` to start the build. We ensure that we do not break the filesystem by specifying `jovyan` as the username, `user-id 1000` specifies a normal user account (not root), and `.` starts the build in the current directory
4. Tag the image and push to dockerhub
