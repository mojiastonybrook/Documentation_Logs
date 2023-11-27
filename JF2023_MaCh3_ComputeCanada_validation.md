# Validations of T2K-SK Joint Fit with MaCh3 on ComputeCanada Cluster

This documentation describes the pre-fitting validations performed by the version of MaCh3 being used in the T2K-SK joint analysis in 2023. The examples are processed on the ComputeCanada cluster Cedar, including running the executables of `PrintEventRate`, `AtmSigmaVar` and `SystLLHScan`.

## Print Event Rate
The event rates of the samples covered by the joint fit are recorded in the tech-note [TN471]{http://t2k.org/docs/technotes/471/JointFitFitterValidv1.2}'s table 2 and table 3.

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
There are two general configurations for event rate printing.
### BANFF tuning
config
### Generated tuning
config

## Sigma Variation
Cross fit validation is done by comparing the sigma variations' results of different fitters. To compatiate PTheta's binning settings, MaCh3 uses a special setup for the systematic parameters of the correlated far detector group.
To reproduce the results,

## Systematics Likelihood Scan
MaCh3 uses the same configuration of the correlated far detector systematics as the previous section.
To reproduce the results,
