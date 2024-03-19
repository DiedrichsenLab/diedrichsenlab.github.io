---
layout: page
title: DICOM to Nifti conversion using autobids routines
---

## The CFMM DICOM Server

MRI datasets acquired at CFMM are stored in the [CFMM DICOM Server](https://dicom.cfmm.uwo.ca/dcm4chee-arc/ui2/study/study). To access the data of a specific project, tipe PRINCIPAL^PROJECT-ID (e.g., DIEDRICHSEN^SMP) in the "Study Description" search field, then click Submit. You will see the list of datasets (i.e., participants, sessions...) collected for your study. Each dataset is identified by a Patient's Name with a standard format YYYY_MM_DD_PROJECT-ID_SUBJ-ID (e.g., 2024_03_13_SMP1_100). In each dataset, MRI data are stored in the DICOM format (.dcm). **You don't need to download the datasets from the CFMM DICOM Server to perform the conversion to Nifti (.nii).

## First step: *cfmm2tar*

DICOM to Nifti conversion involves two steps (*cfmm2tar* and *tar2bids*). Both can be run from the terminal of the CBS server. 

1) Log in into the CBS server. See [this guide](https://osf.io/k89fh/wiki/Computational%20Core%20Server/) to access the CSB server for the first time.

2) Open the Terminal

3) Run the following command:

```
singularity run /srv/containers/cfmm2tar_v1.0.0.sif <out_dir>
```

`<out_dir>` is the directory where *cfmm2tar* puts the output data. You can use */local/scratch* as an output directory for *cfmm2tar*, but always remember to clean the directory when you are done.

You can add flags to *cfmm2tar* to perform this step on a specific project or dataset. With the flag -p <PRINCIPAL^PROJECT-ID> you can input to *cfmm2tar* all the datasets beloning to a specific project:

```
singularity run /srv/containers/cfmm2tar_v1.0.0.sif -p <PRINCIPAL^PROJECT-ID> <out_dir>
```

With the flag -n <Patient's Name> you can input a specific dataset by using the "Patient's Name" field in the CFMM DICOM Server:

```
singularity run /srv/containers/cfmm2tar_v1.0.0.sif -n <Patient's Name> <out_dir>
```

*cfmm2tar* will create a .tar and a .uid file in the output directory.

## Second step: *tar2bids*

After you have created the .tar and .uid file with *cfmm2tar*, the second step involves creating the BIDS repository with the .nii file. Run the following command in the Terminal:

```
singularity run /srv/containers/tar2bids_v0.2.4.sif -h <project-id>_heuristic.py -o <out_dir> <filename>.tar
```

`<filename>` is the name of the .tar file created by *cfmm2tar* in the previous step. `<out_dir>` is the directory where tar2bids puts the Nifti files. One possibility is to set /local/scratch/BIDS as `<out_dir>`. `<project-id>_heuristic.py` is a Python script with the information needed to convert functional runs from DICOM to Nifti. If your project is new and you don't have a heuristic file yet, see the next paragraph to learn how to create one. 

## Heuristic files

## Gradcorrect

After you have a BIDS repository with the Nifti files, run the following command:

```
singularity run /srv/containers/khanlab_gradcorrect_v0.0.3a.sif <bids_dir> <out_dir> participant --grad_coeff_file /srv/software/gradcorrect/coeff_AC84.grad
```




Inside `<out_dir>` tar2bids puts several files and folder, including a sub-XX folder with three folders: anat, func and fmaps. 




