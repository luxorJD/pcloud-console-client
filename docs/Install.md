# Installing pCloud Console Client

## Prerequisites

To build pCloud Console Client you'll need the following requirements:

- [Pthread](https://www.gnu.org/software/pth/): The GNU Portable Threads
- [Fuse](https://github.com/libfuse/libfuse) >= 2.6, < 3.0: Filesystem in Userspace

Also, you'll need the following build tools:

- A C99/C++11 compatible compiler such as
  [GCC](https://gcc.gnu.org),
  [Intel C++ Compiler](https://software.intel.com/content/www/us/en/develop/tools/oneapi/components/dpc-compiler.html),
  [Clang](https://clang.llvm.org) or
  [Apple Clang](https://apps.apple.com/us/app/xcode/id497799835)
- [CMake](https://cmake.org/) >= 3.3
- [GNU Make](https://www.gnu.org/software/make) >= 3.82
- [Conan](https://conan.io/) decentralized package manager with a client-server architecture

**Note:** Some parts of the client use GNU extensions to ISO C99 standard,
thus your compiler should support `-std=gnu99`.

For project dependencies list see `conanfile.txt` bundled with this project.

On Debian and its derivatives you can install the required packages this way:

```sh
$ sudo apt install \
    build-essential \
    cmake \
    fuse \
    libfuse-dev \
    libpthread-stubs0-dev \
    libudev-dev
```
On macOS, you most likely have a bundled with Xcode compiler as well as pthread:

```sh
$ brew install \
    cmake \
    macfuse
```

## Build steps

First you'll need clone the project:

```sh
$ git clone https://github.com/sergeyklay/pcloud-console-client.git
$ cd console-client
```

Then, install project dependencies:

```sh
$ pip install -r requirements.txt
```

Next, initialize project with Conan - this is using the `conanfile.txt`
specifying that Conan should integrate with CMake:

```sh
$ conan install . -if=.build --build=missing
```

This example establishes out-of-source `.build/` folder, so that source folder
is not polluted. For a detailed instruction on how to use and customize conan
please refer [here](https://docs.conan.io/en/latest/getting_started.html).

Next, generate the build files using CMake:

```sh
$ cmake -S . -B .build
```

Finally, build project:

```sh
$ cmake --build .build
```

### Configure flags

To enable any build/run-time feature use CMake flags at configure time.
For example, to enable the feature called `FEATURE` use `-DFEATURE=ON` and to
disable the feature use `-DFEATURE=OFF`. Below are the special flags that are
recognized during the project configuration phase:

| Feature                      | Possible Value                                                      | Description                                                       |
| ---------------------------- |---------------------------------------------------------------------|-------------------------------------------------------------------|
| `PCLSYNC_WARNINGS_AS_ERRORS` | `ON`, `OFF`                                                         | Turn all build warnings into errors.                              |
| `PCLSYNC_SSL_IMPL`           | `mbedtls`, `openssl`, `securetransport`                             | Used SSL implementation.                                          |
| `CMAKE_BUILD_TYPE`           | `Release`, `Debug`, `RelWithDebInfo`, `MinSizeRel`, `Asan`, `Ubsan` | This directory is prepended onto all install directories.         |
| `CMAKE_INSTALL_PREFIX`       | Any path.                                                           | This directory is prepended onto all install directories.         |

#### Switching build modes in CMake

CMake supports different build modes, to allow the stripping of debug
information, or to add coverage information to the binaries.

The following modes are supported:

* **Release:** This generates the default code without any unnecessary
  information in the binary files.
* **Debug:** This generates debug information and disables optimization of the
  code.
* **RelWithDebInfo:** Is the same as **Release**, allowing
  you to have symbol files for debugging.
* **MinSizeRel:** Is the same as **Release**, with its
  optimization configuration just set to minimize size.
* **Asan:** This instruments the code with Address Sanitizer to check for
  memory errors.
* **Ubsan:** This instruments the code with Undefined Behavior Sanitizer to
  check for undefined behavior errors.

To enable any of these build mode use `-DCMAKE_BUILD_TYPE=MODE` at configure
phase, and then specify at build phase, e.g.:

```sh
# Configure client
cmake -S . -B .build -DCMAKE_BUILD_TYPE=Release

# Build client
cmake --build .build --config Release
```

## Install

To install client after the build stage simple use `install` target:

```sh
$ sudo cmake --build .build --target install
```

To install client in a non-standard location you'll need to change the
installation prefix. Use `-DCMAKE_INSTALL_PREFIX=/new/location` to change it
at client configure time as follows:

```sh
$ # Configure build
$ cmake -S . -B .build -DCMAKE_INSTALL_PREFIX=~/.local

# Build client
$ cmake --build .build

# Install client (this will use custom prefix now)
$ cmake --build .build --target install
```