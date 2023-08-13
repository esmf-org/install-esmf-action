# Install ESMF on a GitHub Runner

This GitHub Action installs ESMF libraries on your GitHub runner. By default,
this action caches/restores ESMF libraries in order to speed up your workflow
runs.

[![Tests](https://github.com/esmf-org/install-esmf-action/actions/workflows/acceptance-test.yml/badge.svg)](https://github.com/esmf-org/install-esmf-action/actions/workflows/acceptance-test.yml)

Here's an example workflow using `install-esmf-action`:

```yaml
jobs:
  example-test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Package Install
      run: |
        sudo apt-get -qq -y update
        sudo apt -qq install libnetcdf-dev libnetcdff-dev
    - name: Install ESMF
      uses: esmf-org/install-esmf-action@v1
      env:
        ESMF_NETCDF: nc-config
    - name: Print ESMF Info
      run: |
        cat ${ESMFMKFILE}
```
[![Example](https://github.com/esmf-org/install-esmf-action/actions/workflows/example-test.yml/badge.svg)](https://github.com/esmf-org/install-esmf-action/actions/workflows/example-test.yml)

In the example above the latest release of ESMF is installed on and cached for
the `ubuntu-latest` runner. Prior to installing ESMF, NetCDF libraries were
installed using apt-get. Note that NetCDF libraries are outside the scope of
ESMF and they are not cached. Future workflow runs will use the cached
installation of ESMF libraries. The cached installation of ESMF assumes that
dependencies have been installed in locations determined at the time ESMF
was built.

## Runners

`install-esmf-action` is available on `ubuntu-*` and `macos-*` systems.

## Caching

`install-esmf-action` caches libraries as `esmf@<version>-<esmf-build-hash>`.
The `<esmf-build-hash>` is determined by all ESMF build settings in `make info`.
Note that ESMF dependencies are not cached by `install-esmf-action`. Follow the
["Caching dependencies to speed up workflows"](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)
instructions to cache ESMF dependencies as needed. Caches are deleted after 7
days of inactivity.

## Build Configuration

### Input Options
You can configure your ESMF installation as needed using the follwoing
options.

| Option Key  | Description                                 | Default      |
| ----------- | ------------------------------------------- | ------------ |
| `version`   | version of ESMF library                     | `latest`     |
| `esmpy`     | install esmpy module (requires pip)         | `true`       |
| `cache`     | cache ESMF library for future workflow runs | `true`       |

#### version
`install-esmf-action` automatically determines version when version is set to
`latest` or `develop`. The `latest` version will determine the latest ESMF
release. The `develop` version will determine the latest ESMF commit to the
ESMF `develop` branch. Cache misses will be frequent when selecting the
`develop` version.

### ESMF Environment Variables
ESMF uses environment variables for build configuration. Here are some commonly
used environment variables.

| Variable        | Description                | Examples                               |
| --------------- | -------------------------- | -------------------------------------- |
| `ESMF_COMPILER` | Compiler                   | `gfortran`, `intel`, `nvhpc`           |
| `ESMF_COMM`     | MPI communications library | `mpiuni`, `mpt`, `openmpi`, `intelmpi` |
| `ESMF_NETCDF`   | NetCDF library             | `nc-config`, `split`                   |

For more information on configuration options see section `Building ESMF` in the
[ESMF User's Guide](https://earthsystemmodeling.org/doc) for your selected
version.

### Configuration Example
Here's an example workflow step with configuration options:

```yaml
  - name: Install ESMF
    uses: esmf-org/install-esmf-action@v1
    env:
      ESMF_COMPILER: intel
      ESMF_COMM: openmpi
      ESMF_NETCDF: nc-config
    with:
      version: v8.5.0
      esmpy: false
      cache: true
```

## Links

[ESMF Website](https://earthsystemmodeling.org)
