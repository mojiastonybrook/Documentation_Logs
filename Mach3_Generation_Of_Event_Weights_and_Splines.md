# Guides on Tools with T2KReWeights Libs to Produce Event Weights and Splines for MaCh3

## Installation of T2KReWeights and other tool packages

### List of tools
- T2KReWeights 
 
    the core package for event-reweighting providing libs and parent classes that could be utilized by user-defined softwares to perform reweighting-related tasks.
- NIWGReWeight
    > `AdlerAngle_Dial` implemented as the form like `AdHocPiMom_CC_p_pip`
- GeantReWeight
- neut
    > the dial to tune single matrix element implemented as the form like `ROXXXX` 
- OAGenWeightsApps

    to generate weights using T2KReWeights via the dials implemented in the above packages
- XsecResponse

    to convert weight file into the spline files which are the input of MaCh3
### Installation of T2KReWeight

    
## An Example on Running OAGenWeightsApps
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
    
