---
layout: page
title: DICOM to Nifti conversion using autobids routines
---

## Method 1: Manual conversion of data

# The CFMM DICOM Server

MRI datasets acquired at CFMM are stored in the CFMM DICOM server. To download the data, go to [CFMM Study Managment](https://dicom.cfmm.uwo.ca/), then click on the link to "DICOM Data Browser". To access the data of a specific project, type PRINCIPAL^PROJECT-ID (e.g., DIEDRICHSEN^SMP) in the "Study Description" search field, then click Submit. You will see the list of datasets (i.e., participants, sessions...) collected for your study. Each dataset is identified by a Patient's Name with a standard format YYYY_MM_DD_PROJECT-ID_SUBJ-ID (e.g., 2024_03_13_SMP1_100). In each dataset, MRI data are stored in the DICOM format (.dcm). **You don't need to download the datasets from the CFMM DICOM Server to perform the conversion to Nifti (.nii).

# First step: *cfmm2tar*

DICOM to Nifti conversion involves two steps (*cfmm2tar* and *tar2bids*). Both can be run from the terminal of the CBS server. 

1) Log in into the CBS server. See [this guide](https://osf.io/k89fh/wiki/Computational%20Core%20Server/) to access the CSB server for the first time.

2) Open the Terminal

3) Run the following command:

```
singularity run /srv/containers/cfmm2tar_v1.0.0.sif <out_dir>
```

`<out_dir>` is the directory where *cfmm2tar* puts the output data. You can use */local/scratch* as an output directory for *cfmm2tar*, but always remember to clean the directory when you are done.

You can add flags to *cfmm2tar* to perform this step on a specific project or dataset. With the flag `-p <PRINCIPAL^PROJECT-ID>` you can input to *cfmm2tar* all the datasets beloning to a specific project:

```
singularity run /srv/containers/cfmm2tar_v1.0.0.sif -p <PRINCIPAL^PROJECT-ID> <out_dir>
```

With the flag `-n <PARTICIPANT-ID>` you can input a specific dataset by using the "Patient's Name" field in the CFMM DICOM Server:

```
singularity run /srv/containers/cfmm2tar_v1.0.0.sif -n <Patient's Name> <out_dir>
```

*cfmm2tar* will create a .tar and a .uid file in the output directory.

# Second step: *tar2bids*

After you have created the .tar and .uid file with *cfmm2tar*, the second step involves creating the BIDS repository with the .nii file. Run the following command in the Terminal:

```
singularity run /srv/containers/tar2bids_v0.2.4.sif -h <project-id>_heuristic.py -o <out_dir> <filename>.tar
```

`<filename>` is the full path of the .tar file created by *cfmm2tar* in the previous step. `<out_dir>` is the directory where tar2bids puts the Nifti files. One possibility is to set */local/scratch/BIDS* as `<out_dir>`. `<project-id>_heuristic.py` is a Python script with the information needed to convert functional runs from DICOM to Nifti. Usually, `<project-id>_heuristic.py` is located in */cifs/diedrichsen/data/your_data_directory*. If your project is new and you don't have a heuristic file yet, see the next paragraph to learn how to create one. 

# Heuristic files

This is an example code for a heuristic file:

```
import os
import numpy
from cfmm_base import infotodict as cfmminfodict
from cfmm_base import create_key


def infotodict(seqinfo):
    """Heuristic evaluator for determining which runs belong where
    allowed template fields - follow python string module:
    item: index within category
    subject: participant id
    seqitem: run number during scanning
    subindex: sub index within group
    """

    info = cfmminfodict(seqinfo)

    task = create_key('{bids_subject_session_dir}/func/{bids_subject_session_prefix}_task-task_run-{item:02d}_bold')
    task_sbref = create_key(
        '{bids_subject_session_dir}/func/{bids_subject_session_prefix}_task-task_run-{item:02d}_sbref')

    fmap_sbref = create_key('{bids_subject_session_dir}/fmap/{bids_subject_session_prefix}_dir-{dir}_epi')

    info[task] = []
    info[task_sbref] = []
    info[fmap_sbref] = []

    for idx, s in enumerate(seqinfo):

        if ('1.8iso_AP' in (s.series_description).strip()):
            if (s.dim4 == 1 and 'SBRef' in (s.series_description).strip()):
                info[task_sbref].append({'item': s.series_id})
            elif (s.dim4 > 200):
                info[task].append({'item': s.series_id})

        elif ('1.8iso_PA' in (s.series_description).strip()):
            if (s.dim4 == 1):
                # if 'SBRef' in (s.series_description).strip():
                info[fmap_sbref].append({'item': s.series_id, 'dir': 'PA'})

    return info

```

The variable `s` points to a spreadsheet that you can find in */local/scratch/BIDS/code/tar2bids_YYYY-MM-DD_XXhXXm_ZZZZ/heudiconv/PARTICIPANT-ID/info/dicominfo.tsv*. To create the heuristic file for your project, you need to edit the example above by using the information contained in the spreadsheet. The spreadsheet contains a row for each .dcm file included in the dataset. Each column contains information about .dcm files some. For example, in the case of the project to which the above heuristics belong, the column "series_description" contains the substring "1.8iso_AP" in functional runs, in which the column "dim4" contains a number >200 (which corresponds to the number of volumes collected in each functional run). You can create the spreadsheet by running tar2bids without the -h flag on the first dataset you collect:

```
singularity run /srv/containers/tar2bids_v0.2.4.sif -o <out_dir> <filename>.tar
```

This will create the *dicominfo.tsv* spreadsheet inside `<out_dir>` in the path specified above.

# Gradcorrect (highly recommended for 7T scans)

After you have a BIDS repository with the Nifti files, run the following command:

```
singularity run /srv/containers/khanlab_gradcorrect_v0.0.3a.sif <bids_dir> <out_dir> participant --grad_coeff_file /srv/software/gradcorrect/coeff_AC84.grad
```

`<bids_dir>` is the directory where your BIDS data that is stored. This directory contains the BIDS data before the gradiant corrections. The `participant` argument must be passed as it is, it is NOT the PARTICIPANT-ID. As `<out_dir>` in *gradcorrect* you can put */local/scratch/BIDS_gradcorrect*. Inside `<out_dir>` *gradcorrect* puts several files and folder, including a sub-XX folder with three folders inside: anat, func and fmaps. These three folders contains the Nifti files output by gradcorrect. `<template_imana.m>` from `<spmj>` needs these three folders to be into */project_directory/BIDS/sub-XX/*. 


## Method 2: automatic conversion through autobids (Currently unavailable)
Autobids is a pipeline maintained by the Khan lab that transforms the raw imaging data from DICOM (Properiatary Siemens format) to nifti files, renames them in BIDS standard, and applies gradient nonlinearity correction to the images as appropriate (gradcorrect).

[Gradcorrect](https://github.com/khanlab/gradcorrect) which calls [gradunwarp](https://github.com/kaitj/gradunwarp), adapted from the workflow used in HCP data correction. 

### Steps for getting your data into BIDS format:

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
