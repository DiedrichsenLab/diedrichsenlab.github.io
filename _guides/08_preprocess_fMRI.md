---
layout: page
title: Preprocessing fMRI data
usemathjax: true
---

## Required packages, software installations

* [Matlab DataFrame toolbox](https://github.com/DiedrichsenLab/dataframe)
* SPM12


## Autobids
Autobids is a pipeline maintained by the Khan lab that transforms the raw imaging data from DICOM (Properiatary Siemens format) to nifti files, renames them in BIDS standard, and applies gradient nonlinearity correction to the images as appropriate (gradcorrect).

[Gradcorrect](https://github.com/khanlab/gradcorrect) which calls [gradunwarp](https://github.com/kaitj/gradunwarp), adapted from the workflow used in HCP data correction. 

# Steps for getting your data into BIDS format:

1. **Study Registration**: 
    - Register your study through the [autobids new study form](https://autobids-uwo.ca/new).
2. **Autobids Autherization**: 
    - Have your PI add 'bidsdump' to the list of users who can access the study's data on the [CFMM dicom server](https://dicom.cfmm.uwo.ca/)
3. **Globus Setup**: 
    - Sign up for a [Globus account](http://app.globus.org/). You can use your UWO credentials to sign up.
    - Install the [Globus Connect Personal](https://www.globus.org/globus-connect-personal) on your computer.
4. **Data Downlaod**: 
    - Once your study is registered with autobids and you collect new data, your study specific pipeline will be applied and the data will be available on Globus for you to download.
    - Login to globus and navigate to the "COLLECTIONS" tab. You should see your BIDS converted data shared with you.
    - "autobids_study_{name}_type_{type}" is the folder that contains the BIDS converted data.
    - Collection type is provided in both "rawdata" (DICOM to Nii converted and renamed to BIDS format) or "derivedata" (DICOM to Nii converted, renamed to BIDS format and gradcorrect applied).

# Questions?
For more information about autobids, please visit the [OSF autobids guide](https://osf.io/k89fh/wiki/autobids/).

If you have any questions, please contact Jason Kai at:
- Email: tkai@uwo.ca

# Anatomical pipeline pre-freesurfer: 

1. **Move from BIDS**
    - Unzip, move and rename T1 anatomical from BIDS to <code>anatomicals/subj_id/&lt;subj_id&gt;_anatomical.nii</code>.
    Use <code style="color:red";>template_imana('BIDS:move_unzip_raw_anat', 'sn', &lt;subj number&gt;)</code>
2. **Reslice LPI**
    - Reslice anatomical image within LPI coordinate systems
3. **Center AC**
    - Set AC in T1 anatomicals
4. **Segmentation and Normalization** 
    - Run the SPM12 batch script for segmentation and normalization
5. **Optional**
    - Unzip, move and rename T2 anatomical from BIDS to <code>anatomicals/subj_id/&lt;subj_id&gt;_T2anatomical.nii</code>
    - Coregister T2 to T1

**Result**: The <code>anatomicals/subj_id/anatomical.nii</code> file is the image that defines individual subject space. 
**DO NOT CHANGE THIS IMAGE ANYMORE AFTERWARDS!!**

# Functional pipeline:
1. **Move from BIDS**
    - (Optional) Unzip, move and rename fmap phase and magnitude from BIDS to <code>fieldmaps/subj_id/sess&lt;sess number&gt;/&lt;subj_id&gt;_magnitude.nii</code> and <code>&lt;subj_id&gt;_phase.nii</code>
    - Unzip, move and rename functional runs from BIDS to <code>imaging_data_raw/subj_id/sess&lt;sess number&gt;/&lt;subj_id&gt;\_run\_&lt;run number&gt;.nii</code>

2. ****

**Result**: The anatomicals/subj_id/anatomical.nii file is the image that defines individual subject space. 


## Freesurfer pipeline 