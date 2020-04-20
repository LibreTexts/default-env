# This Dockerfile adds to the Jupyter SciPy Notebook Dockerfile, found
# in this link: https://github.com/jupyter/docker-stacks/blob/master/scipy-notebook/Dockerfile
# Installs packages in the root environment.
# Includes the Python, R, Julia, Octave, and SageMath kernels.
# To build, run: docker build -t libretexts/default-test:<tagname> .
# Don't miss the "." at the end of the previous command.

# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.
ARG BASE_CONTAINER=jupyter/minimal-notebook:58169ec3cfd3
FROM $BASE_CONTAINER

ARG TEST_ONLY_BUILD

LABEL maintainer="Libretexts Jupyter Team <libretextsteam@gmail.com>"

USER root

# Install ffmpeg for matplotlib anim
# Install R prerequisites: fonts-dejavu, gfortran, gcc
# Install Octave and its prerequisites: octave, octave-control
# octave-image, octave-io, octave-optim, octave-signal, octave-statistics
# vim added to 1.9, staging jupyter not working
RUN apt-get update && \
    apt-get install -y --no-install-recommends ffmpeg \
    fonts-dejavu \
    gfortran \
    gcc \
    octave \
    vim \
    octave-* \
    texlive-full && \ 
    rm -rf /var/lib/apt/lists/*

USER $NB_UID

COPY environment.yml environment.yml

RUN conda env update --name base --file environment.yml --prune && \
    conda clean --all -f -y && \
    # Activate ipywidgets extension in the environment that runs the notebook server
    jupyter nbextension enable --py widgetsnbextension --sys-prefix && \
    # Also activate ipywidgets extension for JupyterLab
    # Check this URL for most recent compatibilities
    # https://github.com/jupyter-widgets/ipywidgets/tree/master/packages/jupyterlab-manager
    # jupyter lab matplotlibextension added to 1.9, staging jupyter not working
    jupyter labextension install @jupyter-widgets/jupyterlab-manager@^1.0.1 --no-build && \
    jupyter labextension install jupyterlab_bokeh@1.0.0 --no-build && \
    jupyter labextension install jupyter-matplotlib@0.5.0 --no-build && \
    jupyter lab build --dev-build=False && \
    npm cache clean --force && \
    rm -rf $CONDA_DIR/share/jupyter/lab/staging && \
    rm -rf /home/$NB_USER/.cache/yarn && \
    rm -rf /home/$NB_USER/.node-gyp && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

# Install facets which does not have a pip or conda package at the moment
RUN cd /tmp && \
    git clone https://github.com/PAIR-code/facets.git && \
    cd facets && \
    jupyter nbextension install facets-dist/ --sys-prefix && \
    cd && \
    rm -rf /tmp/facets && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

# Import matplotlib the first time to build the font cache.
ENV XDG_CACHE_HOME /home/$NB_USER/.cache/
RUN MPLBACKEND=Agg python -c "import matplotlib.pyplot" && \
    fix-permissions /home/$NB_USER

USER root
# Install SageMath and its dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    m4 \
    sagemath \
    sagemath-jupyter \
    sagemath-doc-en  && \
    rm -rf /var/lib/apt/lists/*

# Links Sage to Python 2 instead of Python 3
# Replaces the first line of the sage-ipython file to recognize
# the Python 2 environment
RUN sed -i -e '1s:#!/usr/bin/env python:#!/usr/bin/env python2:' /usr/share/sagemath/bin/sage-ipython

# Links the Sage kernel to Python 2 instead of Python 3 by editing the json file
RUN sed -i 's/--python/--python2/' /usr/share/jupyter/kernels/sagemath/kernel.json

# Julia dependencies
# # install Julia packages in /opt/julia instead of $HOME
ENV JULIA_DEPOT_PATH=/opt/julia
ENV JULIA_PKGDIR=/opt/julia
ENV JULIA_VERSION=1.1.0


RUN mkdir /opt/julia-${JULIA_VERSION} && \
    cd /tmp && \
    wget -q https://julialang-s3.julialang.org/bin/linux/x64/`echo ${JULIA_VERSION} | cut -d. -f 1,2`/julia-${JULIA_VERSION}-linux-x86_64.tar.gz && \
    echo "80cfd013e526b5145ec3254920afd89bb459f1db7a2a3f21849125af20c05471 *julia-${JULIA_VERSION}-linux-x86_64.tar.gz" | sha256sum -c - && \
    tar xzf julia-${JULIA_VERSION}-linux-x86_64.tar.gz -C /opt/julia-${JULIA_VERSION} --strip-components=1 && \
    rm /tmp/julia-${JULIA_VERSION}-linux-x86_64.tar.gz
RUN ln -fs /opt/julia-*/bin/julia /usr/local/bin/julia

# Show Julia where conda libraries are \
RUN mkdir /etc/julia && \
    echo "push!(Libdl.DL_LOAD_PATH, \"$CONDA_DIR/lib\")" >> /etc/julia/juliarc.jl && \
    # Create JULIA_PKGDIR \
    mkdir $JULIA_PKGDIR && \
    chown $NB_USER $JULIA_PKGDIR && \
    fix-permissions $JULIA_PKGDIR

USER $NB_UID

# Add Julia packages. Only add HDF5 if this is not a test-only build since
# it takes roughly half the entire build time of all of the images on Travis
# to add this one package and often causes Travis to timeout.
#
# Install IJulia as jovyan and then move the kernelspec out
# to the system share location. Avoids problems with runtime UID change not
# taking effect properly on the .local folder in the jovyan home dir.
RUN julia -e 'import Pkg; Pkg.update()' && \
     (test $TEST_ONLY_BUILD || julia -e 'import Pkg; Pkg.add("HDF5")') && \
         julia -e "using Pkg; pkg\"add Gadfly RDatasets IJulia InstantiateFromURL\"; pkg\"precompile\"" && \ 
    # move kernelspec out of home \
    mv $HOME/.local/share/jupyter/kernels/julia* $CONDA_DIR/share/jupyter/kernels/ && \
    chmod -R go+rx $CONDA_DIR/share/jupyter && \
    rm -rf $HOME/.local && \
    fix-permissions $JULIA_PKGDIR $CONDA_DIR/share/jupyter

# Install RStudio and its Jupyter extension, made available by interchanging
# /lab with /rstudio in the URL.

# Installs RStudio
USER root
RUN apt-get update && \
    curl --silent -L --fail https://download2.rstudio.org/rstudio-server-1.1.419-amd64.deb > /tmp/rstudio.deb && \
    echo '24cd11f0405d8372b4168fc9956e0386 /tmp/rstudio.deb' | md5sum -c - && \
    apt-get install -y /tmp/rstudio.deb && \
    rm /tmp/rstudio.deb && \
    apt-get clean
ENV PATH=$PATH:/usr/lib/rstudio-server/bin

USER $NB_UID

# for some reason, need to rebuild it to make widgets work
RUN jupyter labextension install jupyter-matplotlib@0.5.0 --no-build
RUN jupyter lab build

