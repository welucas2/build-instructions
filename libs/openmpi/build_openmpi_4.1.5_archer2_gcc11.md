Instructions for building OpenMPI 4.1.5 on ARCHER2
==================================================

These instructions are for building OpenMPI 4.1.5 on ARCHER2 (HPE Cray EX, AMD Zen2 7742) using gcc 11.2.0.
Two sets of instructions are provided, one that builds OpenMPI for the OpenFabrics interface (OFI) and the
other for the UCX communications interface.


Setup initial environment
-------------------------

```bash
PRFX=/path/to/work # e.g., PRFX=/work/y07/shared/libs/dev
OPENMPI_LABEL=openmpi
OPENMPI_VERSION=4.1.5
OPENMPI_VERSION_MAJOR=`echo "${OPENMPI_VERSION}" | cut -d"." -f 1-2`
OPENMPI_NAME=${OPENMPI_LABEL}-${OPENMPI_VERSION}
OPENMPI_ROOT=${PRFX}/${OPENMPI_LABEL}

mkdir -p ${OPENMPI_ROOT}
cd ${OPENMPI_ROOT}

wget https://download.open-mpi.org/release/open-mpi/v${OPENMPI_VERSION_MAJOR}/${OPENMPI_NAME}.tar.gz
tar -xzf ${OPENMPI_NAME}.tar.gz
rm ${OPENMPI_NAME}.tar.gz
cd ${OPENMPI_NAME}

module load PrgEnv-gnu
GNU_VERSION_MAJOR=`echo ${GNU_VERSION} | cut -d'.' -f1`

PMI_HDRS_DIR=/usr/include/slurm
PMI_LIBS_DIR=/usr/lib64
```

Remember to change the setting for `PRFX` to a path appropriate for your ARCHER2 project.


Build and install OpenMPI for OFI
---------------------------------

```bash
OFI_ROOT=`pkg-config --variable=prefix libfabric`

OPENMPI_INSTALL_LABEL=${OPENMPI_VERSION}-ofi-gcc${GNU_VERSION_MAJOR}

./configure CC=cc CXX=CC FTN=ftn \
    CFLAGS="-I${PMI_HDRS_DIR} -O2 -march=znver2 -mno-avx512f" LDFLAGS="-L${PMI_LIBS_DIR}" \
    --enable-mpi1-compatibility --enable-mpi-fortran --enable-mpi-interface-warning \
    --with-ofi=${OFI_ROOT} --with-pmi=${PMI_HDRS_DIR} --with-pmi-libdir=${PMI_LIBS_DIR} \
    --with-slurm --with-singularity \
    --prefix=${OPENMPI_ROOT}/${OPENMPI_INSTALL_LABEL} \
    &> configure.log.${OPENMPI_INSTALL_LABEL}

mv config.log config.log.${OPENMPI_INSTALL_LABEL}

make -j 8
make -j 8 install
make -j 8 clean
```


Build and install OpenMPI for UCX
---------------------------------

```bash
UCX_ROOT=/opt/cray/pe/cray-ucx/2.7.0-1/ucx

# setup link for numa library
mkdir -p ${OPENMPI_ROOT}/liblinks
ln -s /usr/lib64/libnuma.so.1.0.0 ${OPENMPI_ROOT}/liblinks/libnuma.so

module swap craype-network-ofi craype-network-ucx
# cray-mpich not required of course - second module swap simply being done for consistency
module swap cray-mpich cray-mpich-ucx

OPENMPI_INSTALL_LABEL=${OPENMPI_VERSION}-ucx-gcc${GNU_VERSION_MAJOR}

./configure CC=cc CXX=CC FTN=ftn \
    CFLAGS="-I${PMI_HDRS_DIR} -O2 -march=znver2 -mno-avx512f" \
    LDFLAGS="-L${PMI_LIBS_DIR} -L${OPENMPI_ROOT}/liblinks" \
    --enable-mpi1-compatibility --enable-mpi-fortran --enable-mpi-interface-warning \
    --with-ucx=${UCX_ROOT} --with-pmi=${PMI_HDRS_DIR} --with-pmi-libdir=${PMI_LIBS_DIR} \
    --with-slurm --with-singularity \
    --prefix=${OPENMPI_ROOT}/${OPENMPI_INSTALL_LABEL} \
    &> configure.log.${OPENMPI_INSTALL_LABEL}

mv config.log config.log.${OPENMPI_INSTALL_LABEL}

make -j 8
make -j 8 install
make -j 8 clean
```
