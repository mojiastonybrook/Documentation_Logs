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
1. edit the bashrc script in ComputeCanada home directory, adding the following two lines:
```
module load nixpkgs/16.09
module load gcc/4.8.5
```

2. Set up the directory where you want to install MaCh3 and download the MaCh3 package from github/t2ksoftware. If use personal access token of github, the following line servers as an example command to get to github
```
git clone https://TOKEN@github.com/t2k-software/MaCh3.git
```

3. change the directory into the MaCh3 directory and check out `DBarrow_JointFit` branch

4. Source SetMeUp.sh file
install root, cmt, irods, niwgreweight if asked; \\
IRODS settings:
answer "no" to the first few questions until seeing the question "save configuration? ". Answer yes to that question and start the build;
A password to IRODS might be needed and for that go to the T2K website: ` https://t2k.org/asg/oagroup/gadatastorage/index_html`, look for the section of ***iRODS web interface***
But the server from iRODS might be problematic for now, so expect downloading nothing from the running the command.

5. change the content in the `setup_niwgreweight.sh` file to the following to make sure of the access to github when using personal token:
```
git clone https://TOKEN@github.com/t2k-software/NIWGReWeight.git
```
source setup_niwgreweight.sh

6. change the content in the `setup_CUDAProb3.sh` to make sure of the access to github if using personal token:
```
git clone https:/TOKEN@github.com:/dbarrow257/CUDAProb3
```
source setup_CUDAProb3.sh

7. source setup_psyche.sh

8. T2KSKTool. This package is remotely stored on GitLab. First check the ability to get to gitlab and follow the instructions to add public key of the cluster to your personal account on gitlab.
source setup_t2ksktool.sh

9.
```
source setup.sh
```

10.
```
make clean
make
```

## Note
source setup.sh whenever want to run executables
