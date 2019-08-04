ARG BASE_CONTAINER=jupyter/datascience-notebook
FROM $BASE_CONTAINER

LABEL maintainer="Libretexts Jupyter Team <libretextsteam@gmail.com>"

USER $NB_UID

# Installs additional packages to the root environment
RUN conda install --quiet --yes \
#    'matplotlib=3.1.*' \
    'numba=0.42*' \
    'numpy=1.16.*' \
    'astropy=3.2.*' \
    'bqplot=0.11.*' \
    'nb_conda_kernels=2.2.*' \
    'requests=2.22.*' && \
    conda clean --all -f -y && \
    jupyter lab build --dev-build=False && \
    npm cache clean --force && \
    rm -rf $CONDA_DIR/share/jupyter/lab/staging && \
    rm -rf /home/$NB_USER/.cache/yarn && \
    rm -rf /home/$NB_USER/.node-gyp && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

# Downgrades conda to install octave
RUN conda config --set allow_conda_downgrades true
RUN conda install conda=4.6.14

# Install the Octave and Sage kernels
RUN conda install --quiet --yes \
    'octave=4.2.1' \
    'octave_kernel=0.28.*' && \
#    'sage=8.8' && \
    conda clean --all -f -y && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

# Modified snippet for octave
#USER root
#WORKDIR /tmp
#
# Install octave and octave_kernel
#RUN apt-get install -y --no-install-recommends \
#        octave liboctave-dev \
#        gnuplot-x11 \
#        libopenblas-base libatlas3-base \
#        ghostscript pstoedit && \
#    pip install -U sympy && \
#    octave --eval 'pkg install -forge struct parallel symbolic odepkg' && \
#    pip3 install -U octave_kernel && \
#    python3 -m octave_kernel.install && \
#    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Original snippet for octave
#USER root
#WORKDIR /tmp
#
## Install octave and octave_kernel
#RUN apt-get update && \
#    apt-get install -y --no-install-recommends \
#        build-essential make \
#        octave liboctave-dev \
#        gnuplot-x11 \
#        libopenblas-base libatlas3-base \
#        ghostscript pstoedit && \
#    pip install -U sympy && \
#    octave --eval 'pkg install -forge struct parallel symbolic odepkg' && \
#    pip3 install -U octave_kernel && \
#    python3 -m octave_kernel.install && \
#    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*


USER $NB_UID
