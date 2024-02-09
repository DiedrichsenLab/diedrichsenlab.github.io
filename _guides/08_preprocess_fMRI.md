---
layout: page
title: Preprocessing fMRI data
usemathjax: true
---

## Required packages, software installations

* [Matlab DataFrame toolbox](https://github.com/jdiedrichsen/dataframe)
* SPM12


## Autobids
Autobids is a pipeline maintained by the Khan lab that transforms the raw imaging data from DICOM (Properiatary Siemens format) to nifti files, renames them in BIDS standard, and applies gradient nonlinearity correction to the images as appropriate (gradcorrect). 

[Gradcorrect](https://github.com/khanlab/gradcorrect) which calls [gradunwarp](https://github.com/kaitj/gradunwarp), adapted from the workflow used in HCP data correction. 

## Steps for getting your data into BIDS format:

1. **Study Registration**: 
    - Register your study through the [autobids new study form](https://autobids-uwo.ca/new).
2. **Autobids Autherization**: 
    - Have your PI add 'bidsdump' to the list of users who can access the study's data on the [CFMM dicom server](https://dicom.cfmm.uwo.ca/)
3. **Globus Setup**: 
    - Sign up for a [Globus account](http://app.globus.org/). You can use your UWO credentials to sign up.
    - Install the [Globus Connect Personal](https://www.globus.org/globus-connect-personal) on your computer.
4. **Downlaod Data**: 
    - Once your study is registered with autobids and you collect new data, your study specific pipeline will be applied and the data will be available on Globus for you to download.
    - Login to globus and navigate to the "COLLECTIONS" tab. You should see your BIDS converted data shared with you.
    - "autobids_study_{name}_type_{type}" is the folder that contains the BIDS converted data.
    - Collection type is provided in both "rawdata" (DICOM to Nii converted and renamed to BIDS format) or "derivedata" (DICOM to Nii converted, renamed to BIDS format and gradcorrect applied).

For more information about autobids, please visit the [OSF autobids guide](https://osf.io/k89fh/wiki/autobids/).

If you have any questions, please contact Jason Kai at:
- Email: tkai@uwo.ca

## OPTIONAL FOR OLD PIPELINES: Changing naming from BIDS to the shorter names for the lab standard 

**Result**: imaging_data_raw/subj_id/ .... 

with the appropriately named files. 
Skip this ideally. 

## Anatomical pipeline pre-freesurfer: 

* Unzip, move and rename T1 anatomical to anatomicals/subj_id/anatomical.nii
* Set AC in T1 anatomical
* Run the SPM12 batch script for segmentation and normalization. 
* Unzip, move and rename T2 anatomical to anatomicals/subj_id/T2anatomical.nii (optional)
* Coregister T2 to T1 (optional)

**Result**: The anatomicals/subj_id/anatomical.nii file is the image that defines individual subject space. 
**DO NOT CHANGE THIS IMAGE ANYMORE AFTERWARDS!!**

## Freesurfer pipeline 

## 


