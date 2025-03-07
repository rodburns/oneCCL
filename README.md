# oneAPI Collective Communications Library (oneCCL) <!-- omit in toc --> <img align="right" width="100" height="100" src="https://spec.oneapi.io/oneapi-logo-white-scaled.jpg">

[Installation](#installation)&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;[Usage](#usage)&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;[Release Notes](https://www.intel.com/content/www/us/en/developer/articles/release-notes/oneapi-collective-communication-library-ccl-release-notes.html)&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;[Documentation](https://oneapi-src.github.io/oneCCL/)&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;[How to Contribute](CONTRIBUTING.md)&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;[License](LICENSE)

oneAPI Collective Communications Library (oneCCL) provides an efficient implementation of communication patterns used in deep learning.

oneCCL is integrated into:
* [Horovod\*](https://github.com/horovod/horovod) (distributed training framework). Refer to [Horovod with oneCCL](https://github.com/horovod/horovod/blob/master/docs/oneccl.rst) for details.
* [PyTorch\*](https://github.com/pytorch/pytorch) (machine learning framework). Refer to [PyTorch bindings for oneCCL](https://github.com/intel/torch-ccl) for details.

oneCCL is part of [oneAPI](https://oneapi.io).

## Table of Contents <!-- omit in toc -->

- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
  - [Launching Example Application](#launching-example-application)
    - [Using external MPI](#using-external-mpi)
  - [Setting workers affinity](#setting-workers-affinity)
    - [Automatic setup](#automatic-setup)
    - [Explicit setup](#explicit-setup)
  - [Using oneCCL package from CMake](#using-oneccl-package-from-cmake)
    - [oneCCLConfig files generation](#onecclconfig-files-generation)
- [Additional Resources](#additional-resources)
  - [Blog Posts](#blog-posts)
  - [Workshop Materials](#workshop-materials)

## Prerequisites 

- Ubuntu* 18
- GNU*: C, C++ 4.8.5 or higher.

Refer to [System Requirements](https://www.intel.com/content/www/us/en/developer/articles/system-requirements/oneapi-collective-communication-library-system-requirements.html) for more details.

### SYCL support <!-- omit in toc -->
Intel(R) oneAPI DPC++/C++ Compiler with Level Zero v1.0 support.

To install Level Zero, refer to the instructions in [Intel(R) Graphics Compute Runtime repository](https://github.com/intel/compute-runtime/releases) or to the [installation guide](https://dgpu-docs.intel.com/installation-guides/index.html) for oneAPI users.

### BF16 support <!-- omit in toc -->

- AVX512F-based implementation requires GCC 4.9 or higher.
- AVX512_BF16-based implementation requires GCC 10.0 or higher and GNU binutils 2.33 or higher.


## Installation

General installation scenario:

```
cd oneccl
mkdir build
cd build
cmake ..
make -j install
```

If you need a clean build, create a new build directory and invoke `cmake` within it.

You can also do the following during installation:
- [Specify installation directory](INSTALL.md#specify-installation-directory)
- [Specify the compiler](INSTALL.md#specify-the-compiler)
- [Specify `SYCL` cross-platform abstraction level](INSTALL.md#specify-sycl-cross-platform-abstraction-level)
- [Specify the build type](INSTALL.md#specify-the-build-type)
- [Enable `make` verbose output](INSTALL.md#enable-make-verbose-output)

## Usage

### Launching Example Application

Use the command:
```bash
$ source <install_dir>/env/setvars.sh
$ mpirun -n 2 <install_dir>/examples/benchmark/benchmark
```

#### Using external mpi

The ccl-bundled-mpi flag in vars.sh can take values "yes" or "no" to control if bundled Intel MPI should be used or not. Current default is "yes", which means that oneCCL temporarily overrides the mpi implementation in use.

In order to suppress the behavior and use user-supplied or system-default mpi use the following command *instead* of sourcing `setvars.sh`:

```bash
$ source <install_dir>/env/vars.sh --ccl-bundled-mpi=no
```

The mpi implementation will not be overridden. Please note that, in this case, user needs to assure the system finds all required mpi-related binaries.

### Setting workers affinity

There are two ways to set worker threads (workers) affinity: [automatically](#setting-affinity-automatically) and [explicitly](#setting-affinity-explicitly).

#### Automatic setup

1. Set the `CCL_WORKER_COUNT` environment variable with the desired number of workers per process.
2. Set the `CCL_WORKER_AFFINITY` environment variable with the value `auto`.

Example:
```
export CCL_WORKER_COUNT=4
export CCL_WORKER_AFFINITY=auto
```
With the variables above, oneCCL will create four workers per process and the pinning will depend from process launcher.

If an application has been launched using `mpirun` that is provided by oneCCL distribution package then workers will be automatically pinned to the last four cores available for the launched process. The exact IDs of CPU cores can be controlled by `mpirun` parameters.

Otherwise, workers will be automatically pinned to the last four cores available on the node.

---

#### Explicit setup

1. Set the `CCL_WORKER_COUNT` environment variable with the desired number of workers per process.
2. Set the `CCL_WORKER_AFFINITY` environment variable with the IDs of cores to pin local workers.

Example:
```
export CCL_WORKER_COUNT=4
export CCL_WORKER_AFFINITY=3,4,5,6
```
With the variables above, oneCCL will create four workers per process and pin them to the cores with the IDs of 3, 4, 5, and 6 respectively.

### Using oneCCL package from CMake

`oneCCLConfig.cmake` and `oneCCLConfigVersion.cmake` are included into oneCCL distribution.

With these files, you can integrate oneCCL into a user project with the [find_package](https://cmake.org/cmake/help/latest/command/find_package.html) command. Successful invocation of `find_package(oneCCL <options>)` creates imported target `oneCCL` that can be passed to the [target_link_libraries](https://cmake.org/cmake/help/latest/command/target_link_libraries.html) command.

For example:

```cmake
project(Foo)
add_executable(foo foo.cpp)

# Search for oneCCL
find_package(oneCCL REQUIRED)

# Connect oneCCL to foo
target_link_libraries(foo oneCCL)
```
#### oneCCLConfig files generation

To generate oneCCLConfig files for oneCCL package, use the provided [`cmake/scripts/config_generation.cmake`](/cmake/scripts/config_generation.cmake) file:

```
cmake [-DOUTPUT_DIR=<output_dir>] -P cmake/script/config_generation.cmake
```

## Additional Resources

### Blog Posts

- [Optimizing DLRM by using PyTorch with oneCCL Backend](https://pytorch.medium.com/optimizing-dlrm-by-using-pytorch-with-oneccl-backend-9f85b8ef6929)
- [Intel MLSL Makes Distributed Training with MXNet Faster](https://medium.com/apache-mxnet/intel-mlsl-makes-distributed-training-with-mxnet-faster-7186ad245e81)

### Workshop Materials

- oneAPI, oneCCL and OFI: Path to Heterogeneous Architecure Programming with Scalable Collective Communications: [recording](https://www.youtube.com/watch?v=ksiZ90EtP98&feature=youtu.be) and [slides](https://www.openfabrics.org/wp-content/uploads/2020-workshop-presentations/502.-OFA-Virtual-Workshop-2020-oneCCL-v5.pdf)

## Contribute <!-- omit in toc -->

See [CONTRIBUTING](CONTRIBUTING.md) for more information.

## License <!-- omit in toc -->

Distributed under the Apache License 2.0 license. See [LICENSE](LICENSE) for more
information.

## Security Policy <!-- omit in toc -->

See [SECURITY](SECURITY.md) for more information.
