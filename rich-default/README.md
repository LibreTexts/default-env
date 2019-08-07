# Rich Default Environment

This environment is based on the
[Jupyter Datascience Notebook](https://github.com/jupyter/docker-stacks/tree/master/datascience-notebook)
and 
[Jupyter Scipy Notebook](https://github.com/jupyter/docker-stacks/tree/master/scipy-notebook).
It builds on top of a tagged version of the 
[Jupyter Minimal Notebook](https://github.com/jupyter/docker-stacks/tree/master/minimal-notebook).

At the moment, this contains the Python, R, Julia, 
and Octave environments.

## Packages
As requested in
[this issue](https://github.com/LibreTexts/metalc/issues/19),
the following packages are installed into the root conda
environment. Some packages come from 
[Jupyter Datascience Notebook](https://github.com/jupyter/docker-stacks/tree/master/datascience-notebook).

* conda-forge::blas=*=openblas
* python=3.6.*
* ipywidgets
* pandas
* numexpr
* matplotlib
* scipy
* seaborn
* scikit-learn
* scikit-image
* sympy
* cython
* numba
* bokeh
* numpy
* astropy
* bqplot
* nb_conda_kernels
* statsmodels
* patsy
* requests
* cloudpickle
* dill
* dask
* sqlalchemy
* hdf5
* h5py
* vincent
* beautifulsoup4
* protobuf
* xlrd
* numba

The Octave kernel relies on these installed packages.
* octave
* gnuplot
* ghostscript
* octave_kernel

These R kernel includes the following packages.
* rpy2
* r-base
* r-irkernel
* r-plyr
* r-devtools
* r-tidyverse
* r-shiny
* r-rmarkdown
* r-forecast
* r-rsqlite
* r-reshape2
* r-nycflights13
* r-caret
* r-rcurl
* r-crayon
* r-randomforest
* r-htmltools
* r-sparklyr
* r-htmlwidgets
* r-hexbin

## Build
To build, run `docker build -t libretexts/<image name>:<tagname> .`

