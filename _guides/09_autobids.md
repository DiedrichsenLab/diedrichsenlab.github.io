---
layout: page
title: DICOM to Nifti conversion using autobids routines
---

## The CFMM DICOM Server

MRI dataset acquired at CFMM are stored in the [CFMM DICOM server](https://dicom.cfmm.uwo.ca/dcm4chee-arc/ui2/study/study). To access the data of a specific project, tipe PRINCIPAL^PROJECT-ID (e.g., DIEDRICHSEN^SMP) in the "Study Description" field, then click Submit. You will see the list of datasets (i.e., participants, sessions...) collected for your study. Each dataset is identified by a Patient's Name with a standard format YYYY_MM_DD_PROJECT-ID_SUBJ-ID (e.g., 2024_03_13_SMP1_100). In each dataset, MRI data are stored in the DICOM format (.dcm).You don't need to download the datasets to perform the conversion to Nifti (.nii).

## First step: cfmm2tar

DICOM to Nifti conversion involves two steps, both can be run from the terminal of the CBS server. 

1) Log in into the CBS server. See [this guide](https://osf.io/k89fh/wiki/Computational%20Core%20Server/) to access the CSB server for the first time.

2) Open the Terminal

3) Type the following command:

```
---
/srv/containers/cfmm2tar_v1.0.0.sif <out_dir>
---
```

Outdir is the directory where cfmm2tar puts the output data. You can use */local/scratch* as an output directory for *cfmm2tar*, but always remember to clean the directory when you are done. **Those who don't clean the directory will need to make 100 push-ups at the next lab meeting**.

You can add flags to *cfmm2tar* to perform this step on a specific project or dataset. With the flag -p <PRINCIPAL^PROJECT-ID> you can input to *cfmm2tar* all the dataset beloning to a specific study:

```
---
/srv/containers/cfmm2tar_v1.0.0.sif -p <PRINCIPAL^PROJECT-ID> <out_dir>
---
```

With the flag -n <Patient's Name> you can input a specific dataset by using the Patient's Name field in the CFMM DICOM Server:

```
---
/srv/containers/cfmm2tar_v1.0.0.sif -n <Patient's Name> <out_dir>
---
```




