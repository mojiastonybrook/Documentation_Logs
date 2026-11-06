# ROOT 6.36.06 Installation Guide — Ubuntu 26.04

## Overall Summary

### Software Environment

| Component            | Version / Configuration         |
| -------------------- | ------------------------------- |
| **Operating system** | Ubuntu 26.04 LTS (64-bit Linux) |
| **ROOT**             | 6.36.06                         |
| **Python**           | 3.13.7 (pyenv)                  |
| **C++ standard**     | C++17                           |
| **Compiler**         | GCC / G++                       |
| **Build interface**  | CMake                           |
| **Build type**       | Release                         |

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

## Sec. 1 — Dependencies Summary

### 1.1 Required Packages

#### Core Dependencies

- **Build tools:** `build-essential`, `binutils`, `cmake`, `dpkg-dev`, `git`, `pkgconf`
- **Python:** `python3`, `python3-dev`
- **Core libraries:** `libssl-dev`, `zlib1g-dev`
- **Graphics:** `libx11-dev`, `libxext-dev`, `libxft-dev`, `libxpm-dev`, `libgif-dev`
- **Parallel computing:** `libtbb-dev`
- **Mathematics:** `libvdt-dev`

#### Additional Requirements for the Selected Configuration

- **Davix:** `davix-dev` (≥ 0.6.4; required for default `davix=ON` with system Davix)
- **MathMore:** `libgsl-dev`
- **FFTW3:** `libfftw3-dev`
- **XML and unfolding:** `libxml2-dev`
- **TMVA SOFIE:** `libprotobuf-dev`, `protobuf-compiler`
- **MPI:** `libopenmpi-dev`, `openmpi-bin`
- **OpenMP:** Provided by GCC

### 1.2 Common Optional Packages

- **Mathematics:** `gfortran`
- **OpenGL and graphics:** `libgl-dev`, `libglu1-mesa-dev`, `libglew-dev`, `libgl2ps-dev`, `libgraphviz-dev`
- **Fonts and images:** `libfreetype-dev`, `libftgl-dev`, `libpng-dev`, `libjpeg-dev`, `libtiff-dev`
- **Compression:** `liblzma-dev`, `liblz4-dev`, `libzstd-dev`, `libxxhash-dev`
- **Scientific data:** `libcfitsio-dev`
- **Database:** `libsqlite3-dev`
- **Network and authentication:** `libcurl4-openssl-dev`, `libkrb5-dev`
- **Utilities:** `libreadline-dev`

### 1.3 Bundled Dependencies

- PCRE
- nlohmann/json
- UNU.RAN
- XRootD
- ZeroMQ
- cppzmq

### 1.4 Required Packages — Installation

```bash
sudo apt update

sudo apt install -y \
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

### 1.5 Common Optional Packages — Installation

```bash
sudo apt install -y \
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

## Sec. 2 — Python Settings

### 2.1 Python Environment

- **Version:** Python 3.13.7
- **Version manager:** pyenv
- **Interpreter:** `~/.pyenv/versions/3.13.7/bin/python3`
- **Dependencies:** NumPy, pip, Python development headers

### 2.2 Python Configuration

```bash
# Select Python
export PYTHON="$HOME/.pyenv/versions/3.13.7/bin/python3"

# Install Python dependencies
"$PYTHON" -m pip install --upgrade pip
"$PYTHON" -m pip install numpy
```

---

## Sec. 3 — CMake Configuration

### 3.1 Bundled Libraries

- `builtin_pcre=ON`
- `builtin_nlohmannjson=ON`
- `builtin_unuran=ON`
- `builtin_xrootd=ON`
- `builtin_zeromq=ON`
- `builtin_cppzmq=ON`

### 3.2 Mathematical and Statistical Libraries

- `mathmore=ON`
- `unfold=ON`
- `unuran=ON`
- `fftw3=ON`

### 3.3 Machine Learning

- `tmva-sofie=ON`

### 3.4 Parallel Computing

- `mpi=ON`
- `minuit2_omp=ON`
- `roofit_multiprocess=ON`

### 3.5 Additional Build Options

- `soversion=ON`
- `fail-on-missing=ON`

### 3.6 CMake Configuration Commands

```bash
# Define directories
export sourcedir="$HOME/software/root/6.36.06/src"
export builddir="$HOME/software/root/6.36.06/build"
export installdir="$HOME/software/root/6.36.06/install"

# Select Python
export PYTHON="$HOME/.pyenv/versions/3.13.7/bin/python3"

# Create directories
mkdir -p "$builddir" "$installdir"

# Configure ROOT
cmake -S "$sourcedir" -B "$builddir" \
  -DCMAKE_INSTALL_PREFIX="$installdir" \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_CXX_STANDARD=17 \
  -DPython3_EXECUTABLE="$PYTHON" \
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

## Sec. 4 — Compiling and Installing

### 4.1 Compilation

- **Command:** `cmake --build`
- **Build directory:** `$builddir`

### 4.2 Installation

- **Command:** `cmake --install`
- **Destination:** `$installdir`
- **Environment setup:** `thisroot.sh`

### 4.3 Commands

```bash
# Compile ROOT
cmake --build "$builddir"

# Install ROOT
cmake --install "$builddir"

# Activate ROOT
source "$installdir/bin/thisroot.sh"
```

---

## References

- [CERN ROOT — Dependencies](https://root.cern/install/dependencies/)
- [CERN ROOT — Building from Source](https://root.cern/install/build_from_source/)
- [ROOT 6.36.06 — CMake Build Options](https://github.com/root-project/root/blob/v6-36-06/cmake/modules/RootBuildOptions.cmake)
- [ROOT 6.36.06 — Dependency Detection](https://github.com/root-project/root/blob/v6-36-06/cmake/modules/SearchInstalledSoftware.cmake)