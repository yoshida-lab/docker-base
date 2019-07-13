# stage1
FROM alpine:latest as s1

LABEL maintainer="TsumiNa <liu.chang.1865@gmail.com>"

# Install some basic utilities
RUN apk add --no-cache -t build-dependencies \
    build-base \
    linux-headers \
    gfortran \
    lapack-dev \
    bash \
    curl 
# perl

RUN echo "install mpich" \
    && curl -L http://www.mpich.org/static/downloads/3.3.1/mpich-3.3.1.tar.gz | tar zvx \
    && cd mpich-3.3.1 \
    && ./configure --prefix=/mpich \
    && make -j 4 \
    && make install \
    && make clean \
    && cd / && rm -rf /mpich-3.3.1

ENV PATH=/mpich/bin:$PATH
ENV LD_LIBRARY_PATH=/mpich/lib:$LD_LIBRARY_PATH

RUN cd / \
    && echo '#include <stdio.h>' >> test.c \
    && echo '#include <mpi.h>' >> test.c \
    && echo '    int main(int argc, char *argv) {' >> test.c \
    && echo '    int rank, proc;' >> test.c \
    && echo '    int name_length = 10;' >> test.c \
    && echo '    char *name[name_length];' >> test.c \
    && echo '    MPI_Init(&argc, &argv);' >> test.c \
    && echo '    MPI_Comm_rank(MPI_COMM_WORLD, &rank);' >> test.c \
    && echo '    MPI_Comm_size(MPI_COMM_WORLD, &proc);' >> test.c \
    && echo '    MPI_Get_processor_name(name, &name_length);' >> test.c \
    && echo '    printf("%s : %d of %d\n", name, rank, proc);' >> test.c \
    && echo '    MPI_Finalize();' >> test.c \
    && echo '    return 0;' >> test.c \
    && echo '}' >> test.c \
    && mpicc test.c -o test \
    && mpiexec -n 2 ./test \
    && rm -f test.c test

RUN echo "install abinit" \
    && curl -L https://www.abinit.org/sites/default/files/packages/abinit-8.10.3.tar.gz | tar zvx \
    && cd abinit-8.10.3 \
    && ./configure --prefix=/abinit FC=mpifort CC=mpicc CXX=mpicxx\
    --with-dft-flavor=wannier90 \
    --with-trio-flavor="netcdf" \
    && make -j 4 \
    && make install \
    && make clean \
    && cd / && rm -rf /abinit-8.10.3


#stage 2
FROM frolvlad/alpine-miniconda3:python3.7

RUN apk add --no-cache lapack bash openssh
COPY --from=s1 /mpich /mpich/
COPY --from=s1 /abinit /abinit/

ENV PATH=/mpich/bin:/abinit/bin:$PATH
ENV LD_LIBRARY_PATH=/mpich/lib:/abinit/lib:$LD_LIBRARY_PATH

RUN conda config --add channels matsci \
    && conda config --add channels conda-forge \
    && conda config --add channels abinit \
    && conda install -y \
    numpy \
    scipy \
    pandas \
    jupyter \
    abipy \
    pymatgen \
    jupyterlab \
    && conda clean -ya 

CMD [ "python" ]
