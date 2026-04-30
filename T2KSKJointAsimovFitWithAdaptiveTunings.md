# Notes on Adaptive Step Size Tuning for T2K-SK Joint Asimov Fit with Shifting and Smearing Detector Uncertainties 
The joint analysis of T2K and SK experiments could possibly utilize a common set of detector uncertainty systematics for the samples that cover similar neutrino energy spectrum and have similar event selection cuts, which consists of parameters linearly distorting, i.e. shifting and smearing, the corresponding event selection variables. These common shifting and smearing parameters form an optional detector uncertainty model together with T2K-sample-specific and SK-sample-specific detector uncertainties. The introduction of the optional model to the joint analysis would cause difficulty in chain convergence rate if those additional parameters' step sizes are not properly tuned. For convenience, the feature of auto-adaptive tuning on step sizes has been implemented for those parameters specifically in the framework of MaCh3, [https://github.com/t2k-software/MaCh3/tree/DBarrow_JointFit_AtmDetSyst](https://github.com/t2k-software/MaCh3/tree/DBarrow_JointFit_AtmDetSyst). This documentation describes some important notes on how the auto-adaption is integrated and utilized in the jobs to generate MCMC samples fit to Asimov dataset as preliminary sensitivity studies to test the feasibility of the optional detector model. 
## General scheme for invloving auto-adaption feature in MCMC
<img width="408" height="273" alt="image" src="https://github.com/user-attachments/assets/ed1cd6db-8aac-4aa6-b36e-681928ef8a40" />

As the example in the above diagram illustrates, the process of producing a single MCMC chain could be broken into three-stages, with the feature of auto-adaptive step size tuning enabled for additional parameters in the optional model. 

The chain at its first two stages is in the phase of enabling auto-adaptive step size tuning. Generally in this phase, the chains start with their initial setting and run with adaptions on their proposal functions via continuing to update the covariance matrix of the parameters every a few thousands steps according to the accumulated samples. This phase will last until the chains reach or get close to their equilibrium states and the last updated covariance matrix are kept for the third stage.

At the third stage, the chain moves into a different phase where the auto-adaption is fully turned off. The chains start at where they are stopped at the previous stage; the adapted covariance matrix in the proposal functions are frozen at fixed values during the whole execution of this stage.

This scheme is realized in the MaCh3 version specified for the T2K-SK joint analysis with shifting and smearing detector uncertainty model [https://github.com/t2k-software/MaCh3/tree/DBarrow_JointFit_AtmDetSyst](https://github.com/t2k-software/MaCh3/tree/DBarrow_JointFit_AtmDetSyst). A corresponding job batch submitter is also prepared for massive production of MCMC chains: [https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC](https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC); some brief instrcutions on this submitter could be found in its repository. 

### Auto-adaption configuration
A template of the configuration card for executing T2K-SK joint Asimov fit in MaCh3 is [https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/main/AtmConfig_adaptiveMC_temp_GlobalSize.cfg](https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/main/AtmConfig_adaptiveMC_temp_GlobalSize.cfg).

The auto-adaption on step sizes are activated for `AtmSKDet` (shifting and smearing parameters), `SKCalibration` (supplementary SK-sample-specified uncertainties) and `SKDetBeam` (supplementary T2K-sample-specified uncertainties) with the following commands in the card:
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
The `ADAPTIVEFILENAME[XXX]` for each group indicates where the adapted covariance matrices are saved as root files; the `ADAPTIVEMATRIXNAME[XXX]` and `ADAPTIVEMEANNAME[XXX]` are the names of the covariance matrix and the mean values of the parameters stored in that root file.

If `USEADAPTIVE[XXX]` is set to `false` then the group will use the default step-size setting in MCMC. In that mode there is no adaption at all in the whole process. 

According to adpative MH algorithm, there should be a modfication to the step size scaling factor for each group, which is related to the size of that group. All the parameters with auto-adaption on step sizes are formally treated as entries of one "global matrix"; in this case, each group forms a sub-block in that global matrix, uncorrelated to the other blocks. The following commands indicate that the step size scaling factors respective for the above groups are modfied by the size of the same global matrix and the size is `263` as `224 (atmskdet) + 21 (skcalib) + 18 (skdetbeam)`.  
```
USEGLOBALMATRIXSIZE = true
GLOBALMATRIXSIZE = 263 
```


## Adaption-on Phase
As introduced above, the adaption on step sizes is turned on for the additional detector uncertainty parameters, i.e. the covaraince matrix used in the proposal function would be constantly updated every a few hundreds or thousands of steps according to the sampled points in the prameter space during that interval. 

In practice, as shown in the scheme diagram, this phase is broken into two stages with different adaption settings.
### Stage 1
The chain starts with initial setting; the adaption on the covariance matrix is firstly made after sampling `1000` points and would then be updated once every `100` steps until the total amount of steps generated with adaption reaches to `100000`, according to the set-up of `Stage 1` job in the batch submitter 
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
  
The purpose of using this setting above is to boost the burn-in stage of MCMC with the help of auto-adaption on the step sizes of the additional detector uncertainty parameters, so hopefully by the end of a `Stage 1` job, a chain could reach or be close to its equilibrium status. The sampling is then moved to `Stage 2` where the adapative covariance matrix would be updated with samples collected from the equilibrium region in the parameter space.  
### Stage 2
As shown in the scheme, at the very begining of `Stage 2` the chain has to "forget" the previous history of adaption because so far the adapted covariance matrix is calculated by steps mostly sampled in the burn-in period and we hope to improve the quality of the adaptive covariance matrix with more steps when the chain is at its equilibrium. So we ***reset*** the adaptive covariance matrix, the mean values of the parameters and the adaption step counter to `0`. This feature is implemented in `JointAtmFit_Asimov` 
- [https://github.com/t2k-software/MaCh3/blob/07e7abdbecafc835698789db87ffeb0dfb8511b1/AtmJointFit_Src/JointAtmFit_Asimov.cpp#L76-L80](https://github.com/t2k-software/MaCh3/blob/07e7abdbecafc835698789db87ffeb0dfb8511b1/AtmJointFit_Src/JointAtmFit_Asimov.cpp#L76-L80):
   ```
       cov->setNumberOfSteps(0);
       cov->useSeparateThrowMatrixReset(adaptiveFileName.c_str(),adaptiveMatrixName.c_str(),adaptiveMeanName.c_str());
   ```
utlizing functions defined in the `covarianceBase` class [https://github.com/t2k-software/MaCh3/blob/DBarrow_JointFit_AtmDetSyst/covariance/covarianceBase.cpp](https://github.com/t2k-software/MaCh3/blob/DBarrow_JointFit_AtmDetSyst/covariance/covarianceBase.cpp).

Besides setting the adaptive covariance matrix, the mean values of the parameters and the adaption step counter to `0`, the final saved adaptive covariance matrix from last stage would be read in to form the initial proposal function at ***reset***. 

The chain would then first accumulate `100000` samples with this proposal function and calculate a new adaptive covariance matrix from those samples; the covariance matrix then is to be updated once every `1000` steps and the auto-adaption stops when there are `200000` steps accumulated with adaption at this stage, according to the following set-up in batch job submitter 
- [https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/0c1c48d5140a4826c37e0efe4a46fa8ee68ff50d/LetsGo_Adaptive_vGlob.py#L388](https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/0c1c48d5140a4826c37e0efe4a46fa8ee68ff50d/LetsGo_Adaptive_vGlob.py#L388):
  ```
    adaptive_setting = {'lower_adapt':10000, 'upper_adapt':200000,'update_interval':1000}
  ```

**NOTE:** `lower_adapt` and `upper_adapat` would be referred to `ADAPTIONTHRESHOLDS` in configuration card (example: [https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/0c1c48d5140a4826c37e0efe4a46fa8ee68ff50d/AtmConfig_adaptiveMC_temp_GlobalSize.cfg#L45](https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/0c1c48d5140a4826c37e0efe4a46fa8ee68ff50d/AtmConfig_adaptiveMC_temp_GlobalSize.cfg#L45)) to control when auto-adaptions happen during MCMC. In MaCh3 ([https://github.com/t2k-software/MaCh3/tree/DBarrow_JointFit_AtmDetSyst](https://github.com/t2k-software/MaCh3/tree/DBarrow_JointFit_AtmDetSyst)), there is also a counter respective for keeping records of the number of steps with adaption, `total_steps`, which is defined in `covarianceBase` class ([covarianceBase](https://github.com/t2k-software/MaCh3/blob/DBarrow_JointFit_AtmDetSyst/covariance/covarianceBase.h)). ***Nota bene***, `total_steps` is not always equal to the full amount of steps of the chain, because after resetting adaption `total_steps` is forced to be `0`, meaning that we basically want to start the adaption from scratch again, while the acutal amount of the steps in that one chain is not zero since there are some steps accumulated from last job and the chain just reassumes running where it stops.      

The reseting of adaption only happens for once at `Stage 2`, specificaly to the chain produced by job of the first iteration at this stage. For that specific job scripit, a flag `RESETADAPTION` in configuration card (example: [https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/0c1c48d5140a4826c37e0efe4a46fa8ee68ff50d/AtmConfig_adaptiveMC_temp_GlobalSize.cfg#L49](https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/0c1c48d5140a4826c37e0efe4a46fa8ee68ff50d/AtmConfig_adaptiveMC_temp_GlobalSize.cfg#L49)) should be set as `true`, which is done by the function setting up `Stage 2` scripts in the job batch submitter as 
- [https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/0c1c48d5140a4826c37e0efe4a46fa8ee68ff50d/LetsGo_Adaptive_vGlob.py#L407](https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/0c1c48d5140a4826c37e0efe4a46fa8ee68ff50d/LetsGo_Adaptive_vGlob.py#L407)
  ```
    SedCommand = "sed -i 's|RESETADAPTION.*|RESETADAPTION = true|' "+configs[ijob][iexe]
  ```

The job to produce a chain at this stage lasts for `2` iterations by default: 
- [https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/0c1c48d5140a4826c37e0efe4a46fa8ee68ff50d/LetsGo_Adaptive_vGlob.py#L386](https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/0c1c48d5140a4826c37e0efe4a46fa8ee68ff50d/LetsGo_Adaptive_vGlob.py#L386)
  ```
    nIterations = 2
  ```
In principal, there could be any numbers of iterations or any amount of steps at this stage, as long as the setting of `lower_adapt`, `upper_adapt`, `NSTEPS` for each iteration and the number of iterations is compatible to each other.

## Adaption-off Phase
We belive the final adaptive covariance matrix by the end of `Stage 2` is optimized with most of the steps used in the calculation of the matrix being sampled at the chain's equilibrium state. Moving on to the next stage, the adaptive covariance matrix is frozen at those final values with no more adaptions in `Stage 3`. So in this phase, the chains could be considered as normal MCMC chains and only the steps collected during this phase would be used for analysis later.

### Stage 3
Within the current framework of MaCh3, we set the following in job batch submitter to turn off auto-adaption at this stage: 
- [https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/0c1c48d5140a4826c37e0efe4a46fa8ee68ff50d/LetsGo_Adaptive_vGlob.py#L491](https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/0c1c48d5140a4826c37e0efe4a46fa8ee68ff50d/LetsGo_Adaptive_vGlob.py#L491)
  ```
        adaptive_setting = {'lower_adapt':10000, 'upper_adapt':10001,'update_interval':1000}
  ```
At this stage, the adaption step number counter `total_steps` should be well beyond the set value of `upper_adapt`, so in this version of [MaCh3](https://github.com/t2k-software/MaCh3/tree/DBarrow_JointFit_AtmDetSyst) the covariance matrix would not be updated any more. The other values in this setting is to keep the auto-adaption implementation uniform. 

As the current set-up in the job batch submitter, there are `3` iterations of jobs at this stage:
- [https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/0c1c48d5140a4826c37e0efe4a46fa8ee68ff50d/LetsGo_Adaptive_vGlob.py#L489](https://github.com/mojiastonybrook/MaCh3JobSubmitterForAdaptMC/blob/0c1c48d5140a4826c37e0efe4a46fa8ee68ff50d/LetsGo_Adaptive_vGlob.py#L489)
  ```
     nIterations = 3
  ```
The number of iterations could be set as any values.

## Supporting Functions in MaCh3 
### `JointAtmFit_Asimov`
The setting in configuration card would be referred to the executable in MaCh3. A function using those referred setting values in the executable `JointAtmFit_Asimov` is called `setupAdaptiveCov()`
- [https://github.com/t2k-software/MaCh3/blob/07e7abdbecafc835698789db87ffeb0dfb8511b1/AtmJointFit_Src/JointAtmFit_Asimov.cpp#L63](https://github.com/t2k-software/MaCh3/blob/07e7abdbecafc835698789db87ffeb0dfb8511b1/AtmJointFit_Src/JointAtmFit_Asimov.cpp#L63)

As described above, there are basically three situations in the process of producing MCMC with auto-adaption active:
- `Stage 1`, the chain starts from scratch and with initial values of covaraince matrix in the proposal function; it also begins adaption
- `Stage 2`'s first iteration job, the chain continues from where it has stopped by the end of last job; it takes the adapted covariance matrix from previou job in the proposal function and then reset the memory of adaption to zeros in order to start adaption from scratch again;
- `Stage 2`'s following iteration jobs, the chain continues from where it has stooped by the end of last job and aslo reassumes the adaption    

### `covarianceBase`
Most of the functions related to the auto-adaption feature could be found in the [`covarianceBase`](https://github.com/t2k-software/MaCh3/blob/DBarrow_JointFit_AtmDetSyst/covariance/covarianceBase.cpp) class. Here are a few important functions:

- [useSeparateThrowMatrix](https://github.com/t2k-software/MaCh3/blob/07e7abdbecafc835698789db87ffeb0dfb8511b1/covariance/covarianceBase.cpp#L1195)
- [useSeparateThrowMatrixReset](https://github.com/t2k-software/MaCh3/blob/07e7abdbecafc835698789db87ffeb0dfb8511b1/covariance/covarianceBase.cpp#L1203)
- [useSeparateThrowMatrix()](https://github.com/t2k-software/MaCh3/blob/07e7abdbecafc835698789db87ffeb0dfb8511b1/covariance/covarianceBase.cpp#L1154)
- [updateAdaptiveCovariance](https://github.com/t2k-software/MaCh3/blob/07e7abdbecafc835698789db87ffeb0dfb8511b1/covariance/covarianceBase.cpp#L1252)
- [saveAdaptiveToFile](https://github.com/t2k-software/MaCh3/blob/07e7abdbecafc835698789db87ffeb0dfb8511b1/covariance/covarianceBase.cpp#L1363)
