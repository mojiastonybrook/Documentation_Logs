# SigmaVariation Study with Low Pion Momentum Dials
## Running SigmaVariation
The source code for the so-called `SigmaVariation` executable in MaCh3 is stored as `AtmJointFit_Src/AtmSigmaVar.cpp` in `DBarrow_JointFit` branch.

Running SigmaVariation needs Monte Carlo samples(mtuple), spline files(weights), covariance matrix files(correlations between systematics) and cfg file(marco script to setup).
- Mtuple
- Spline
- Covariance Matrix
- Cfg

Link the sk atm mtuple files and the spline files in MaCh3 subdirectory `input/skatm`. An exampl of running SigmaVariation on ComputeCanada is shown here.
The default sk atm muple files and corresponding spline files on Cedar of ComputeCanada are stored in 
```
${PROJECT}/rpp-blairt2k/jiangcc/storage_pub/MaCh3_storage/m3_input_mcmc_skatm/SKMtuples_Sept052022
${PROJECT}/rpp-blairt2k/jiangcc/storage_pub/MaCh3_storage/m3_input_mcmc_skatm_spline/SKSplines_Sept052022
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
