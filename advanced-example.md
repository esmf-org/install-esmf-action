# Advanced Example

In the following example you'll see how to integrate an ESMF installation into
an existing cache. Prior to installing ESMF this example installs OpenMPI,
HDF5, and NetCDF. Notice that the `Install ESMF` step installs ESMF into the
same software stack directory and disables `cache`. Also, the `Install ESMF`
step does not check `cache-hit`. `install-esmf-action` executes every time the
`build-test` job executes. During execution it compares the installed ESMF
version to the latest ESMF release and installs a newer version if needed.

[![Advanced Example](https://github.com/esmf-org/install-esmf-action/actions/workflows/adv-example-test.yml/badge.svg)](https://github.com/esmf-org/install-esmf-action/actions/workflows/adv-example-test.yml)

```yaml
jobs:
  build-test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Library Directory
      run: |
        export STACK_ROOT=${HOME}/stack
        mkdir -p ${STACK_ROOT}/{include,lib,lib64,bin}
        echo "STACK_ROOT=${STACK_ROOT}" >> $GITHUB_ENV
        export LD_LIBRARY_PATH=${STACK_ROOT}/lib64:${STACK_ROOT}/lib:${LD_LIBRARY_PATH}
        echo "LD_LIBRARY_PATH=${LD_LIBRARY_PATH}" >> $GITHUB_ENV
        echo "CPPFLAGS=-I${STACK_ROOT}/include" >> $GITHUB_ENV
        echo "LDFLAGS=-L${STACK_ROOT}/lib" >> $GITHUB_ENV
        echo "${STACK_ROOT}/bin" >> $GITHUB_PATH
    - name: Cache Libraries
      id: cache-libraries
      uses: actions/cache@v3
      with:
        path: ${{env.STACK_ROOT}}
        key: stack-ubuntu-gfortran-openmpi
    - name: Install OPENMPI
      env:
        CACHE_HIT: ${{steps.cache-libraries.outputs.cache-hit}}
      run: |
        if [[ "$CACHE_HIT" != 'true' ]]; then
          OPENMPI_URL="https://download.open-mpi.org/release/open-mpi/v4.0/openmpi-4.0.3.tar.gz"
          mkdir ${{runner.temp}}/openmpi
          cd ${{runner.temp}}/openmpi
          curl -L $OPENMPI_URL | tar --strip-components=1 -xz
          ./configure --prefix=${{env.STACK_ROOT}}
          make -j 2 install
        fi
        echo "CC=${{env.STACK_ROOT}}/bin/mpicc" >> $GITHUB_ENV
        echo "CXX=${{env.STACK_ROOT}}/bin/mpicxx" >> $GITHUB_ENV
        echo "F77=${{env.STACK_ROOT}}/bin/mpif77" >> $GITHUB_ENV
        echo "F90=${{env.STACK_ROOT}}/bin/mpif90" >> $GITHUB_ENV
        echo "FC=${{env.STACK_ROOT}}/bin/mpifort" >> $GITHUB_ENV
        echo "OPENMPI_ROOT=$STACK_ROOT" >> $GITHUB_ENV
    - name: Install HDF5
      env:
        CACHE_HIT: ${{steps.cache-libraries.outputs.cache-hit}}
      run: |
        if [[ "$CACHE_HIT" != 'true' ]]; then
          HDF5_URL="https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.12/hdf5-1.12.2/src/hdf5-1.12.2.tar.gz"
          mkdir ${{runner.temp}}/hdf5
          cd ${{runner.temp}}/hdf5
          curl -L $HDF5_URL | tar --strip-components=2 -xz
          ./configure --enable-fortran --prefix=${{env.STACK_ROOT}}
          make -j 2 install
        fi
        echo "HDF5_ROOT=${{env.STACK_ROOT}}" >> $GITHUB_ENV
    - name: Install NetCDF-C
      env:
        CACHE_HIT: ${{steps.cache-libraries.outputs.cache-hit}}
      run: |
        if [[ "$CACHE_HIT" != 'true' ]]; then
          NETCDF_URL="https://github.com/Unidata/netcdf-c/archive/v4.9.0.tar.gz"
          mkdir ${{runner.temp}}/netcdf-c
          cd ${{runner.temp}}/netcdf-c
          curl -L $NETCDF_URL | tar --strip-components=1 -xz
          ./configure --enable-netcdf-4 --disable-dap --prefix=${{env.STACK_ROOT}}
          make -j 2 install
        fi
        echo "NETCDF_ROOT=${{env.STACK_ROOT}}" >> $GITHUB_ENV
    - name: Install NetCDF-Fortran
      env:
        CACHE_HIT: ${{steps.cache-libraries.outputs.cache-hit}}
      run: |
        if [[ "$CACHE_HIT" != 'true' ]]; then
          NETCDFF_URL="https://github.com/Unidata/netcdf-fortran/archive/v4.5.4.tar.gz"
          mkdir ${{runner.temp}}/netcdf-fortran
          cd ${{runner.temp}}/netcdf-fortran
          curl -L $NETCDFF_URL | tar --strip-components=1 -xz
          ./configure --prefix=${{env.STACK_ROOT}}
          make -j 2 install
        fi
        echo "NETCDFF_ROOT=${{env.STACK_ROOT}}" >> $GITHUB_ENV
    - name: Install ESMF
      uses: esmf-org/install-esmf-action@v1
      env:
        ESMF_COMPILER: gfortran
        ESMF_COMM: openmpi
        ESMF_NETCDF: nc-config
        ESMF_INSTALL_PREFIX: ${STACK_ROOT}
      with:
        version: latest
        esmpy: false
        cache: false
    - name: Print ESMF Info
      run: |
        cat ${ESMFMKFILE}
```
