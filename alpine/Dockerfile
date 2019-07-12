FROM frolvlad/alpine-miniconda3:python3.7

LABEL maintainer="TsumiNa <liu.chang.1865@gmail.com>"

# Install some basic utilities
RUN apk add --no-cache -t build-dependencies \
    build-base \
    linux-headers \
    gfortran \
    curl \
    bash \
    perl \
    cmake \
    git \
    && echo "install mpich" \
    && curl -L http://www.mpich.org/static/downloads/3.3.1/mpich-3.3.1.tar.gz | tar zvx \
    && cd mpich-3.3.1 \
    && ./configure \
    && make -j 4 \
    && make install \
    && make clean \
    && cd / && rm -rf /mpich-3.3.1


COPY sample.c .
# RUN cd / \
#     && mpicc sample.c -o test \
#     && mpiexec -n 2 ./test \
#     && rm -f sample.c test