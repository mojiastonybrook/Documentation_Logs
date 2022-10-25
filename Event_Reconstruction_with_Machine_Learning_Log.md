# Water Cherenkov Event Reconstruction with Convolutional Neural Networks

## Run a remote job on a nnhome cluster with GPU
```
sbatch run_train_model.sh
```

the job script should have the following necessary configurations to setup the cluster:
```
#SBATCH --nodelist=fir
#SBATCH --gres=gpu
```
