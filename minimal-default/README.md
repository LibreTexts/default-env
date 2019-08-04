# Minimal Default Environment

This environment is based on the 
[Jupyter Scipy Notebook](https://github.com/jupyter/docker-stacks/tree/master/scipy-notebook).

This only contains the Python environment.
It adds the following packages, as requested in
[this issue](https://github.com/LibreTexts/metalc/issues/19):
* matplotlib=3.1.*
* numba=0.42*
* numpy=1.16.*
* astropy=3.2.*
* bqplot=0.11.*
* nb_conda_kernels=2.2.*
* requests=2.22.*

To build, run `docker build -t libretexts/<image name>:<tagname> .`
