# How to Install MaCh3 on ComputeCanada's Clusters
## Application for a ComputeCanda Account
 1. Go to ComputeCanda's website ` https://alliancecan.ca/en/services/advanced-research-computing/account-management/apply-account`for application and fellow the instruction there.
 Choose “External Collaborator” for “Position”.

 2. A sponsor is needed for the confirmation of the application and his/her CCRI code is required during the application. The current sponsor is Blair Jamieson, bl.jamieson@uwinnipeg.ca, CCRI : `imn-664-02`

 3. Once getting the account, use ssh to connect to the clusters
 ```
 ssh -Y username@clustername.computecanada.ca
 ```
## Install MaCh3
1. Prepare a bashrc script `setup.sh` to set up the computing environments on the cluster. An example for the contents in the script:
```
#!/bin/bash

module load nixpkgs/16.09
module load gcc/5.4.0
module load gsl/1.16

module load root/5.34.36

module load cuda/8.0.44
export CUDAPATH=${CUDA_HOME}
```

2. Set up the directory where you want to install MaCh3 and download the MaCh3 package from github/t2ksoftware.
    -2.1 If use personal access token of github, the following line servers as an example command to get to github
      ```
      git clone https://TOKEN@github.com/t2k-software/MaCh3.git
      ```
       ### note

       To setup a personal token on Github follow the link [Creating a personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
   
    -2.2.1 If use ssh key to access the github account, follow the instruction to generate a ssh key and add it to your github account
      [Generating a new SSH key and adding it to the ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent?platform=linux)

    -2.2.2 After setting up the ssh key, use the command below to download MaCh3 from its github repository
      ```
      git clone git@github.com:t2k-software/MaCh3.git
      ```
   
4. change the directory into the MaCh3 directory and check out `DBarrow_JointFit` branch

5. `source SetMeUp.sh`

    if ask whether to install root, answer: no;
    
    if ask whether to install **cmt**, **irods**, and **niwgreweight**, answer: yes;

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
## Prerequisites inputs for MaCh3
Before running MaCh3 execuables, check if the following necessary inputs exit for MaCh3, otherwise errors would appear during the executable running.

### Configuration cards for separate SK atmospheric samples
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
