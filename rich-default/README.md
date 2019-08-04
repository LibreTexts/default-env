# Rich Default Environment

This environment is builds on top of the
[Jupyter Datascience Notebook](https://github.com/jupyter/docker-stacks/tree/master/datascience-notebook).

At the moment, this contains the Python, R, Julia, 
and Octave environments.

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

