# SigmaVariation Study with Low Pion Momentum Dials
## Running SigmaVariation
Source code for the so-called `SigmaVariation` executable in MaCh3 is stored as `AtmJointFit_Src/AtmSigmaVar.cpp` in `DBarrow_JointFit` branch.

Running SigmaVariation needs Monte Carlo samples(mtuple), spline files(weights), covariance matrix files(correlations between systematics) and cfg file(marco script to setup).
- Mtuple
- Spline
- Covariance Matrix
- Cfg

## Implementing Spline from An Additional Dial
The spline file storing the weights from the additional dial `Matrix_Element_Ro` has been produced by `XsecResponse` and `OAGenWeightsApps`.

Modifications should be done to a `.xml` file which defines the covariance matrix of the systematics in order to declare the dial and describe the possible relationship with other uncertainties.

An example of such a `.xml` file could be found here: https://github.com/t2k-software/MaCh3/blob/DBarrow_JointFit/nd280_utils/xsecMatrixMaker/xsec_covariance_2020a_IncAtmosphericModel_v9_NoBeamSIPN.xml
