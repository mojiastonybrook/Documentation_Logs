# ROOT Installation Guide
This is a documentation of the installation of `root` to a personal local computer with linux system, for the purpose of developing and testing `nnxssst` for JUNO neutrino oscillation analysis. Instead of using a precompiled binary package of root, this documentation compiles `root` from its source files in order to have full control and configurations of the software.

## Overall Summary
The versions of `root` and `python` are chosen to align with other developer's computing environment for `nnxssst`. The features of `root` are extended to include a comprehensive set of functions based on user's own choice.

### Software Environment

| Component            | Version / Configuration         |
| -------------------- | ------------------------------- |
| **Operating system** | Ubuntu 26.04 LTS (64-bit Linux) |
| **ROOT**             | 6.36.06                         |
| **Python**           | 3.13.7                          |
| **C++ standard**     | C++17                           |
| **Compiler**         | GCC / G++                       |
| **Build interface**  | CMake                           |
| **Build type**       | Release (default option)        |

### Expected ROOT Features

**1. Core Data Analysis**

- ROOT C++ interpreter (Cling) and PyROOT
- TTree, RNTuple, and RDataFrame
- Histograms, graphs, and statistical fitting
- ROOT file I/O and compression

**2. Statistical Analysis and Inference**

- Minuit2: MIGRAD, HESSE, and MINOS
- RooFit, RooStats, and HistFactory
- MathMore, UNU.RAN, and FFTW3
- ROOT unfolding libraries

**3. Machine Learning**

- TMVA classification and regression
- TMVA CPU-based algorithms
- TMVA SOFIE inference code generation
- Python interoperability

**4. Parallel Computing**

- Implicit multithreading (Intel TBB)
- Parallel RDataFrame processing
- OpenMP-enabled Minuit2
- ROOT MPI support
- RooFit multiprocessing with ZeroMQ

**5. Detector Simulation and Scientific Data**

- ROOT geometry and GDML
- FITS and XML data processing
- Interoperability with external Geant4 applications

**6. Visualization**

- ROOT 2D/3D graphics and canvases
- OpenGL and X11 support
- Web-based visualization
- Image export

**7. Remote Data Access**

- XRootD distributed file access
- Davix HTTP/WebDAV access
- HTTPS through OpenSSL

**8. Additional Features**

- Versioned shared libraries
- Bundled PCRE, UNU.RAN, XRootD, ZeroMQ, and cppzmq

---

## Step 1 — Install Dependencies
There are prerequisite packages and libraries to configure, build and run `root`. CERN's website of root lists the contents that are needed by different `root` versions on various platforms, [http://root.cern/install/dependencies/](http://root.cern/install/dependencies/). The ones working with ubuntu are shown [here](https://root.cern/install/dependencies/#ubuntu-and-other-debian-based-distributions). If users choose more options when building `root`, there will be additional requirements as well.   
### 1.1 Dependencies Summary

#### Core Packages

- **Build tools:** `build-essential`, `binutils`, `cmake`, `dpkg-dev`, `git`, `pkgconf`
- **Python:** `python3`, `python3-dev`
- **Core libraries:** `libssl-dev`, `zlib1g-dev`
- **Graphics:** `libx11-dev`, `libxext-dev`, `libxft-dev`, `libxpm-dev`, `libgif-dev`
- **Parallel computing:** `libtbb-dev`
- **Mathematics:** `libvdt-dev`
- **Davix:** `davix-dev` (≥ 0.6.4; **NOTE:** It is required by `root` implicitly set `davix=ON` in cmake configuration which is however not covered by cern's core inventory.)

#### Common Optional Packages

- **Mathematics:** `gfortran`
- **OpenGL and graphics:** `libgl-dev`, `libglu1-mesa-dev`, `libglew-dev`, `libgl2ps-dev`, `libgraphviz-dev`
- **Fonts and images:** `libfreetype-dev`, `libftgl-dev`, `libpng-dev`, `libjpeg-dev`, `libtiff-dev`
- **Compression:** `liblzma-dev`, `liblz4-dev`, `libzstd-dev`, `libxxhash-dev`
- **Scientific data:** `libcfitsio-dev`
- **Database:** `libsqlite3-dev`
- **Network and authentication:** `libcurl4-openssl-dev`, `libkrb5-dev`
- **Utilities:** `libreadline-dev`

#### Additional Requirements for the User Selected Configuration

- **MathMore:** `libgsl-dev`
- **FFTW3:** `libfftw3-dev`
- **XML and unfolding:** `libxml2-dev`
- **TMVA SOFIE:** `libprotobuf-dev`, `protobuf-compiler`
- **MPI:** `libopenmpi-dev`, `openmpi-bin`
- **OpenMP:** Provided by GCC

#### Bundled Dependencies
There packages could be built-in along installing root without manual installations to the ubuntu platform: 
- PCRE
- nlohmann/json
- UNU.RAN
- XRootD
- ZeroMQ
- cppzmq

### 1.2 Installation
Update ubuntu's application list
```bash

sudo apt update
``` 
#### 1. Install required packages
```bash

sudo apt install \
  build-essential \
  binutils \
  cmake \
  dpkg-dev \
  git \
  pkgconf \
  \
  python3 \
  python3-dev \
  \
  libssl-dev \
  zlib1g-dev \
  \
  libx11-dev \
  libxext-dev \
  libxft-dev \
  libxpm-dev \
  libgif-dev \
  \
  libtbb-dev \
  libvdt-dev \
  \
  davix-dev \
  libgsl-dev \
  libfftw3-dev \
  libxml2-dev \
  \
  libprotobuf-dev \
  protobuf-compiler \
  \
  libopenmpi-dev \
  openmpi-bin
```

#### 2. Install optional packages

```bash

sudo apt install \
  gfortran \
  \
  libgl-dev \
  libglu1-mesa-dev \
  libglew-dev \
  libgl2ps-dev \
  libgraphviz-dev \
  \
  libfreetype-dev \
  libftgl-dev \
  libpng-dev \
  libjpeg-dev \
  libtiff-dev \
  \
  liblzma-dev \
  liblz4-dev \
  libzstd-dev \
  libxxhash-dev \
  \
  libcfitsio-dev \
  libsqlite3-dev \
  \
  libcurl4-openssl-dev \
  libkrb5-dev \
  \
  libreadline-dev
```

---

## Step 2 — Set Python
Ubuntu suggests developers use `pyenv` to manage different python versions on one computer, [https://ubuntu.com/developers/docs/howto/python-setup/?utm_source=chatgpt.com#managing-multiple-python-versions-with-pyenv](https://ubuntu.com/developers/docs/howto/python-setup/?utm_source=chatgpt.com#managing-multiple-python-versions-with-pyenv). We then switch to a specific python version to make `root` built against that during compiling. 
### 2.1 Python Environment
- **Version:** Python 3.13.7
- **Version manager:** pyenv
- **modules:** NumPy, pip, iminuit, pandas, uproot, pyroot, cppyy
  
### 2.2 Use `pyenv` to install a different version of python
#### Install and enable `pyenv`
Follow the instruction to install `pyenv`, [https://ubuntu.com/developers/docs/howto/python-setup/?utm_source=chatgpt.com#managing-multiple-python-versions-with-pyenv](https://ubuntu.com/developers/docs/howto/python-setup/?utm_source=chatgpt.com#managing-multiple-python-versions-with-pyenv)
#### Check if all the prerequisites for python have been installed
For ubuntu, an example is provided here: [https://github.com/pyenv/pyenv/wiki#suggested-build-environment](https://github.com/pyenv/pyenv/wiki#suggested-build-environment).
#### Download and install python 3.13.7
Run 
```bash
pyenv install 3.13.7
```

then set `3.13.7` as the global default python version

```bash
pyenv global 3.13.7
```

---

## Step 3 — Configure CMake 
### 3.1 Summary of additional compiling options
The default cmake options of building root 6.36 could be found here: [http://root.cern/install/build_from_source/#all-build-options](http://root.cern/install/build_from_source/#all-build-options). The following summary table thus explicitly states only the options that are set as opposite to their defaults.
#### Bundled Libraries

- `builtin_pcre=ON`
- `builtin_nlohmannjson=ON`
- `builtin_unuran=ON`
- `builtin_xrootd=ON`
- `builtin_zeromq=ON`
- `builtin_cppzmq=ON`

#### Mathematical and Statistical Libraries

- `mathmore=ON`
- `unfold=ON`
- `unuran=ON`
- `fftw3=ON`

#### Machine Learning

- `tmva-sofie=ON`

#### Parallel Computing

- `mpi=ON`
- `minuit2_omp=ON`
- `roofit_multiprocess=ON`

#### Additional Build Options

- `soversion=ON`
- `fail-on-missing=ON`

### 3.2 Download `root` source file
#### create a directory structure 
For example
```
├─ Software/
        ├── root/
        │    ├── src/           # ROOT source code
        │    ├── build/         # CMake build files
        │    └── install/       # Installed ROOT
        │
        └── geant4/             # Future other installations for example

```
#### put the source file into `root/src`
This directory where `root`'s sources are will be referred as `<sourcedir>`.
#### cd into the build directory
The directory for the build will be referred as `<builddir>.` The whole compiling process followed should happen within this very directory.
```bash
cd <builddir>
```
The directory where a built `root` is installed will be referred as `<installdir>`.
### 3.3 CMake Configuration
Config cmake with the following command. Cmake will handle the dependency between the packages and generate corresponding compiling files automatically. 
```bash
# Define directories
export sourcedir=<sourcedir>
export installdir=<installdir>

# Configure ROOT
cmake -DCMAKE_INSTALL_PREFIX=$installdir $sourcedir \
  -DCMAKE_CXX_STANDARD=17 \
  \
  -Dbuiltin_pcre=ON \
  -Dbuiltin_nlohmannjson=ON \
  -Dbuiltin_unuran=ON \
  -Dbuiltin_xrootd=ON \
  -Dbuiltin_cppzmq=ON \
  -Dbuiltin_zeromq=ON \
  \
  -Dmathmore=ON \
  -Dunfold=ON \
  -Dunuran=ON \
  -Dfftw3=ON \
  -Dtmva-sofie=ON \
  -Dsoversion=ON \
  \
  -Dmpi=ON \
  -Dminuit2_omp=ON \
  -Droofit_multiprocess=ON \
  -Dfail-on-missing=ON
```

---

## Step 4 — Compiling and Installing

### 4.1 Compile and install

Run command:
```bash
cmake --build . --target install
```

### 4.2 Source shell script after installation
```bash
# Activate ROOT
source "$installdir/bin/thisroot.sh"
```

### 4.3 Simple checks

```bash
root-config --version
```
and
```bash
root-config --features
```
---

## References

- [CERN ROOT — Dependencies](https://root.cern/install/dependencies/)
- [CERN ROOT — Building from Source](https://root.cern/install/build_from_source/)
- [ROOT 6.36.06 — CMake Build Options](https://github.com/root-project/root/blob/v6-36-06/cmake/modules/RootBuildOptions.cmake)
- [ROOT 6.36.06 — Dependency Detection](https://github.com/root-project/root/blob/v6-36-06/cmake/modules/SearchInstalledSoftware.cmake)
