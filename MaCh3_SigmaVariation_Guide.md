# SigmaVariation Study with Low Pion Momentum Dials
## Running SigmaVariation
The source code for the so-called `SigmaVariation` executable in MaCh3 is stored as `AtmJointFit_Src/AtmSigmaVar.cpp` in `DBarrow_JointFit` branch.

Running SigmaVariation needs Monte Carlo samples(mtuple), spline files(weights), covariance matrix files(correlations between systematics) and cfg file(marco script to setup).
- Mtuple
- Spline
- Covariance Matrix
- Cfg

An exampl of running SigmaVariation on ComputeCanada is shown here.

The default sk atm muple files and corresponding spline files on Cedar of ComputeCanada are stored in 
```
${PROJECT}/rpp-blairt2k/jiangcc/storage_pub/MaCh3_storage/m3_input_mcmc_skatm/SKMtuples_Sept052022
${PROJECT}/rpp-blairt2k/jiangcc/storage_pub/MaCh3_storage/m3_input_mcmc_skatm_spline/SKSplines_Sept052022
```
Link these sk atm mtuple files and the spline files in MaCh3 subdirectory `input/skatm` as `SKMC` and `SKMCSplines`. 
```
#from MaCh3 top directory
cd input/skatm
ln -s PATH_TO_DIRECTORY DIRECTORY_NAME
```

In case to use t2k beam mtuple and spline files, those files are stored in 
```
${PROJECT}/rpp-blairt2k/jiangcc/storage_pub/MaCh3_storage/m3_input_mcmc_t2kbeam/T2KMtuples_Sept052022
${PROJECT}/rpp-blairt2k/jiangcc/storage_pub/MaCh3_storage/m3_input_mcmc_t2kbeam_spline/T2KSplines_Sept052022
```
Link the two directories as `SK_19b_13av7_fitqun20` and `SK_19b_13av7_splines20` directories in `input`.

Set up the configuration of sigmaVariation. Change the content in the cfg file `configs/AtmosphericConfigs/AtmConfig.cfg` according to the need.
- `OUTPUTNAME` 
- `OSCPARAM` Oscillation parameter sets, `Asimov A` or `UnOsc`. Use `UnOsc`.
- `ATMPDFS` atm samples 

Some setting in the source code of sigmaVariation also might need change accordingly. In `AtmJointFit_Src/AtmSigmaVar.cpp` with the following setup
```
  bool useT2K = false;

  bool useAtmFlux = false;         // true
  bool useSKCalibration = true;    // true
  bool useAtmSKDet = false;
  bool useBeamSKDet = false;
  bool useSKDetBeam = false;
  bool useATMPDDet = false;         // true
  bool useXsec = true;
  bool skipFlux = true;
```
to enable the cross-section uncertainties only for solo atm samples. Whenever to run an exceutable, do the followings first:
```
# at MaCh top directory
source setup.sh
make clean
make
```

Prepare a remote job script for convenience, here is an example script:
```
#!/bin/bash                                                                                                                                                            

#SBATCH --time=12:00:00                                                                                                                                                
#SBATCH --mail-user=mo.jia@stonybrook.edu                                                                                                                        
#SBATCH --mail-type=END,FAIL                                                                                                                                                
#SBATCH --job-name=AtmSigmaVar                                                                                                     
#SBATCH --mem-per-cpu=4096M                                                                                                                                            
#SBATCH --cpus-per-task=16                                                                                                                                             
#SBATCH --account=rpp-blairt2k                                                                                                                                         

cd /home/mojia/T2KSKJointFit_software/MaCh3
source setup.sh
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
./AtmJointFit_Bin/AtmSigmaVar configs/AtmosphericConfigs/AtmConfig.cfg
```


## Implementing Spline from An Additional Dial
The spline file storing the weights from the additional dial `Matrix_Element_Ro` has been produced by `XsecResponse` and `OAGenWeightsApps`.

Modifications should be done to a `.xml` file which defines the covariance matrix of the systematics in order to declare the dial and describe the possible relationship with other uncertainties.

An example of such a `.xml` file could be found here: https://github.com/t2k-software/MaCh3/blob/DBarrow_JointFit/nd280_utils/xsecMatrixMaker/xsec_covariance_2020a_IncAtmosphericModel_v9_NoBeamSIPN.xml

A counterpart dial on the low pion momentum shape has already been declared in this file as line 338:
```
<parameter name="RS_DelDecay_AdlerAngle_LowEnergy" nom="0" prior="0" lb="-3.0" ub="3.0" error="0.333333" renorm="0"  type="spline" splineind="22" detid="216" sk_spline_name="AdlerAngle" nd_spline_name="AdlerAngle" stepscale="1.0">
    <sk_mode>1 5 6 12</sk_mode>
  </parameter>
```
The meaning of the parameters and the determination of the values are listed as follow:
- name
- nom
- prior
- lb
- ub
- error
- renorm
- type
- splineind
- detid
- sk_spline_name
- nd_spline_name
- stepscale
- sk_model

Once the additional dial has been declared in this `.xml` file, the covariance matrix file could be generated from xml by running a python script: https://github.com/t2k-software/MaCh3/blob/DBarrow_JointFit/nd280_utils/xsecMatrixMaker/makeXSecMatrix.py
