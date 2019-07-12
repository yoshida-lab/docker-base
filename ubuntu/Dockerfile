FROM ubuntu:18.04

LABEL maintainer="TsumiNa <liu.chang.1865@gmail.com>"

RUN echo "dependences Installing..." \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
    build-essential \
    curl \
    git \
    cmake \
    gfortran \
    && rm -rf /var/lib/apt/lists/*  \
    && apt-get clean \
    && echo "install mpich" \
    && curl -L http://www.mpich.org/static/downloads/3.3.1/mpich-3.3.1.tar.gz | tar zvx \
    && cd mpich-3.3.1 \
    && ./configure \
    && make -j 4 \
    && make install \
    && make clean \
    && cd / && rm -rf /mpich-3.3.1

COPY sample.c .
RUN cd / \
    && mpicc sample.c -o test \
    && mpiexec -n 2 ./test \
    && rm -f sample.c test
