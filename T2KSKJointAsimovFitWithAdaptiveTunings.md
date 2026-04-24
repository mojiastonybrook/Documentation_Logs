# Notes on Adaptive Step Size Tuning for T2K-SK Joint Asimov Fit with Shifting and Smearing Detector Uncertainties 
The joint analysis of T2K and SK experiments could possibly utilize a common set of detector uncertainty systematics for the samples that cover similar neutrino energy spectrum and have similar event selection cuts, which consists of parameters linearly distorting, i.e. shifting and smearing, the corresponding event selection variables. These common shifting and smearing parameters form an optional detector uncertainty model together with T2K-sample-specific and SK-sample-specific detector uncertainties. The introduction of the optional model to the joint analysis would cause difficulty in chain convergence rate if those additional parameters' step sizes are not properly tuned. For convenience, the feature of auto-adaptive tuning on step sizes has been implemented for those parameters specifically in the framework of MaCh3, [https://github.com/t2k-software/MaCh3/tree/DBarrow_JointFit_AtmDetSyst](https://github.com/t2k-software/MaCh3/tree/DBarrow_JointFit_AtmDetSyst). This documentation describes some important notes on how the auto-adaptation is integrated and utilized in the jobs to generate MCMC samples fit to Asimov dataset as preliminary sensitivity studies to test the feasibility of the optional detector model. 
## General scheme for invloving auto-adapation feature in MCMC
<img width="408" height="273" alt="image" src="https://github.com/user-attachments/assets/ed1cd6db-8aac-4aa6-b36e-681928ef8a40" />

As the example in the above diagram illustrates, the process of producing a single MCMC chain could be broken into three-stages, with the feature of auto-adaptive step size tuning enabled for additional parameters in the optional model. 

The chain at its first two stages is in the phase of enabling auto-adaptive step size tuning. Generally in this phase, the chains start with their initial setting and run with adaptions on their proposal functions via continuing to update the covariance matrix of the parameters every a few thousands steps according to the accumulated samples. This phase will last until the chains reach or get close to their equilibrium states and the last updated covariance matrix are kept for the third stage.

At the third stage, the chain moves into a different phase where the auto-adaption is fully turned off. The chains start at where they are stopped at the previous stage; the adapted covariance matrix in the proposal functions are frozen at fixed values during the whole execution of this stage.

This scheme is realized in the MaCh3 version specified for the T2K-SK joint analysis with shifting and smearing detector uncertainty model [https://github.com/t2k-software/MaCh3/tree/DBarrow_JointFit_AtmDetSyst](https://github.com/t2k-software/MaCh3/tree/DBarrow_JointFit_AtmDetSyst). A corresponding job batch submitter is also prepared for massive production of MCMC chains: [https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC](https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC). 

### Auto-adapation configuration
A template of the configuration card for executing T2K-SK joint Asimov fit in MaCh3 is [https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/main/AtmConfig_adaptiveMC_temp_GlobalSize.cfg](https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/main/AtmConfig_adaptiveMC_temp_GlobalSize.cfg).

The auto-adapation on step sizes are on for `AtmSKDet` (shifting and smearing parameters), `SKCalibration` (supplementary SK-sample-specified uncertainties) and `SKDetBeam` (supplementary T2K-sample-specified uncertainties) with the following commands in the card:
```
// AtmSKDet
USEADAPTIVEATMSKDET = true
ADAPTIVEFILENAMEATMSKDET = ""
ADAPTIVEMATRIXNAMEATMSKDET = "AtmSKDet_covariance"
ADAPTIVEMEANNAMEATMSKDET = "AtmSKDet_means"
ADAPTIVEBLOCKSATMSKDET = []        // no blocking for now

// SKCalibration
USEADAPTIVESKCALIB = true
ADAPTIVEFILENAMESKCALIB = ""
ADAPTIVEMATRIXNAMESKCALIB = "skcalib_covariance"
ADAPTIVEMEANNAMESKCALIB = "skcalib_means"
ADAPTIVEBLOCKSSKCALIB = []

//SKDetBeam
USEADAPTIVESKDETBEAM = true
ADAPTIVEFILENAMESKDETBEAM = ""
ADAPTIVEMATRIXNAMESKDETBEAM = "skdetbeam_covariance"
ADAPTIVEMEANNAMESKDETBEAM = "skdetbeam_means"
ADAPTIVEBLOCKSSKDETBEAM = []
```
The `ADAPTIVEFILENAME` for each group indicates where the adapted covariance matrices are saved as root files; the `ADAPTIVEMATRIXNAME` and `ADAPTIVEMEANNAME` are the names of the covariance matrix and the mean values of the parameters stored in that root file.

If `USEADAPTIVE` is set to `false` then the group will use the default step-size setting in MCMC. In that mode there is no adapation at all in the whole process. 

All the parameters with auto-adapation on step sizes are formally treated as entries of one "global matrix"
```
USEGLOBALMATRIXSIZE = true
GLOBALMATRIXSIZE = 263 
```
## Adapation-on Phase

### Stage 1
[https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/0c1c48d5140a4826c37e0efe4a46fa8ee68ff50d/LetsGo_Adaptive_vGlob.py#L376](https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/0c1c48d5140a4826c37e0efe4a46fa8ee68ff50d/LetsGo_Adaptive_vGlob.py#L376):
```
    adaptive_setting = {'lower_adapt':1000, 'upper_adapt':100000,'update_interval':100}
```

### Stage 2
[https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/0c1c48d5140a4826c37e0efe4a46fa8ee68ff50d/LetsGo_Adaptive_vGlob.py#L388](https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/0c1c48d5140a4826c37e0efe4a46fa8ee68ff50d/LetsGo_Adaptive_vGlob.py#L388):
```
    adaptive_setting = {'lower_adapt':10000, 'upper_adapt':200000,'update_interval':1000}
```

1. Prepare a bashrc script `setup.sh` to set up the computing environments on the cluster. **Cedar** An example for the contents in the script:
```
#!/bin/bash

module load nixpkgs/16.09
module load gcc/5.4.0
module load gsl/1.16

module load root/5.34.36

module load cuda/8.0.44
export CUDAPATH=${CUDA_HOME}
```

   On **Beluga** the environments should be :
```
module load nixpkgs/16.09
module load gcc/4.8.5
module load gsl/1.16
module load cuda/7.5.18
export CUDAPATH=${CUDA_HOME}
```

2. Set up the directory where you want to install MaCh3 and download the MaCh3 package from github/t2ksoftware.
   
    - 2.1 If use personal access token of github, the following line servers as an example command to get to github
      ```
      git clone https://TOKEN@github.com/t2k-software/MaCh3.git
      ``` 
       ### note
       To setup a personal token on Github follow the link [Creating a personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
   
    - 2.2.1 If use ssh key to access the github account, follow the instruction to generate a ssh key and add it to your github account
      [Generating a new SSH key and adding it to the ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent?platform=linux)

    - 2.2.2 After setting up the ssh key, use the command below to download MaCh3 from its github repository
      ```
      git clone git@github.com:t2k-software/MaCh3.git
      ```
   
3. change the directory into the MaCh3 directory and check out `DBarrow_JointFit` branch

4. `source SetMeUp.sh`

    if ask whether to Install root, answer: no;
    
    if ask whether to Install **cmt**, **irods**, and **niwgreweight**, answer: yes;

    if ask whether to Build CMAKE binary, answer: no

    IRODS settings:
    answer "no" to the first few questions until seeing the question "save configuration? ". Answer yes to that question and start the build;
A password to IRODS might be needed and for that go to the T2K website: ` https://t2k.org/asg/oagroup/gadatastorage/index_html`, look for the section of ***iRODS web interface***. 
But the server of iRODS might be problematic for now, so expect downloading nothing from the running the command.

6. **irods** has been used to download near detector data and MC but not anymore. To prepare the ND280 files, link the existing files on Cedar to the directory where the subdirecory **MaCh3** is by   
   ```
   ln -s /project/6008045/skt2kjoint/MaCh3_storage_pub/ND280_DataMC_OA2020/P6Data P6Data
   ln -s /project/6008045/skt2kjoint/MaCh3_storage_pub/ND280_DataMC_OA2020/P6MC P6MC
   ```

7. If **niwgreweight** is not installed successfully in Step 4, then it could be solo compiled by running ` source setup_niwgreweight.sh`.
   - change the content in the `setup_niwgreweight.sh` file to the following to make sure of the access to github when using personal token:
    ```
    git clone https://TOKEN@github.com/t2k-software/NIWGReWeight.git
    ```
   - On **Beluga**: installation might fail because there is no path to root; try reload the `root/5.34.36` module  
      
8. Install **CUDAProbs3** by `source setup_CUDAProb3.sh`
   - change the content in the `setup_CUDAProb.sh` to make sure of the access to github if using personal token:
     ```
     git clone https://TOKEN@github.com:/dbarrow257/CUDAProb3
     ```

9. Install **psyche** by `source setup_psyche.sh`

10. Install **T2KSKTool**. 
 
    - This package is remotely stored on GitLab. First check the ability to get to gitlab https://git.t2k.org/users/sign_in and follow the instructions to add public key of the cluster to your personal account on gitlab.
      ### note
      How to set up the public key for GitLab:
      - in home directory run `ssh-keygen -t ed25519`. Enter the directory and name you want the key file to be saved with. Enter a phrase.
      - if the key file is not stored with the default directory or name, the following steps should be taken to make the key workable:
        - `eval $(ssh-agent -s)` 
        - `ssh-add <directory to private SSH key>` For example `ssh-add /home/mojia/.ssh/id_ed25519_t2kGitLab`. Enter the passphrase if asked
        - create a config file under `.ssh` directory. For example `vim .ssh/config`. In the configuration file write:
          ```
          Host git.t2k.org
            PreferredAuthentications publickey
            IdentityFile ~/.ssh/id_ed25519_t2kGitLab
          ```  
      - Paste the public key to the GitLab account. Look for file ended with `.pub`. On GitLab, select the avatar, then ***Preference*** and then on the left side bar ***SSH Keys***

    - Run installation by `source setup_T2KSKTools.sh`. If on computeCanada clusters, the installation of **t2ksk-common** and **t2ksk-detcovmat** packages need to be fixed by the following extra procedures
      - for **t2ksk-common**, follow the commands in `setup_T2KSKTools.sh` to download the package and checkout the right version.
      - In `CMakeLists.txt`, change the line `list(APPEND CMAKE_MODULE_PATH $ENV{ROOTSYS}/etc/cmake)` to `list(APPEND CMAKE_MODULE_PATH $ENV{ROOTSYS}/etc/root/cmake)`
      - then follow the commands in `setup_T2KSKTool.sh` to generate the compiling script and compile the package
      - for **t2ksk-detcovmat**, do similar change in its `CMakeLists.txt` and then compile after successfully implementing **t2ksk-common** and **eign** packages
    
11. Setup MaCh3 computing environments by `source setup.sh`.
   - Check the contents in the shell script and correct the formats before sourcing.
   - Change the command `module load cuda` to `module load cuda/8.0.44` 

11. Compile the MaCh3 executables by  
    ```
    make clean
    make
    ```
## Adaption-off Phase
Before running MaCh3 execuables, check if the following necessary inputs exit for MaCh3, otherwise errors would appear during the executable running.

### Stage 3
In the **MaCh3** subdirectory `configs/AtmosphericConfigs/`, check if there are `.cfg` files with names such as `AtmSample_X.cfg` where **X** represents the figure from 1 to 19, referring the total 19 samples in the SK atmospheric neutrino. If not, then in that directory run the command:
```
python2 makeConfigs.py
```
to generate these configuration cards from templates.

### SK atmospheric MCs and Splines
The root files containing the SK atm MCs and splines are provided to **MaCh3** in directories with hard-coded names as `MaCh3/inputs/skatm/SKMC` and `MaCh3/inputs/skatm/SKMCSplines`. If they are not existing, link the public files with the names shown by:
```
cd MaCh3/inputs/skatm
ln -s /project/6008045/skt2kjoint/MaCh3_storage_pub/m3_input_mc_skatm_spline SKMCSplines
ln -s /project/6008045/skt2kjoint/MaCh3_storage_pub/m3_input_mc_skatm SKMC
```

### T2K beam MCs and Splines
Similar as the SK atm samples, the MCs and splines for t2k beam samples are supposed to be contained in the directories `MaCh3/inputs/SK_19b_13av7_fitqun20` and `MaCh3/inputs/SK_19b_13av7_splines20`. If not existing, then:
```
cd MaCh3/inputs
ln -s /project/6008045/skt2kjoint/MaCh3_storage_pub/m3_input_mc_t2kbeam_spline SK_19b_13av7_splines20
ln -s /project/6008045/skt2kjoint/MaCh3_storage_pub/m3_input_mc_t2kbeam SK_19b_13av7_fitqun20 
```


## Note
`source setup.sh` whenever want to run executables
