---
layout: page
title: Working with CIFTI files
---

## Types of cifti-files


Cifti files can be pretty handy to store imaging data in a flexible way. They can store data in a 2D matrix, with the two axis defining the type of data you are looking at. This allows for a lot of flexibility in how data is stored and displayed.


There are 5 different kinds of axis:

* BrainModelAxis: each row/column is a voxel or vertex - multiple brainModelAxis can be combined into a single axis
* ParcelsAxis: each row/column is a group of voxels and/or vertices
* ScalarAxis: each row/column has a unique name (with optional meta-data)
* LabelAxis: each row/column has a unique name and label table (with optional meta-data)
* SeriesAxis: each row/column is a timepoint, which increases monotonically

The two different axes define a set of different type of CIFTI files - each with a different extension. The column axis usually is a BrainModel or Parcels axis - this is the space that is being displayed. The rows is the type of data that is displayed in that space.


| File extension | Description | Rwo Axis[0] | Column Axis[1] |
|----------------|-------------|---------|---------|
| .dscalar.nii   | Dense scalar data | Scalar | BrainModel |
| .pscalar.nii   | Parcel scalar data | Scalar | Parcel |
| .dtseries.nii   | Dense Timeseries | Series | BrainModel |
| .ptseries.nii   | Parcel Timeseries | Series | Parcel |
| .dlabel.nii   | Dense label data | Label | BrainModel |
| .plabel.nii   | Parcel label data | Label | Parcel |
| .dconn.nii   | Dense connectivity | BrainModel | BrainModel |
| .pconn.nii   | ROI-to-ROI connectivity  | Parcel | Parcel |
| .dpconn.nii   | Dense-to-ROI connectivity | Parcel | BrainModel |
| .pdconn.nii   | ROI-to-dense connectivity | BrainModel | Parcel |
| --------------|-------------|---------|---------|

See also:
[https://balsa.wustl.edu/about/fileTypes](https://balsa.wustl.edu/about/fileTypes)

There is no reason that you cannot have other combination of axes, but connectome workbench will only display the above types of files.


## Displaying CIFTI files in workbench

When you load a cifti-file into connectome workbench, you can display it automatically. For example you may have extracted ROI mean activation data and stored it in a pscalae file. You can load this file into workbench and display it on the surface and / or volume (depending whether the parcels you defined contain vertices or voxels).

dconn and pconn files in workbench are assumed to be connectivity values between the same structure with itself (i.e. intracortical connectivity - so the connectivity matrix needs to be square).

dpconn and pdconn files can have connectivity values between different structures (i.e. Cortical parcels to cerebellar voxels), so they can be displayed.

You can render pdconn files on the cortical surface, and select the row (cerebellar voxel) by clicking on the volume display.

![pdconn example](/assets/workbench_pdconn_example.png)

Vice versa, you can render dpconn files in the cerebellar volume, and select the row (cortical parcel) by clicking on the surface display.

![dpconn example](/assets/workbench_dpconn_example.png)





