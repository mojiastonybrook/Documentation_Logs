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

# and add the conda path to framework paths
export PATH=${CONDA_PREFIX}/bin:$PATH
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
Add `nnxssst`'s own libraries to `$PYTHONPATH` so python could find corresponding modules.
```bash
export PYTHONPATH=${workdir}/nnxssst_neutrino_energy_spectrum_fitter/fitter:${PYTHONPATH}
export PYTHONPATH=${workdir}/nnxssst_neutrino_energy_spectrum_fitter/external/dyb_prediction/src:${PYTHONPATH}
```
This could be written into a shell script which should be sourced before running any `nnxssst` codes. 
## Example jobs
At this stage, you could try running some simple examples to check if `nnxssst` works. 
### Interactive example
The simplest example that could be run interactively is **the Standard test for 6-years NMO**:
```bash
# change to the nnxssst installation directory if not already in
cd ${WORKDIR}/nnxssst_neutrino_energy_spectrum_fitter

python nmo_chi2_fit.py -c config/example.yaml -o output/example --false-mo
```
### Example of a single remote job
To launch a remote job to the CC cluster, a job script with descriptions about the parameters of the job is needed.

Here is an example of a job script to run a chi-square fit **for Neutrino 2026 analysis roughly 210 days JUNO + 30 days TAO data**
```
#!/usr/bin/bash
# submit_job_Nu26.sh

# command to run the script:
# sbatch submit_job_Nu26.sh

# sbatch settings
#SBATCH --job-name=nnxssst_%j
#SBATCH --output=nnxssst_%j.log
#SBATCH --account=juno
#SBATCH --partition=htc
#SBATCH --cpus-per-task=1
#SBATCH --mem=2G
#SBATCH --time=0-02:00:00
#SBATCH --licenses=sps

# secure execution
set -eo pipefail # removed -u option since cvmfs fails

# definitions
WORKDIR="/sps/juno/${USER}/nnxssst"
JOBS="${WORKDIR}/jobs"

NNXSSST="${WORKDIR}/nnxssst_neutrino_energy_spectrum_fitter"
CONDADIR="${WORKDIR}/conda"

# output
OUTPUT="${JOBS}/job_${SLURM_JOB_ID}"
mkdir -p "${OUTPUT}"
cp submit_job_Nu26.sh ${OUTPUT}

# settings
source ${CONDADIR}/bin/activate
conda activate nnxssst_conda
export PATH=${CONDA_PREFIX}/bin:$PATH
# Set nnxssst environment variables to PYTHONPATH
source ${WORKDIR}/setup_nnxssst_pro.sh

config="config/26B_16.5_NO.yaml config/TAO_NO.yaml"
inputs="./input/26B/26B_1-4b_16.5.root:fixed_binning/data_rate ./input/26B/TAOInputs_2026_6_15.root:IBD_Spectra/IBD_spectrum_bkg_not_subtracted_FV800"
#toys=""    # "${NNXSSST}/config/toy_parameters.yaml"
#cp ${config} ${toys} ${OUTPUT}

# processing
cd ${NNXSSST}

# running
cmd="python ${NNXSSST}/nmo_chi2_fit.py -c ${config} --input-data ${inputs} -o ${OUTPUT} --hesse --minos DeltaMSq12,DeltaMSq13,sinSqDouble12,sinSqDouble13 --unblind"
echo ${cmd}
exec ${cmd}
```

To submit this job script with `slurm` command:
```bash
sbatch submit_job_Nu26.sh
```

