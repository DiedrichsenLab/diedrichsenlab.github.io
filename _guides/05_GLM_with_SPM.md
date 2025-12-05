---
layout: page
title: First-level GLM in SPM
usemathjax: true
date: 2023-10-26
category: fMRI
description: Setting up GLM in SPM.
---

## Model setup 
The first step is to set up the GLM - this will process will set up the following fields

*Data*: 
```
% xY.P  - nScan x 1 cell array of file names for the time-series data
% xY.VY - nScan x 1 struct array of image handles (see spm_vol)
%         Images must have the same orientation, voxel size and data type
%       - Any scaling should have already been applied via the image handle
%         scalefactors.
```

*Design matrix*
```
%
% xX    - Structure containing design matrix information
%       - Required fields are:
%         xX.X      - Design matrix (raw, not temporally smoothed)
%         xX.name   - cellstr of parameter names corresponding to columns
%                     of design matrix
%       - Optional fields are:
%         xX.K      - cell of session-specific structures (see spm_filter)
%                   - Design & data are pre-multiplied by K
%                     (K*Y = K*X*beta + K*e)
%                   - Note that K should not smooth across block boundaries
%                   - defaults to speye(size(xX.X,1))
%         xX.W      - Optional whitening/weighting matrix used to give
%                     weighted least squares estimates (WLS). If not
%                     specified spm_spm will set this to whiten the data
%                     and render the OLS estimates maximum likelihood
%                     i.e. W*W' = inv(xVi.V).
```


*Covariance Structure*: 
```
%
% xVi   - Structure describing intrinsic temporal non-sphericity
%       - Required fields are:
%         xVi.Vi    - array of non-sphericity components
%                   - defaults to {speye(size(xX.X,1))} - i.i.d.
%                   - specifying a cell array of constraints (Qi)
%                     These constraints invoke spm_reml to estimate
%                     hyperparameters assuming V is constant over voxels.
%                     that provide a high precise estimate of xX.V
```

*Mask:*
```
% xM    - Structure containing masking information, or a simple column vector
%         of thresholds corresponding to the images in VY [default: -Inf]
%       - If a structure, the required fields are:
%         xM.TH - nVar x nScan matrix of analysis thresholds, one per image
%         xM.I  - Implicit masking (0=>none, 1 => implicit zero/NaN mask)
%         xM.VM - struct array of explicit mask image handles
%       - (empty if no explicit masks)
%               - Explicit mask images are >0 for valid voxels to assess.
%               - Mask images can have any orientation, voxel size or data
%                 type. They are interpolated using nearest neighbour
%                 interpolation to the voxel locations of the data Y.
%       - Note that voxels with constant data (i.e. the same value across
%         scans) are also automatically masked out.
```


*Working directory*
```
% swd   - Directory where the output files will be saved [default: pwd]
%         If exists, it becomes the current working directory.
```


## Covariance estimation (first pass)
The first pass through `spm_spm` runs through the estimation first and tries to establish a estimate of the noise-covariance from the initial residuals using OLS.  
This calls then the function `spm_reml`, wihich uses restricted maximum likelihood to estimate the variance-covariance matrix and adds the following fields to the SPM-structure: 

```
%     xVi.V      - estimated non-sphericity trace(V) = rank(V)
%     xVi.h      - hyperparameters  xVi.V = xVi.h(1)*xVi.Vi{1} + ...
%     xVi.Cy     - spatially whitened <Y*Y'> (used by ReML to estimate h)
```

## Estimation (second pass)
Having obtained the noise-covariance matrix, we can now estimate the parameters (betas) of the model. 
For every voxel in the mask, the parameter estimates are obtained by solving the General-least-squares equation:

$$
\hat{\mathbf{b}} = (\mathbf{X}^T\mathbf{V}^{-1}\mathbf{X})^{-1}\mathbf{X}^T\mathbf{V}^{-1}\mathbf{y}
$$ 

where $$\mathbf{V}$$ is the noise-covariance matrix (xX.V) that includes filtering, weighting, and whitening (K * W * xVi.V * W' * K'). 

Computationally this is actually solved in the following way: 

```matlab:
xX.xKXs = K*W*X             % filtered and whitened design matrix
y_filt = K*W*y              % filtered and whitened data
xX.pKX = spm_pinv(xX.xKXs); % pseudo-inverse of filtered and whitened design matrix
beta = xX.pKX*y_filt          % parameter estimates
```


Overall this process adds the following fields to the SPM-structure: 

```
%     Vbeta     - struct array of beta image handles (relative)
%     VResMS    - file struct of ResMS image handle  (relative)
%     VM        - file struct of Mask  image handle  (relative)
```

and the following image files in the working directory: 

beta_????.nii                                     - parameter images
These are 32-bit (float32) images of the parameter estimates. The image
files are numbered according to the corresponding column of the
design matrix. Voxels outside the analysis mask (mask.<ext>) are given
value NaN.

ResMS.nii                           - estimated residual variance image
This is a 64-bit (float64) image of the residual variance estimate.
Voxels outside the analysis mask are given value NaN.

mask.nii                                          - analysis mask image
8-bit (uint8) image of zero-s & one's indicating which voxels were
included in the analysis. This mask image is the intersection of the
explicit, implicit and threshold masks specified in the xM argument.
The XYZ matrix contains the voxel coordinates of all voxels in the
analysis mask. The mask image is included for reference, but is not
explicitly used by the results section.



It also adds the following fields to the design matrix (xX): 
```
%     xX.W      - if not specified W*W' = inv(x.Vi.V)
%     xX.V      - V matrix (K*W*Vi*W'*K') = correlations after K*W is applied
%     xX.xKXs   - space structure for K*W*X, the 'filtered and whitened'
%                 design matrix
%               - given as spm_sp('Set',xX.K*xX.W*xX.X) - see spm_sp
%     xX.pKX    - pseudoinverse of K*W*X, computed by spm_sp
%     xX.Bcov   - xX.pKX*xX.V*xX.pKX - variance-covariance matrix of
%                 parameter estimates
%                 (when multiplied by the voxel-specific hyperparameter ResMS
%                 of the parameter estimates (ResSS/xX.trRV = ResMS) )
%     xX.trRV   - trace of R*V
%     xX.trRVRV - trace of RVRV
%     xX.erdf   - effective residual degrees of freedom (trRV^2/trRVRV)
%     xX.nKX    - design matrix (xX.xKXs.X) scaled for display
%                 (see spm_DesMtx('sca',... for details)
```

Where `xX.Bcov` is especially important, as it tells you the variance-covariance matrix of the parameter estimates.

spm_spm also adds the following fields for using Gaussian Random Field theory to correct for multiple comparisons: 

``` 
%     xVol.M    - 4x4 voxel->mm transformation matrix
%     xVol.iM   - 4x4 mm->voxel transformation matrix
%     xVol.DIM  - image dimensions - column vector (in voxels)
%     xVol.XYZ  - 3 x S vector of in-mask voxel coordinates
%     xVol.S    - Lebesgue measure or volume       (in voxels)
%     xVol.R    - vector of resel counts           (in resels)
%     xVol.FWHM - Smoothness of components - FWHM, (in voxels)
```

## Evaluating model fit with RWLS toolbox

For instructions and fields, see: 

[https://www.diedrichsenlab.org/imaging/robustWLS.html](https://www.diedrichsenlab.org/imaging/robustWLS.html)

Even if you are not using 'WLS' model, but standard model, the RWLS toobox stores a residual timeseries in your SPM structure: 

```
SPM.ResStats.ss % Sum of squares of residuals - summed over voxels 
SPM.ResStats.n  % Number of voxels 
```

So, to get a plot of the residual noise-std, you can just plot 

```
sqrt(SPM.ResStats.ss/SPM.ResStats.n)
```

which is what `spm_rwls_resstats can` does. 

## Contrast

Every time you specify a contrast, SPM will add a new entry to the SPM.xCon-structure, with SPM.xCon(i) being the i-th contrast. 

```
%     xCon      - Contrast structure (created by spm_FcUtil.m)
%     xCon.name - Name of contrast
%     xCon.STAT - 'F', 'T' or 'P' - for F/T-contrast ('P' for PPMs)
%     xCon.c    - (F) Contrast weights
%     xCon.X0   - Reduced design matrix (spans design space under Ho)
%                 It is in the form of a matrix (spm99b) or the
%                 coordinates of this matrix in the orthogonal basis
%                 of xX.X defined in spm_sp.
%     xCon.iX0  - Indicates how contrast was specified:
%                 If by columns for reduced design matrix then iX0 contains
%                 the column indices. Otherwise, it's a string containing
%                 the spm_FcUtil 'Set' action: Usually one of {'c','c+','X0'}
%                 (Usually this is the input argument F_iX0.)
%     xCon.X1o  - Remaining design space (orthogonal to X0).
%                 It is in the form of the coordinates of this matrix in
%                 the orthogonal basis of xX.X defined in spm_sp.
%     xCon.eidf - Effective interest degrees of freedom (numerator df)
%     xCon.Vcon - ...for handle of contrast/ESS image (empty at this stage)
%     xCon.Vspm - ...for handle of SPM image (empty at this stage)
```

The t-contrast is computed as 

$$
t = \frac{c^T\hat{\beta}}{\sqrt{c^T\hat{\Sigma}_B c \hat{\sigma}^2}}
$$

Where $$\hat{\beta}$$ is the parameter estimate, $$\hat{\Sigma}$$ is the variance-covariance matrix of the parameter estimates (xX.Bcov) and $$\hat{\sigma}^2$$ is the residual variance (xX.trRV/xX.erdf). stored in the ResMS.nii file.

## Optimal contrast weights
Sometimes you have a GLM with pretty extensive design matrix (superset)

$$
\mathbf{y} = \mathbf{X} \boldsymbol{\beta} + \epsilon\\
\hat{\boldsymbol{\beta}}=(\mathbf{X}^{T}\mathbf{X})^{-1}\mathbf{y}
$$

And want to know what estimates / contrast as simpler GLM where the regressors are some linear combinations of the existing regressors.  
Then you are interested in a linear subspace, i.e. the projection on a arbitrary new design matrix: 

$$
\mathbf{Z}=\mathbf{XC}
$$

Then you can simply obtain any new beta estimate by re-weighting the old estimates

$$
\boldsymbol{\gamma}=(\mathbf{C}^T\mathbf{X}^T\mathbf{X}\mathbf{C})^{-1}\mathbf{C}^T\mathbf{X}^T\mathbf{y}\\
=(\mathbf{C}^T\mathbf{X}^T\mathbf{X}\mathbf{C})^{-1}\mathbf{C}^T\mathbf{X}^T\mathbf{X}\hat{\boldsymbol{\beta}}\\
$$

Example: see Jupyter notebook - Mumford-Neuroimage_2011
