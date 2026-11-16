# `nnxssst` Installation at IN2P3 Computing Centre (CC)
This documentation describes basic procedures for installing `nnxssst` ([code source](https://code.ihep.ac.cn/juno-group-b/nnxssst_neutrino_energy_spectrum_fitter)) on CCIN2P3 cluster. It is inspired by the `README` in `nnxssst`'s repository and hacks some scripts from users working with previous versions of `nnxssst` as examples.

## Prerequisites 
- Access to `nnxssst`'s repository at GitLab. It requires an IHEP SSO account, which should also be added to the list of developers of `nnxssst`.
- Request a CCIN2P3 account and sign up to the `juno` group.  

## Installation
It is suggested to create a working directory with user's specific account name in the `/sps/juno` space on CC. Then the packages related to `nnxssst` could be stored in a subdirectory within. For example a file structure like
```
/sps/juno/
├─ ${USER}/
        ├── nnxssst/
        │    ├── nnxssst sources/           # nnxssst source code
        │    ├── jobs/                      # remote job scripts
        │    └── libs/                      # libraries for nnxssst
        │
        └── porjects/             # Future other installations for example

```
by running
```bash
mkdir -p /sps/juno/${USER}/nnxssst

cd /sps/juno/${USER}/nnxssst
```
This directory will be referred as the working directory throughout this documentation, `WORKDIR=/sps/juno/${USER}/nnxssst`.
### 1. Set up the computing environment with Conda

#### Make a local `conda` available in your CC account.
Download `conda`'s source code to your current directory
```bash
curl -L -O https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh
```
Install conda to the working directory
```bash
bash Miniforge3-Linux-x86_64.sh -b -p ${WORKDIR}/conda
```
Activate conda after installation
```bash
source ${WORKDIR}/conda/bin/activate
```
#### Configure the computing environments of `nnxssst` with conda.
The computing environments required by `nnxssst` should be done for once before installing `nnxssst`. We do this configuration with `conda` to create a virtual environment for `nnxssst` with a `.yaml` file that includes the specific requirements. 
```bash
# change to the working directory if not already in 
cd $WORKDIR 

conda env create -f requirements.yaml -name <ENV_NAME>
```
Replace `<ENV_NAME>` with any names you like. The `requirements.yaml` should exist in the working directory; its contents are pasted here:
```
name: nnxssst
channels:
  - conda-forge
  - defaults
dependencies:
  - python=3.12
  - iminuit
  - pyyaml
  - pandas
  - scipy
  - matplotlib
  - jupyter
  - pre-commit
  - numpy
  - uproot
  - root
  - pip
  - pip:
      - dayabay-model
      - pymc
      - arviz
      - pytensor
```
Always activate the new environment before running any codes in `nnxssst` 
```bash
conda activate <ENV_NAME>
```

### 2. Download `nnxssst` from GitLab
Download the package from GitLab repository, along with all of its submodules:
```bash
# change to the working directory if not already in 
cd $WORKDIR

# download nnxssst
git clone --recurse-submodules '<GitLab-clone-URL>' <nnxssst_install_name>
```
You could make a link to `<nnxssst_install_name>` if the name of the `nnxssst` installation directory is different to its default:
```bash
ln -s <nnxsst_install_name> nnxssst_neutrino_energy_spectrum_fitter
```
#### note
if only the main repository is downloaded, to download the missing submodules:
```
git submodule update --init --recursive
```
### 3. Set `nnxssst` environment variables

- Run from the repository root. Replace the example variable names with the names required by `nnxssst`.

```bash
export REPLACE_WITH_PROJECT_ROOT_VARIABLE="$(pwd -P)"
# Add a line for each additional project variable.
# export REPLACE_WITH_VARIABLE_NAME='/path/to/resource'
```

## Example jobs

- Replace the executable paths and arguments with those for your checkout.

```bash
'./<path-to-first-executable>' '<first-example-argument>'
'./<path-to-second-executable>' '<second-example-argument>'
```

