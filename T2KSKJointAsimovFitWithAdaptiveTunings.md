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

According to adpative MH algorithm, there should be a modfication to the step size scaling factor for each group, which is related to the size of that group. All the parameters with auto-adapation on step sizes are formally treated as entries of one "global matrix"; in this case, each group forms a sub-block in that global matrix, uncorrelated to the other blocks. The following commands indicate that the step size scaling factors respective for the above groups are modfied by the size of the same global matrix and the size is `263` as `224 (atmskdet) + 21 (skcalib) + 18 (skdetbeam)`.  
```
USEGLOBALMATRIXSIZE = true
GLOBALMATRIXSIZE = 263 
```


## Adapation-on Phase
As introduced above, the adapation on step sizes is turned on for the additional detector uncertainty parameters, i.e. the covaraince matrix used in the proposal function would be constantly updated every a few hundreds or thousands of steps according to the sampled points in the prameter space during that interval. 

In practice, as shown in the scheme diagram, this phase is broken into two stages with different adapation settings.
### Stage 1
The chain starts with initial setting; the adapation on the covariance matrix is firstly made after sampling `1000` points and would then be updated once every `100` steps until the total amount of steps generated with adapation reaches to `100000`, according to the set-up of `Stage 1` job in the batch submitter 
- [https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/0c1c48d5140a4826c37e0efe4a46fa8ee68ff50d/LetsGo_Adaptive_vGlob.py#L376](https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/0c1c48d5140a4826c37e0efe4a46fa8ee68ff50d/LetsGo_Adaptive_vGlob.py#L376):
  ```
    adaptive_setting = {'lower_adapt':1000, 'upper_adapt':100000,'update_interval':100}
  ```

The job to produce a chain at this stage lasts only for `1` interation
- [https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/0c1c48d5140a4826c37e0efe4a46fa8ee68ff50d/LetsGo_Adaptive_vGlob.py#L375](https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/0c1c48d5140a4826c37e0efe4a46fa8ee68ff50d/LetsGo_Adaptive_vGlob.py#L375)
  ```
    nIterations = 1
  ```
  **NOTE**: For simplicity and convenience in practice, the amount of steps of each job interation is forced to be set as `100000`, which is also indicated in the manual of the job batch submitter. This amount is the same for the jobs at all stages. 
  
The purpose of using this setting above is to boost the burn-in stage of MCMC with the help of auto-adapation on the step sizes of the additional detector uncertainty parameters, so hopefully by the end of a `Stage 1` job, a chain could reach or be close to its equilibrium status. The sampling is then moved to `Stage 2` where the adapative covariance matrix would be updated with samples collected from the equilibrium region in the parameter space.  
### Stage 2
As shown in the scheme, at the very begining of `Stage 2` the chain has to "forget" the previous history of adapation because so far the adapted covariance matrix is calculated by steps mostly sampled in the burn-in period and we hope to improve the quality of the adaptive covariance matrix with more steps when the chain is at its equilibrium. So we ***reset*** the adaptive covariance matrix, the mean values of the parameters and the adapation step counter to `0`. This feature is implemented in `JointAtmFit_Asimov` 
[https://github.com/t2k-software/MaCh3/blob/07e7abdbecafc835698789db87ffeb0dfb8511b1/AtmJointFit_Src/JointAtmFit_Asimov.cpp#L76-L80](https://github.com/t2k-software/MaCh3/blob/07e7abdbecafc835698789db87ffeb0dfb8511b1/AtmJointFit_Src/JointAtmFit_Asimov.cpp#L76-L80):
-  ```
       cov->setNumberOfSteps(0);
       cov->useSeparateThrowMatrixReset(adaptiveFileName.c_str(),adaptiveMatrixName.c_str(),adaptiveMeanName.c_str());
   ```
utlizing functions defined in the `covarianceBase` class [https://github.com/t2k-software/MaCh3/blob/DBarrow_JointFit_AtmDetSyst/covariance/covarianceBase.cpp](https://github.com/t2k-software/MaCh3/blob/DBarrow_JointFit_AtmDetSyst/covariance/covarianceBase.cpp).

[https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/0c1c48d5140a4826c37e0efe4a46fa8ee68ff50d/LetsGo_Adaptive_vGlob.py#L388](https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/0c1c48d5140a4826c37e0efe4a46fa8ee68ff50d/LetsGo_Adaptive_vGlob.py#L388):
```
    adaptive_setting = {'lower_adapt':10000, 'upper_adapt':200000,'update_interval':1000}
```

   On **Beluga** the environments should be :
```
module load nixpkgs/16.09
module load gcc/4.8.5
module load gsl/1.16
module load cuda/7.5.18
export CUDAPATH=${CUDA_HOME}
```

A password to IRODS might be needed and for that go to the T2K website: ` https://t2k.org/asg/oagroup/gadatastorage/index_html`, look for the section of ***iRODS web interface***. 
But the server of iRODS might be problematic for now, so expect downloading nothing from the running the command.


## Adaption-off Phase
Before running MaCh3 execuables, check if the following necessary inputs exit for MaCh3, otherwise errors would appear during the executable running.

### Stage 3


