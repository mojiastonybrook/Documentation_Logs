# `nnxssst` installation on CC

## 1. Set up the computing environment with Conda

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

