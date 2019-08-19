FROM nvidia/cuda:10.0-base

# Install some basic utilities
RUN apt-get update && apt-get install -y \
    curl \
    ca-certificates \
    sudo \
    bzip2 \
    libxrender1 \
    libxext6 \
    && rm -rf /var/lib/apt/lists/*

# Create a non-root user and switch to it
# All users can use /home/user as their home directory
RUN adduser --disabled-password --gecos '' --shell /bin/bash user &&\
    echo "user ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/90-user &&\
    mkdir /workspace && chown -R user:user /workspace &&\
    chmod 777 -R /home/user && chmod 777 -R /workspace
USER user
ENV HOME=/home/user
WORKDIR /workspace

# Install Miniconda
RUN curl -so ~/miniconda.sh https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh \
    && chmod +x ~/miniconda.sh \
    && ~/miniconda.sh -b -p ~/miniconda \
    && rm ~/miniconda.sh \
    && /home/user/miniconda/bin/conda update -n base -c defaults conda \
    && umask 000
ENV PATH=/home/user/miniconda/bin:$PATH
ENV CONDA_AUTO_UPDATE_CONDA=false

# Create a Python 3.6 environment
RUN /home/user/miniconda/bin/conda update -n base -c defaults conda \
    && /home/user/miniconda/bin/conda install conda-build \
    && /home/user/miniconda/bin/conda create -y --name py37 python=3.7 \
    && /home/user/miniconda/bin/conda clean -ya
ENV CONDA_DEFAULT_ENV=py37
ENV CONDA_PREFIX=/home/user/miniconda/envs/$CONDA_DEFAULT_ENV
ENV PATH=$CONDA_PREFIX/bin:$PATH

# install ppytorch 1.0.0 with CUDA 10.0
# use CUDA 10.0-specific steps
RUN conda install -y -c pytorch \
    cudatoolkit=10.0 \
    magma-cuda100=2.5.0 \
    "pytorch=1.1.0=py3.7_cuda10.0.130_cudnn7.5.1_0" \
    torchvision=0.3.0 &&\
    python -c "import torch; print('pytorch %s' % torch.__version__)"

# install rdkit and pymatgen
RUN conda install -y -c rdkit rdkit=2019.03.2.0 &&\
    python -c "import rdkit; print('rdkit %s' % rdkit.__version__); from rdkit import Chem"

# install other packages
RUN sudo apt-get update \
    && sudo apt-get install -y gcc \
    && conda install -y \
    jupyter=1.0.0 \
    scikit-learn=0.21.2 \
    scipy=1.3.0 \
    pandas=0.24.2 \
    requests \
    joblib \
    nodejs \
    seaborn \
    openpyxl \
    "plotly>=4.0" \
    "pytest>=5.0" \
    "pytest-cov=2.7" \
    && conda clean -ya \
    && pip install --no-cache-dir \
    jupyterlab \
    pybtex \
    mordred \
    torchtext \
    Python-Deprecated \
    tensorly \
    pymatgen==2019.7.2 \
    matminer\
    && jupyter labextension install @jupyterlab/plotly-extension \
    && jupyter labextension install jupyterlab-chart-editor \
    && python -c "import numpy; print('numpy %s' % numpy.__version__)" \
    && python -c "import scipy; print('scipy %s' % scipy.__version__)" \
    && python -c "import pandas; print('pandas %s' % pandas.__version__)" \
    && python -c "import mordred; print('mordred %s' % mordred.__version__)" \
    && python -c "import tensorly; print('tensorly %s' % tensorly.__version__)" \
    && python -c "import pymatgen; print('pymatgen %s' % pymatgen.__version__)" \
    && sudo apt-get purge gcc  -y\
    && sudo apt-get autoremove -y \
    && sudo apt-get clean -y \ 
    && sudo rm -rf /var/lib/apt/lists/* 

# allow matplotlib working in docker
WORKDIR /etc
RUN echo "backend : Agg" | sudo tee -a matplotlibrc 

# Set the default command to python3
CMD ["python3"]