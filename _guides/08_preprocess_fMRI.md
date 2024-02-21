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

1&#46; **Move from BIDS**
- Unzip, move and rename T1 anatomical from <code>BIDS</code> to <code>anatomicals/subj_id/&lt;subj_id&gt;_anatomical.nii</code>.
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
- Unzip, move and rename T2 anatomical from <code>BIDS</code> to <code>anatomicals/subj_id/&lt;subj_id&gt;_T2anatomical.nii</code><br>
- Coregister T2 to T1

**DO NOT CHANGE THE ANATOMICAL IMAGE ANYMORE AFTERWARDS!!**

## **Functional pipeline:**
The code snippets are from the <code>template_imana.m</code> on [spmj_tools](https://github.com/DiedrichsenLab/spmj_tools) repository.

1&#46; **Move from BIDS**
- (Optional) Unzip, move and rename fmap phase and magnitude from <code>BIDS<</code>> to <code>fieldmaps/subj_id/sess&lt;sess number&gt;/&lt;subj_id&gt;_magnitude.nii</code> and <code>&lt;subj_id&gt;_phase.nii</code>
- Unzip, move and rename functional runs from <code>BIDS</code> to <code>imaging_data_raw/subj_id/sess&lt;sess_number&gt;/&lt;subj_id&gt;_run_&lt;run_number&gt;.nii</code>

    {% highlight matlab %}template_imana('BIDS:move_unzip_raw_fmap', 'sn', subj_number) 
template_imana('BIDS:move_unzip_raw_func', 'sn', subj_number){% endhighlight %}


2&#46; **(Optional) Make VDM fieldmaps**
- In some projects you may use fieldmaps to unwarp your functional images. Fieldmaps can quantify the deformations caused by inhomogeneities of the magnetic field. When the subject's head is placed inside the scanner bore, the magnetic field of the scanner will get bent. The amount of bending is different in different locations in the brain (e.g. the bending is higher in hippocampus). Using fieldmaps and unwarping we could correct some of the deformations. The Siemens FieldMap sequence (used in CFMM), produces two magnitude images (<code>magnitude1.nii</code> and <code>magnitude2.nii</code>), and a subtracted phase image (<code>phasediff.nii</code>). SPM uses the <code>phasediff.nii</code> and the shorter TE magnitude image (<code>magnitude1.nii</code>) to generate a *voxel displacement map* (VDM) for each functional run. Then in the *realign & unwarp* process, it uses these VDMs to correct the deformations.

{% highlight matlab %}template_imana('FUNC:make_fmap', 'sn', subj_number){% endhighlight %}

- **Why do you use unwarping to correct the deformations?** When subjects move during data acquisition, the interaction of *the magnetic distortions* and *subject's movements* introduces an unwanted variance (noise) to the BOLD signal. The *realign & unwarp* process, removes partially this unwanted variance.

- **Disadvantages:** Unwarping removes some of the *true* variance from BOLD signals. If the reduction of the unwanted variance is low, unwarping will do us no good. For instance, when there is little movement in our data (<1mm <1deg), we may avoid this step. On the other hand, when focusing on particularly susceptible areas such as Frontal pole, Orbito-frontal cortext, Medial temporal lobe (especially hippocampus), unwarping can dramatically reduce the unwanted variance.

- ref: [SPM guide](https://www.fil.ion.ucl.ac.uk/spm/data/fieldmap/), [UCL mfD course slides](https://www.fil.ion.ucl.ac.uk/mfd_archive/2015/page1/2015-16/RealignmentUnwarp.pptx)

3&#46; **Realignment (& Unwarping)**

- Run the SPM12 <code>spm_realign()</code> (or <code>spm_realign_unwarp()</code>) script for motion correction. *Realign* process runs a rigid transformation (3 x,y,z translation parameters and 3 rotation parameters; overall 6 params) for every single functional volume to a reference image. This reference image is by default the 1st volume in the first run of the session. Alternatively in SPM, you can choose to first align to the 1st volume of the first run and then align everything to the *mean volume* of all runs. 

{% highlight matlab %}template_imana('FUNC:realign_unwarp', 'sn', subj_number, 
                                      'rtm', reference_image_option){% endhighlight %}

OR

{% highlight matlab %}template_imana('FUNC:realign', 'sn', subj_number, ...
                               'rtm', reference_image_option){% endhighlight %}

- ***'rtm'*** **option:** *rtm* is short for *register to mean*. If this option is set as 0, SPM aligns all the volumes in a session to the 1st volume of the first run. If set as 1, SPM first aligns to the first volume and then aligns to *mean of all volumes of all runs*. 

4&#46; **Move Realigned Images**
- move the realigned (& unwarped) images from <code>imaging_data_raw</code> to <code>imaging_data/sess&lt;sess_number&gt;/&lt;prefix&gt;&lt;subj_id&gt;_run_&lt;run_number&gt;.nii</code>

{% highlight matlab %}template_imana('FUNC:move_realigned_images', 'sn', subj_number, ...
                                             'rtm', reference_image_option, ...
                                             'prefix', prefix_for_functional_files){% endhighlight %}

- ***'prefix'* option:** SPM tends to add prefix to file names after running specific processes. Here if you run <code>spm_realign()</code> on the raw functional images, it will add <code>r</code> as a prefix for the **r**ealigned file. If you run <code>spm_realign_unwarp()</code> SPM will add <code>u</code> as the prefix denoting the files are **u**nwarped and realigned. Depending on what you used in **step 3**, you should set the proper *prefix*.

5&#46; **Mean Image Bias Correction**
- In **step 3** we did realign (& unwarp) process on the functional images of each session. Realign (& unwarp) produces a *mean image* based on *rtm* (refer to step 3) based on the option you use. We will use this *mean image* as a reference to register our functional images to the anatomical image. But to improve the coregistration step, we first to a mean bias correction to this mean image. 
- This step results in a new *mean image* file with prefix <code>b</code> denoting **b**ias corrected image.

{% highlight matlab %}template_imana('FUNC:meanimage_bias_correction', 'sn', subj_number, ...
                                                 'rtm', reference_image_option, ...
                                                 'prefix', prefix_for_functional_files){% endhighlight %}

6&#46; **Coregistration of Funcational To Anatomical**
- The goal here is to register the functional images to the anatomical images within each subject. Doing this step we achieve two things: 1) All functional images of different sessions/days will exist in the same coordinates and are aligned to each other. 2) All your functional data will be aligned to the anatomical brain regions, so you know what area you are looking at. 
- All the functional volumes of each session are aligned to the *mean image* we corrected in **step 5**. Therefore, we just need to find the transformation matrix that registers the *mean image* to the anatomical image. Finally, we can use the estimated transformation matrix to bring all the functional volumes to the anatomical coordinates. 

1. Manually do the functional/anatomical registration to ensure a better registration:
    - Open fsleyes
    - Add anatomical image and b*mean*.nii (bias corrected mean) image to overlay
    - click on the bias corrected mean image in the 'Overlay list' in the bottom left of the fsleyes window. list to highlight it.
    - Open tools -> Nudge
    - Manually adjust b*mean*.nii image to the anatomical by changing the 6 paramters (tranlation xyz and rotation xyz) and Do not change the scales! 
    - When done, click apply and close the tool tab. Then to save the changes, click on the save icon next to the mean image  name in the 'Overlay list' and save the new image by adding 'r' in the beginning of the name: rb*mean*.nii. If you don't set the format to be .nii, fsleyes automatically saves it as a .nii.gz so either set it or gunzip afterwards to make it compatible with SPM.

1. Run the automatic registration code:

{% highlight matlab %}template_imana('FUNC:coreg', 'sn', subj_number, ...
                             'rtm', reference_image_option, ...
                             'prefix', prefix_for_functional_files){% endhighlight %}

- This process only estimates a transformation matrix. It does not do any reslicing or smoothing. The registration is a rigid transformation (3 x,y,z translation parameters and 3 rotation parameters; overall 6 params). 


7&#46; **Replace the Transformation Matrix of Functional Images**




## **Freesurfer pipeline:**