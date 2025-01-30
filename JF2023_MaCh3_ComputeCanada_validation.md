# Validations of T2K-SK Joint Fit with MaCh3 on ComputeCanada Cluster

This documentation describes the pre-fitting validations performed by the version of MaCh3 being used in the T2K-SK joint analysis in 2023. The examples are processed on the ComputeCanada cluster Cedar, including running the executables of `PrintEventRate`, `AtmSigmaVar` and `SystLLHScan`.

-note- all the job scripts to be submitted to a remote node should be put in the `\project` space of ComputeCanada. The `\home` space of ComputeCanada does not allow slurm to acess. 

## Print Event Rate
The event rates of the samples covered by the joint fit are recorded in the tech-note [TN471](http://t2k.org/docs/technotes/471/JointFitFitterValidv1.2)'s table 2 and table 3.
### run script example
To reprodce the results, run the executable `PrintEventRate` in MaCh3's `AtmJointFit_Bin/` with a configuration card in MaCh3's `configs/AtmosphericConfigs/`. 
A job script for example:
```
#!/bin/bash

#SBATCH --time=03:00:00
#SBATCH --mail-user=USEREMAILADDRESS
#SBATCH --mail-type=ALL
#SBATCH --job-name=PrintEventRates_AtmT2K_BANFF_osc
#SBATCH --ntasks=1
#SBATCH --nodes=1
#SBATCH --mem-per-cpu=5G
#SBATCH --cpus-per-task=4
#SBATCH --gres=gpu:v100l:1
#SBATCH --account=rpp-blairt2k

cd /home/mojia/MaCh3_2023/MaCh3
source setup.sh
#export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK                                                                                                                   
export OMP_NUM_THREADS=8
./AtmJointFit_Bin/PrintEventRate configs/AtmosphericConfigs/AtmConfig_PrintEventRate_Banff.cfg
```
There are two general configurations for event rate printing. The setups are defined in a `cfg` card with the default one stored as `configs/AtmosphericConfigs/AtmConfig.cfg` in the local installation directory of MaCh3. 
### BANFF tuning
The setup for Banff tuning in the configuration file:
```
//Covariances

USEGENTUNE = False
USEBANFF = True
```

### Generated tuning
The setup for MC generated tuning in the configuration file:
```
//Covariances

USEGENTUNE = True
USEBANFF = False
```

## Sigma Variation
Cross fit validation is done by comparing the sigma variations' results of different fitters. To compatiate PTheta's binning settings, MaCh3 uses a special setup for the systematic parameters of the correlated far detector group.
To reproduce the results, change the corresponding commands in `configs/AtmosphericConfigs/CorrelatedFDDetSyst.cfg` to the following contents:
```
CorrelatedFDDetFileName = "inputs/skatm/AdriensMatrixConvertedToMaCh3Matrix_76Pars_T2KNuEMomNuMuErec_Updated_AsymEscaleDiagValCorrected.root"
T2KFDDetMatrix_IsNueErec = false
```
The default setup for the correlated far detector systematics is `AdriensMatrixConvertedToMaCh3Matrix_72Pars_T2KNuErec_Updated_AsymEscaleDiagValCorrected.root`.
### run script example
```
#!/bin/bash

#SBATCH --time=10:00:00
#SBATCH --mail-user=USEREMAILADDRESS
#SBATCH --mail-type=ALL
#SBATCH --job-name=AtmSigmaVar_AtmT2K_Generated_osc
#SBATCH --ntasks=1
#SBATCH --nodes=1
#SBATCH --mem-per-cpu=5G
#SBATCH --cpus-per-task=4
#SBATCH --gres=gpu:v100l:1
#SBATCH --account=rpp-blairt2k

source /home/mojia/setup.sh
cd /home/mojia/MaCh3_2023/MaCh3
source setup.sh
#export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK                                                                                                                   
export OMP_NUM_THREADS=8
./AtmJointFit_Bin/AtmSigmaVar configs/AtmosphericConfigs/AtmConfig_AtmSigmaVar_osc.cfg
```

There are two sets of setups in the configuaration card for sigma variation: one using the Asimov A set of oscillation parameters and the other one with unoscillated set of parameters.

## Systematics Likelihood Scan
MaCh3 uses the same configuration of the correlated far detector systematics as the previous section.
To reproduce the results, run with Asimov A set of oscillation parameters in the configuration card.
```
#!/bin/bash

#SBATCH --time=24:00:00
#SBATCH --mail-user=USEREMAILADDRESS
#SBATCH --mail-type=ALL
#SBATCH --job-name=SystLLHScan_AtmT2K_Generated_osc
#SBATCH --ntasks=1
#SBATCH --nodes=1
#SBATCH --mem-per-cpu=5G
#SBATCH --cpus-per-task=4
#SBATCH --gres=gpu:v100l:1
#SBATCH --account=rpp-blairt2k

source /home/mojia/setup.sh
cd /home/mojia/MaCh3_2023/MaCh3
source setup.sh
#export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK                                                                                                                   
export OMP_NUM_THREADS=8
./AtmJointFit_Bin/SystLLHScan configs/AtmosphericConfigs/AtmConfig_SystLLHScan_osc.cfg
```

