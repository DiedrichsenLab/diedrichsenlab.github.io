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

## **participants.tsv:** 
A <code>participants.tsv</code> file must be created for every fMRI project. This file must be stored in the main directory of the project (<code>baseDir</code> in the <code>template_imana.m</code>). It includes some general information about subjects such as *subj_id*, *sex*, etc., and some project-specfic information such as *number of sessions*, *list  of runs in each session*, *name of raw anatomical file*, etc. Check out the example <code>participants.tsv</code> in the [spmj_tools](https://github.com/DiedrichsenLab/spmj_tools) repository. Be careful when using office softwares for editing and saving the <code>participants.tsv</code> for your own projects. Sometimes softwares such as Ubuntu's LibreOffice changes the format of the .tsv file and it cannot be loaded using the <code>dload()</code> function from [dataframe toolbox](https://github.com/DiedrichsenLab/dataframe). I personally have used MS Office and it works fine. Alternatively, create the .tsv files using Pandas in Python or <code>dsave()</code> function in [dataframe toolbox](https://github.com/DiedrichsenLab/dataframe).

## **Anatomical pipeline pre-freesurfer:** 

The code snippets are from the <code>template_imana.m</code> on [spmj_tools](https://github.com/DiedrichsenLab/spmj_tools) repository.

1&#46; **Move from BIDS:**
- Unzip, move and rename T1 anatomical from BIDS to <code>anatomicals/subj_id/&lt;subj_id&gt;_anatomical.nii</code>.
    {% highlight matlab %}template_imana('BIDS:move_unzip_raw_anat', 'sn', subj_number){% endhighlight %}

2&#46; **Reslice LPI**
- Reslice anatomical image within LPI coordinate systems. LPI stands for Left, Posterior, Inferior which means coordinates increase when you go from *Left to Right*, *Posterior to Anterior*, and *Inferior to Superior* in the anatomical image.
    {% highlight matlab %}template_imana('ANAT:reslice_LPI', 'sn', subj_number){% endhighlight %}
3&#46; **Center AC**
- Set AC (*Anterior Commissure*) in T1 anatomicals as [0,0,0] in the anatomical image coordinates. [Here](https://imaging.mrc-cbu.cam.ac.uk/imaging/FindingCommissures) is a nice guide to locate AC in the anatomical image. 
    1. Open the Anatomical image in fsleyes (or similar apps) 
    1. Find the Anterior Commissure by moving around the slices
    1. Put the center of the red cursor (in fsleyes) on AC
    1. Read the slice indices (X,Y,Z)
    1. Insert the X,Y,Z indices in the <code>participants.tsv</code>. The default <code>participants.tsv</code> (found on [spmj_tools](https://github.com/DiedrichsenLab/spmj_tools)) has three columns named locACx, locACy, and locACz where you should insert the values.
    {% highlight matlab %}template_imana('ANAT:centre_AC', 'sn', subj_number){% endhighlight %}

4&#46; **Segmentation and Normalization**
- Run the SPM12 batch script for segmentation and normalization. 
    {% highlight matlab %}template_imana('ANAT:segmentation', 'sn', subj_number){% endhighlight %}
- **Results in** 5 .nii files starting with c1, c2, ..., c5. These are masks separating the anatomical image into 5 different segments such as *grey matter*, *white matter*, *skull*, etc. 

5&#46; **Optional**
- Unzip, move and rename T2 anatomical from BIDS to <code>anatomicals/subj_id/&lt;subj_id&gt;_T2anatomical.nii</code><br>
- Coregister T2 to T1

**DO NOT CHANGE THE ANATOMICAL IMAGE ANYMORE AFTERWARDS!!**

## **Functional pipeline:**
The code snippets are from the <code>template_imana.m</code> on [spmj_tools](https://github.com/DiedrichsenLab/spmj_tools) repository.

1&#46; **Move from BIDS**<br>
    - (Optional) Unzip, move and rename fmap phase and magnitude from BIDS to <code>fieldmaps/subj_id/sess&lt;sess number&gt;/&lt;subj_id&gt;_magnitude.nii</code> and <code>&lt;subj_id&gt;_phase.nii</code>
    - Unzip, move and rename functional runs from BIDS to <code>imaging_data_raw/subj_id/sess&lt;sess number&gt;/&lt;subj_id&gt;\_run\_&lt;run number&gt;.nii</code>

    {% highlight matlab %}template_imana('BIDS:move_unzip_raw_fmap', 'sn', subj_number) 
template_imana('BIDS:move_unzip_raw_func', 'sn', subj_number){% endhighlight %}




## **Freesurfer pipeline:**