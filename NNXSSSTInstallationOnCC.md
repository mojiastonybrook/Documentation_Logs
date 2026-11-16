# `nnxssst` Installation at IN2P3 Computing Centre (CC)
This documentation describes basic procedures for installing `nnxssst` ([code source](https://code.ihep.ac.cn/juno-group-b/nnxssst_neutrino_energy_spectrum_fitter)) on CCIN2P3 cluster. It is inspired by the `README` in `nnxssst`'s repository and hacks some scripts from users working with previous versions of `nnxssst` as examples.

## Prerequisites 
- Access to `nnxssst`'s repository at GitLab. It requires an IHEP SSO account, which should also be added to the list of developers of `nnxssst`.
- Request a CCIN2P3 account and sign up to the `juno` group.  

## Installation
It is suggested to create a working directory with user's specific account name in the `/sps/juno` space on CC. Then the packages related to `nnxssst` could be stored in a subdirectory within. For example,


```bash
mkdir -p /sps/juno/${USER}/nnxssst

cd /sps/juno/${USER}/nnxssst
```
### 1. Set up the computing environment with Conda

- Make `conda` available in your CC shell using the site’s setup instructions.
- Replace the bracketed placeholders before running the commands.

```bash
conda create --name nnxssst '<required-package-spec>'
conda activate nnxssst
```

## 2. Download `nnxssst` from GitLab

```bash
git clone '<GitLab-clone-URL>' nnxssst
cd nnxssst
# Optional: check out the tag or commit used for this installation.
# git checkout '<tag-or-commit>'
```

## 3. Set `nnxssst` environment variables

- Run from the repository root. Replace the example variable names with the names required by `nnxssst`.

```bash
export REPLACE_WITH_PROJECT_ROOT_VARIABLE="$(pwd -P)"
# Add a line for each additional project variable.
# export REPLACE_WITH_VARIABLE_NAME='/path/to/resource'
```

## 4. Run example executables

- Replace the executable paths and arguments with those for your checkout.

```bash
'./<path-to-first-executable>' '<first-example-argument>'
'./<path-to-second-executable>' '<second-example-argument>'
```

