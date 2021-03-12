FROM nvidia/cuda:10.2-base

# Install some basic utilities
RUN apt-get update && apt-get install -y \
    curl \
    build-essential \
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


# Install Mambaforge (move from Miniconda)
RUN curl -so ~/mambaforge.sh -L https://github.com/conda-forge/miniforge/releases/latest/download/Mambaforge-Linux-x86_64.sh \
    && chmod +x ~/mambaforge.sh \
    && ~/mambaforge.sh -b -p ~/mambaforge \
    && rm ~/mambaforge.sh \
    && umask 000
ENV PATH=/home/user/mambaforge/condabin:$PATH
ENV CONDA_AUTO_UPDATE_CONDA=false

# Create a Python 3.8 environment
RUN /home/user/mambaforge/condabin/mamba update -n base mamba conda \
    && /home/user/mambaforge/condabin/mamba install conda-build \
    && /home/user/mambaforge/condabin/mamba create -y --name py38 python=3.8 \
    && /home/user/mambaforge/condabin/mamba clean -ya
ENV CONDA_DEFAULT_ENV=py38
ENV CONDA_PREFIX=/home/user/mambaforge/envs/$CONDA_DEFAULT_ENV
ENV PATH=$CONDA_PREFIX/bin:$PATH

# install pytorch 1.7.1 with CUDA 10.2
RUN conda install -y -n py38 -c pytorch \
    cudatoolkit=10.2 \
    magma-cuda102=2.5.2 \
    pytorch=1.7.1 \
    torchtext=0.8.1 \
    # torchvision=0.8.2 \
    && mamba clean -ya \
    && python -c "import torch; print('pytorch %s' % torch.__version__)"

# install other packages
RUN echo "\nInstall using mamba -------------------------" \
    && mamba install -y -n py38 \
    'libblas=*=*mkl' \
    rdkit=2020.09.5 \
    scikit-learn=0.24 \
    scipy=1.6 \
    pandas=1.2 \
    jedi>=0.16 \
    shapely \
    descartes \
    requests \
    joblib \
    lz4 \
    seaborn \
    openpyxl \
    nodejs \
    numba \
    pytest \
    jupyterlab \
    ipywidgets \
    widgetsnbextension \
    plotly \
    geopandas \
    optuna \
    fastcluster \
    xgboost \
    pybtex \
    mordred \
    tensorly \
    'pymatgen>=2020.10.9' \
    matminer \
    ipympl \
    GPy \
    umap-learn \
    minepy \
    ngboost \
    && mamba clean -ya \
    && echo "\nInstall using pip -------------------------" \
    && pip install --no-cache-dir \
    OpenNMT-py==1.2.0 \
    Python-Deprecated \
    && echo "\nInstall labextension -------------------------" \
    && jupyter labextension install --no-build @jupyter-widgets/jupyterlab-manager \
    && jupyter labextension install --no-build jupyterlab-plotly \
    # && jupyter labextension install --no-build jupyterlab-chart-editor \
    && jupyter lab build  --dev-build=False --minimize=False\
    && jupyter nbextension enable --py widgetsnbextension \
    && python -c "import numpy; print('numpy %s' % numpy.__version__)" \
    && python -c "import scipy; print('scipy %s' % scipy.__version__)" \
    && python -c "import sklearn; print('sklearn %s' % sklearn.__version__)" \
    && python -c "import pandas; print('pandas %s' % pandas.__version__)" \
    && python -c "import geopandas; print('geopandas %s' % geopandas.__version__)" \
    && python -c "import shapely; print('shapely %s' % shapely.__version__)" \
    && python -c "import mordred; print('mordred %s' % mordred.__version__)" \
    && python -c "import tensorly; print('tensorly %s' % tensorly.__version__)" \
    && python -c "import pymatgen; print('pymatgen %s' % pymatgen.__version__)" \
    && python -c "import matminer; print('matminer %s' % matminer.__version__)" \
    && python -c "import rdkit; print('rdkit %s' % rdkit.__version__); from rdkit import Chem" \
    && echo "\nCleanup -------------------------" \
    && jupyter lab clean -y \
    && npm cache clean --force \
    && rm -rf "/home/user/.cache/yarn" \
    && rm -rf "/home/user/.node-gyp" \
    && sudo rm -rf /var/lib/apt/lists/* 

# allow matplotlib working in docker
WORKDIR /etc
RUN echo "backend : Agg" | sudo tee -a matplotlibrc 

# Set the default command to python3
CMD ["python3"]