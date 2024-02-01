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

[Gradcorrect](https://github.com/khanlab/gradcorrect which calls [gradunwarp](https://github.com/kaitj/gradunwarp), adapted from the workflow used in HCP data correction. 

**Result**: BIDS-directory

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


