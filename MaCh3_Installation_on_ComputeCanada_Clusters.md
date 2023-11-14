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
    - If use personal access token of github, the following line servers as an example command to get to github
      ```
      git clone https://TOKEN@github.com/t2k-software/MaCh3.git
      ```
       ### note
      To setup a personal token on Github follow the link [Creating a personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
   
    - If use ssh key to access the github account, follow the instruction to generate a ssh key and add it to your github account
      [Generating a new SSH key and adding it to the ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent?platform=linux)
   
3. change the directory into the MaCh3 directory and check out `DBarrow_JointFit_CorrelatedFDDetSysts_MinorUpdates` branch

4. `source SetMeUp.sh`

    install root, cmt, irods, niwgreweight if asked;

    IRODS settings:
    answer "no" to the first few questions until seeing the question "save configuration? ". Answer yes to that question and start the build;
A password to IRODS might be needed and for that go to the T2K website: ` https://t2k.org/asg/oagroup/gadatastorage/index_html`, look for the section of ***iRODS web interface***. 
But the server of iRODS might be problematic for now, so expect downloading nothing from the running the command.

5. If **niwgreweight** is not installed successfully in the previous step, then it could be solo compiled by running ` source setup_niwgreweight.sh`.
   - change the content in the `setup_niwgreweight.sh` file to the following to make sure of the access to github when using personal token:
    ```
    git clone https://TOKEN@github.com/t2k-software/NIWGReWeight.git
    ```
6. Install **CUDAProbs3** by `source setup_CUDAProb3.sh`
   - change the content in the `setup_CUDAProb.sh` to make sure of the access to github if using personal token:
     ```
     git clone https://TOKEN@github.com:/dbarrow257/CUDAProb3
     ```

7. Install **psyche** by `source setup_psyche.sh`

8. Install **T2KSKTool**. 
 
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

    - then `source setup_t2ksktool.sh`
    
10. `source setup.sh`

11. 
```
make clean
make
```

## Note
`source setup.sh` whenever want to run executables
