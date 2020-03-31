FROM nvidia/cuda:10.1-base

# Install some basic utilities
RUN apt-get update && apt-get install -y \
    curl \
    ca-certificates \
    openssh-client \
    git \
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

# Create a Python 3.7 environment
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
    cudatoolkit=10.1 \
    magma-cuda101=2.5.1 \
    pytorch=1.4.0 \
    torchtext=0.5.0 \
    torchvision=0.5.0 &&\
    conda clean -ya && \
    python -c "import torch; print('pytorch %s' % torch.__version__)"

# install rdkit and pymatgen
RUN conda install -y -c rdkit rdkit=2020.03.1.0 &&\
    conda clean -ya && \
    python -c "import rdkit; print('rdkit %s' % rdkit.__version__); from rdkit import Chem"

# install other packages
RUN sudo apt-get update \
    && sudo apt-get install -y gcc g++ \
    && conda install -y \
    scikit-learn=0.22.1 \
    scipy=1.4.1 \
    pandas=1.0.3 \
    requests \
    joblib \
    lz4 \
    seaborn \
    openpyxl \
    nodejs \
    && conda clean -ya \
    && pip install --upgrade --no-cache-dir \
    git+https://github.com/stanfordmlgroup/ngboost.git \
    && pip install --no-cache-dir \
    jupyterlab \
    ipywidgets \
    widgetsnbextension \
    fastcluster \
    xgboost \
    plotly \
    pybtex \
    mordred \
    torchtext \
    Python-Deprecated \
    tensorly \
    pymatgen==2020.3.13 \
    matminer\
    && jupyter labextension install --no-build @jupyter-widgets/jupyterlab-manager \
    && jupyter labextension install --no-build jupyterlab-plotly \
    # && jupyter labextension install --no-build jupyterlab-chart-editor \
    && jupyter lab build \
    && jupyter nbextension enable --py widgetsnbextension \
    && python -c "import numpy; print('numpy %s' % numpy.__version__)" \
    && python -c "import scipy; print('scipy %s' % scipy.__version__)" \
    && python -c "import pandas; print('pandas %s' % pandas.__version__)" \
    && python -c "import mordred; print('mordred %s' % mordred.__version__)" \
    && python -c "import tensorly; print('tensorly %s' % tensorly.__version__)" \
    && python -c "import pymatgen; print('pymatgen %s' % pymatgen.__version__)" \
    && sudo apt-get purge gcc g++ -y\
    && sudo apt-get autoremove -y \
    && sudo apt-get clean -y \ 
    && sudo rm -rf /var/lib/apt/lists/* 

# allow matplotlib working in docker
WORKDIR /etc
RUN echo "backend : Agg" | sudo tee -a matplotlibrc 

# Set the default command to python3
CMD ["python3"]