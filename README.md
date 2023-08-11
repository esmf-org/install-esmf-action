# Install ESMF on a GitHub Runner

This GitHub Action installs ESMF libraries on your GitHub runner. By default,
this action caches/restores ESMF libraries in order to speed up your workflow
runs.

Here's an example workflow using `install-esmf-action`:

```yaml
jobs:
  test-software:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Package Install
      run: |
        sudo apt-get -qq -y update
        sudo apt -qq install libnetcdf-dev libnetcdff-dev
    - name: Install ESMF
      uses: esmf-org/install-esmf-action@v1
    - name: Test ESMF Installation
      run: |
        cat ${ESMFMKFILE}
```

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

## Configuration

You can configure your ESMF installation as needed using the follwoing
options.

| Option Key  | Description                                 | Default      |
| ----------- | ------------------------------------------- | ------------ |
| `version`   | version of ESMF library                     | `latest`     |
| `compiler`  | compiler used for building ESMF             | `gfortran`   |
| `comm`      | library used for communication              | `mpiuni`     |
| `netcdf`    | NetCDF library configuration                | `nc-config`  |
| `cache`     | cache ESMF library for future workflow runs | true         |

### version
`install-esmf-action` automatically determines version when version is set to
`latest` or `develop`. The `latest` version will determine the latest ESMF
release. The `develop` version will determine the latest ESMF commit to the
ESMF `develop` branch. Cache misses will be frequent when selecting the
`develop` version.

### compiler|comm|netcdf
For more information on configuration options see section `Building ESMF` in the
[ESMF User's Guide](https://earthsystemmodeling.org/doc) for your selected
version.

### configuration example
Here's an example workflow step with configuration options:

```yaml
  - name: Install ESMF
    uses: esmf-org/install-esmf-action@v1
    with:
      version: v8.5.0
      compiler: intel
      comm: openmpi
      netcdf: nc-config
      cache: true
```

## Links

[ESMF Website](https://earthsystemmodeling.org)
