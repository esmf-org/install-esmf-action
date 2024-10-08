# Install ESMF on a GitHub Runner

This GitHub Action installs ESMF libraries on your GitHub runner. By default,
this action caches/restores ESMF libraries in order to speed up your workflow
runs. If you're familiar with GitHub workflows and ESMF then skip ahead to an
[advanced example](advanced-example.md).

`install-esmf-action` sets up your environment as needed. It adds the ESMF
executables to your PATH, adds ESMF libraries to your LD_LIBRARY_PATH, and
sets ESMFMKFILE and ESMF_ROOT in your environment.

[![Tests](https://github.com/esmf-org/install-esmf-action/actions/workflows/acceptance-test.yml/badge.svg)](https://github.com/esmf-org/install-esmf-action/actions/workflows/acceptance-test.yml)

Here's an example workflow using `install-esmf-action`:

```yaml
jobs:
  example-test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
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
[![Basic Example](https://github.com/esmf-org/install-esmf-action/actions/workflows/example-test.yml/badge.svg)](https://github.com/esmf-org/install-esmf-action/actions/workflows/example-test.yml)

In the example above the latest release of ESMF is installed on and cached for
the `ubuntu-latest` runner. Prior to installing ESMF, NetCDF libraries were
installed using apt-get. Note that NetCDF libraries are outside the scope of
ESMF and they are not cached. Future workflow runs will use the cached
installation of ESMF libraries. The cached installation of ESMF assumes that
dependencies have been installed in locations determined at the time ESMF
was built.

## Runners

`install-esmf-action` is available on `ubuntu-*` and `macos-*` systems.

## Install Directory

By default `install-esmf-action` installs the ESMF package into
`$HOME/.esmf-cache`. This can be changed by setting the
`ESMF_INSTALL_PREFIX` environment variable.

For convience binaries are always installed into `$ESMF_INSTALL_PREFIX/bin`,
libraries are always installed into `$ESMF_INSTALL_PREFIX/lib`, and modules
are always installed into `$ESMF_INSTALL_PREFIX/mod`.

## Building

`install-esmf-action` utilizes two jobs while building.

## Caching

`install-esmf-action` caches libraries as `esmf@<version>-<esmf-cache-key>`.
The `<esmf-cache-key>` is determined by ESMF version, ESMF environment
variables, compiler versions, and external libraries. If multiple
workflows/jobs share the same `<esmf-cache-key>` but this is not desired then
set a unique `build-key`.

Note that ESMF dependencies are not cached by `install-esmf-action`. Follow the
["Caching dependencies to speed up workflows"](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)
instructions to cache ESMF dependencies as needed. You may install ESMF
in a directory of your choice and provide your own caching. If you do
so then please disable caching. If you are using `latest` or `develop`
it is recommended that you use caching provided by `install-esmf-action`.
Note that each time ESMF is updated it will create a new cache, which
will consume your cache quota.

Caches are deleted after 7 days of inactivity.

## Build Configuration

### Input Options
You can configure your ESMF installation as needed using the follwoing
options.

| Option Key      | Description                                 | Default      |
| --------------- | ------------------------------------------- | ------------ |
| `build-key`     | unique build description                    | (blank)      |
| `version`       | version of ESMF library                     | `latest`     |
| `esmpy`         | install esmpy module                        | `false`      |
| `rebuild-check` | check version, compilers, and linker paths  | `standard`   |
| `cache`         | cache ESMF library for future workflow runs | `true`       |

#### build-key
Uniquely describe the ESMF build context in order to prevent `esmf-cache-key`
collisions.

#### version
`install-esmf-action` automatically determines version when version is set to
`latest` or `develop`. The `latest` version will determine the latest ESMF
release. The `develop` version will determine the latest ESMF commit to the
ESMF `develop` branch. Cache misses will be frequent when selecting the
`develop` version and this will consume the cache quota.

#### esmpy
Enabling esmpy installs esmpy using pip and requires ESMF v8.3.0+.
[Setup Python](https://github.com/marketplace/actions/setup-python)
must be called prior to executing `install-esmf-action`.

```yaml
  - name: Setup Python
    uses: actions/setup-python@v5
    with:
      python-version: '3.x'
```

#### rebuild-check
The `standard` rebuild check compares ESMF version, ESMF environment variables,
compiler versions, and external libraries to the installation. If any of these
settings have changed then `install-esmf-action` rebuilds ESMF. The `thorough`
option rebuilds ESMF if any setting has changed in `make info`, including make
and cmake versions. The `quick` rebuild check does not check compiler versions
or external libraries. It is a best practice to set a unique `build-key` when
using the `quick` rebuild check.

#### cache
See [Caching](#caching) for more information.

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
      ESMF_INSTALL_PREFIX: $HOME/software-stack
    with:
      version: v8.6.0
      esmpy: true
      cache: true
```

## Links

[ESMF Website](https://earthsystemmodeling.org)
