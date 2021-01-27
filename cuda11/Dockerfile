FROM nvidia/cuda:11.0-base

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


# Install Miniconda
RUN curl -so ~/miniconda.sh https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh \
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

# install ppytorch 1.7.1 with CUDA 11.0
RUN conda install -y -c pytorch \
    cudatoolkit=11.0.221 \
    magma-cuda110=2.5.2 \
    pytorch=1.7.1 \
    torchtext=0.8.1 \
    torchvision=0.8.2 &&\
    conda clean -ya && \
    python -c "import torch; print('pytorch %s' % torch.__version__)"

# install rdkit and pymatgen
RUN conda install -y -c rdkit rdkit=2020.03.3.0 &&\
    conda clean -ya && \
    python -c "import rdkit; print('rdkit %s' % rdkit.__version__); from rdkit import Chem"

# install other packages
RUN echo "\nInstall using conda -------------------------" \
    && conda install -y -c defaults \
    scikit-learn=0.23.2 \
    scipy=1.5.2 \
    pandas=1.2.1 \
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
    && conda clean -ya \
    && echo "\nInstall using pip -------------------------" \
    && pip install --no-cache-dir \
    geopandas \
    optuna \
    fastcluster \
    xgboost \
    pybtex \
    mordred \
    tensorly \
    pymatgen \
    matminer \
    ipympl \
    GPy \
    umap-learn \
    minepy \
    ngboost \
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
    && python -c "import pandas; print('pandas %s' % pandas.__version__)" \
    && python -c "import geopandas; print('geopandas %s' % geopandas.__version__)" \
    && python -c "import shapely; print('shapely %s' % shapely.__version__)" \
    && python -c "import mordred; print('mordred %s' % mordred.__version__)" \
    && python -c "import tensorly; print('tensorly %s' % tensorly.__version__)" \
    && python -c "import pymatgen; print('pymatgen %s' % pymatgen.__version__)" \
    && python -c "import matminer; print('matminer %s' % matminer.__version__)" \
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