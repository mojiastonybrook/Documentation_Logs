# Guides on Tools with T2KReWeights Libs to Produce Event Weights and Splines for MaCh3

## Installation of T2KReWeights and other tool packages

### List of tools
- T2KReWeights 
 
    the core package for event-reweighting providing libs and parent classes that could be utilized by user-defined softwares to perform reweighting-related tasks.
    - branch on Seawulf `feature_SK_SI_dev`

- NIWGReWeight
    - branch on Seawulf `OA2021Tidy`
    > `AdlerAngle_Dial` implemented as the form like `AdHocPiMom_CC_p_pip`
- GeantReWeight
    - branch on Seawulf `feature_PN`
- neut
    - branch on Seawulf `PreOA2021DevelopmentMerge` 
    > the dial to tune single matrix element implemented as the form like `ROXXXX` 
- OAGenWeightsApps
    to generate weights using T2KReWeights via the dials implemented in the above packages
    - branch on Seawulf `DBarrow_JointFit`
- XsecResponse  
    to convert weight file into the spline files which are the input of MaCh3
    - branch on Seawulf `DBarrow_JointFit`
 
In addition, the following might be need as well
- FSIFitter
    - branch on Seawulf `bugfix/shebang`
    
### Installation of T2KReWeight
To use `T2KReWeight`, at least one of `neut`,`NIWGReWeight`and `GeantReWeight` need to be installed first.

Follow the instructions for the packages on their github pages.

### Compilations of executables in OAGenWeightsApps
Based on the branch `DBarrow_JointFit`
- go to the `OAGenWeightsApps` directory
- make a directory `mkdir build` to store the build from codes for first-time compilation.
- go to  `build` directory `cd build` and run cmake for pre-compileing `cmake ../ -DUSE_SK=ON` with the 'sk' configuration
- compile the codes 
    ```
    make 
    make install
    ```
### Compliations of XsecResponse
The branch is `DBarrow_JointFit`
- from the top directory of this package run `make`

The source codes are stored in `src_bin` sub-directory.

## Running OAGenWeightsApps
### Requirements
Make sure of having installed `T2KReWeights` `NIWGReWeight` `neut` and other packages where the dials of interests are implemented
### Using OAGenWeightsApps
- soruce the setup script. The file runs the shell scripts of the tool packages above and `root` to setup the necessary environmental variables. An example of such a setup file is
    ```
    source {PathToRoot}/bin/this.root
    source {PathToNeut}/build/Linux/setup.sh
    source {PathToNiwgReweight}/build/Linux/setup.sh
    source {PathToT2kReweight}/build/Linux/setup.sh
    ...
    ```
- go to the directory of `OAGenWeightsApps`. The branch is `DBarrow_JointFit`. The main source codes live in `app/SK` directory.
   - Make changes to the existing `.cxx` files if you want. Or
   - Create a new `.cxx` file for the app you want to build. This new code should be added in the `CMakeList.txt` to be available for compiling. The corresponding modifications could follow the examples made by the original executables.
-  Compile the source codes to executabes as the instruction of **Compilation of executables in OAGenWeightsApps**.
-  setup the necessary computing environment by sourcing the shell script of `OAGenWeightsApps` whenever to use the executables
    ```
    source {PathToOAGenWeightsApps}/build/Linux/setup.sh
    ```
- run the executable.
    need to provide the Monte Carlo files to the executable. The MCs are stored as `.root` files. Here is an example of the command to run such a executable:
    ```
    {PathToExe}/genWeights_T2KSKAtm_AdlerAngle_MatrixElement -s {PathToMCs}/{fileName}.root -o {PathToOutputDir}/{outputFileName}.root
    ```
    `-s` the source MCs;  
    `-o` output files

## Running XsecResponse
This package is used to convert weights into splines. The splines are stored as `.root` files that could be direct input to `MaCh3`
- Setup the computing environment.
    ```
     export LD_LIBRARY_PATH={PathToXsecResponse}/lib:$LD_LIBRARY_PATH
    ```
- Running XsecResponse needs MC files, its corresponding weight files.
- An example of command to run XsecResponse:
    ```
    ${xsecDir}/bin/make_xsec_response_sk_2019_2d \
                -w ${weightfile_dir}/${filename}_Sample${sampleID}_Channel${channelID}_T2KReWeight_Weights.root \
                -m ${rootfile_dir}/${filename}_Sample${sampleID}_Channel${channelID}.root \
                -s ${sampleID} \
                -o ${output_dir}/${filename}_Sample${sampleID}_Channel${channelID}_XsecResponse_Splines.root
    ```
    `-w` weight files;
    `-m` MC files;
    `-s` sample ID;
    `-o` saved as the output files

## Example: Production of Event Weights by Matrix_Element_Dial
This dial is mianly used for CC1\pi samples. For `SK_atmospheric_FC_sub-GeV_neutrino` samples there are three catagories that are cc1\pi enriched,
- `1 Ring e-like 1 decay` with `ATMPDEventType` or sample ID: `2`, oscillation channel ID: `1` as `nue->nue`
- `1 Ring mu-like 2 decay` with sample ID: `6`, oscillation channel ID: `5` as `numu->numu`
- `2 Ring pi0-like` with sample ID: `7`

A copy of MCs of `SK_atmospheric` samples is store at SeaWulf:
```
/gpfs/projects/McGrewGroup/jjjiang/my_MaCh3/MaCh3/inputs/skatm/SKMC/
```
An example of the names of the MC files is: `sk4_fcmc_tau_pcmc_ummc_fQv4r0_sf_minituple_500yr_Sample2_Channel1.root`

The directory together with the name of a certainty MC file should be fed to `OAGenWeightsApps` executable as the `-s` parameter.

This the example codes to run `OAGenWeightsApps` on SeaWulf interactively:
```
source /gpfs/projects/McGrewGroup/mojia/t2ksoftware/t2kreweight/OAGenWeightsApps/build/Linux/setup.sh
cd ~/scratch/T2KReWeightOutput/
cd ${work_directory}

/gpfs/projects/McGrewGroup/mojia/t2ksoftware/t2kreweight/OAGenWeightsApps/build/Linux/bin/genWeights_T2KSKAtm_AdlerAngle_MatrixElement -s /gpfs/projects/McGrewGroup/jjjiang/my_MaCh3/MaCh3/inputs/skatm/SKMC/sk4_fcmc_tau_pcmc_ummc_fQv4r0_sf_minituple_500yr_Sample2_Channel1.root -o sk4_fcmc_tau_pcmc_ummc_fQv4r0_sf_minituple_500yr_Sample2_Channel1_T2KReWeight_Weights.root &> Sample2_Channel1.log
```
## Example: Conversion of Weights from Mtuples to Splines
The weight files produced from the `OAGenWeightsApps` store weights from various dials as mtuples. These mtuples should be converted into splines if to be used as inputs of MaCh3. The conversion is done by running `XsecResponse`.

The key parts in modifying a source script from this package in order to produce corresponding splines for a respective dial from its mtuples are to add name of the branch/dial into the list of systematics already listed, and then declare which indexed knot is the nominal position.

This is an example to run `XsecResponse` for Low Pion Momentum dials on SeaWulf interactively:
```
export LD_LIBRARY_PATH=/gpfs/projects/McGrewGroup/mojia/t2ksoftware/t2kreweight/XsecResponse/lib:$LD_LIBRARY_PATH

cd /gpfs/projects/McGrewGroup/mojia/t2ksoftware/t2kreweight/XsecResponse

/gpfs/projects/McGrewGroup/mojia/t2ksoftware/t2kreweight/XsecResponse/bin/make_xsec_response_sk_2019_2d_pionMom \
           -w /gpfs/scratch/mojia/T2KReWeightOutput/SKAtmWeights_PionMomDial/correlatedVar_7knots/sk4_fcmc_tau_pcmc_ummc_fQv4r0_sf_minituple_500yr_Sample2_Channel1_T2KReWeight_Weights.root \
           -m /gpfs/projects/McGrewGroup/jjjiang/my_MaCh3/MaCh3/inputs/skatm/SKMC/sk4_fcmc_tau_pcmc_ummc_fQv4r0_sf_minituple_500yr_Sample2_Channel1.root \
           -s 2 \
           -o ${outputDir}/sk4_fcmc_tau_pcmc_ummc_fQv4r0_sf_minituple_500yr_Sample2_Channel1_XsecResponse_Splines.root
```

