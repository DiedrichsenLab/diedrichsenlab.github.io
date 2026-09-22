---
layout: page
title: JARVIS multi-camera tracking setup guide
author: Ivan Sivakov
date: 2026-09-21
category: tracking
description: How to set up the JARVIS multi-camera tracking system.
---

## Overview

This guide explains how to set up, operate, and calibrate the lab's eight-camera JARVIS tracking system.

## System setup

### 1. Hardware setup

The system consists of:

- An Arduino Uno microcontroller
- A lab desktop with the JARVIS TriggerFirmware, AcquisitionTool, and AnnotationTool
- Eight FLIR cameras
- USB data cables connecting the cameras to the desktop
- Trigger cables connecting the cameras to the Arduino
- A USB cable connecting the Arduino to the desktop

#### 1.1. Connect the cameras

1. Connect each camera to the lab desktop with its data cable.
2. Confirm that each camera's indicator LED is solid green, indicating that the camera is powered and connected.

#### 1.2. Connect the camera trigger wires

Combine the trigger-cable wires from the eight cameras into two bundles:

- Trigger bundle (green wires in the lab setup)
- Ground bundle (brown wires in the lab setup)

#### 1.3. Connect the Arduino

Using hook-and-pin wires, connect:

- The green bundle to Arduino pin 6
- The brown bundle to an Arduino GND pin

### 2. Install the JARVIS TriggerFirmware

The Arduino must be flashed with the JARVIS TriggerFirmware to trigger the cameras. Repeat this procedure if the Arduino is replaced or its firmware is erased.

1. Connect the Arduino to the lab desktop by USB.
2. If the `JARVIS-TriggerFirmware` repository is not already on the desktop, clone it:

   ```bash
   git clone --recursive https://github.com/JARVIS-MoCap/JARVIS-TriggerFirmware.git
   ```

3. Change to the repository directory:

   ```bash
   cd JARVIS-TriggerFirmware
   ```

4. Run the installation script:

   ```bash
   ./install_arduino_uno.sh
   ```

### 3. Start the JARVIS AcquisitionTool

If the `JARVIS-AcquisitionTool` is not installed, follow the [JARVIS AcquisitionTool README](https://github.com/JARVIS-MoCap/JARVIS-AcquisitionTool).

Launch the tool:

```bash
AcquisitionTool
```

The AcquisitionTool detects and configures the cameras, connects the external trigger, previews camera streams, and records synchronized video.

### 4. Detect the cameras and trigger

1. Open the **Connection** (second) tab in the AcquisitionTool.
2. Click **Auto Detect Cameras** and confirm that all eight cameras are detected.
3. By default, the cameras are assigned IDs 0-7 (`Camera_0` through `Camera_7`). You may rename them, but the remaining instructions assume the default names.
4. In the **Trigger** section, click **+** to add a trigger. The Arduino should appear automatically as an available device.
5. Confirm that all eight cameras and the trigger show the `Ready...` status.

### 5. Set up the cameras

1. Return to the **Camera** (first) tab.
2. Click **Setup Cameras** and accept the default settings.
3. Click the **▶** preview button and confirm that all eight camera streams are visible and displaying correctly.

## Calibration

JARVIS reconstructs 3D positions using two types of camera calibration:

1. **Intrinsic calibration** determines each camera's internal parameters.
2. **Extrinsic calibration** determines the cameras' relative positions and orientations in 3D space.

Both procedures use recordings of a checkerboard moved through the tracking area.

### 6. Create the calibration directories

Create a calibration directory on the lab desktop with the following structure:

```text
<calibration_directory>/
    Intrinsics/
    Extrinsics/
```

### 7. Record intrinsic calibrations

Record 30-60 seconds of checkerboard video for each camera. In each recording:

- Move the checkerboard so that it fills most of the frame.
- Rotate and tilt it through a range of orientations.
- Move it to different positions and distances from the camera.

A sample intrinsic calibration recording is shown below:

<video width="640" height="360" controls><source src="multi_camera_setup/sample_intrinsic_calibration.mp4" type="video/mp4"></video>

#### 7.1. Record with the AcquisitionTool

1. Use the blue folder icon in the recording toolbar to select the recording directory.
2. Enter a recording name in the text box beside the folder icon. See [Recording name](#recording-name) if the name is rejected.
3. Click the red **⏺** button to begin recording.
4. Click the red **⏹** button to stop recording.

#### 7.2. Record each camera

1. Record the intrinsic calibration for one camera at a time. The AcquisitionTool records all camera streams simultaneously; for each take, keep only the video from the camera being calibrated.
2. Place or rename the selected recordings in the `Intrinsics/` directory as follows:

   ```text
   <calibration_directory>/
       Intrinsics/
           Camera_0.avi
           Camera_1.avi
           ...
           Camera_7.avi
   ```

### 8. Record extrinsic calibrations

First, choose a primary camera to define the world reference frame. It should:

- Have a clear view of the tracking area.
- Share a checkerboard view with each other camera during calibration.
- Be securely mounted and unlikely to move. Do not move any camera after extrinsic calibration.

Record 3-4 minutes of the checkerboard moving throughout the tracking area. Cover a range of positions and orientations visible to the primary camera and each paired camera.

1. Record the extrinsic calibration as described in [section 7.1](#71-record-with-the-acquisitiontool).
2. Place the recordings in the `Extrinsics/` directory as shown below. This example uses `Camera_0` as the primary camera:

   ```text
   <calibration_directory>/
       Extrinsics/
           Camera_0-Camera_1/
               Camera_0.avi
               Camera_1.avi
           Camera_0-Camera_2/
               Camera_0.avi
               Camera_2.avi
           ...
           Camera_0-Camera_7/
               Camera_0.avi
               Camera_7.avi
   ```

### 9. Create the calibration

If the `JARVIS-AnnotationTool` is not installed, follow the [JARVIS AnnotationTool README](https://github.com/JARVIS-MoCap/JARVIS-AnnotationTool).

1. Launch the tool:

   ```bash
   AnnotationTool
   ```

2. Select **Create new Calibration**.
3. Set the calibration save path to the calibration directory created in [section 6](#6-create-the-calibration-directories), then enter a name for the calibration set.
4. Select **Separate Recordings for Intrinsics**.
5. Select the `Intrinsics/` and `Extrinsics/` directories.
6. Enable **Save Debug Images**.
7. Enter the checkerboard layout. Width and height refer to the number of inner corners, not squares. For example, a 9 x 6 inner-corner pattern has 10 x 7 squares.
8. Click **▶ Calibrate**. The process may take several minutes.
9. Confirm that the calibration finishes successfully and is saved in the calibration directory.
10. Review the reprojection errors; values around 1 pixel are generally reasonable. If calibration fails or the errors are high, inspect the saved debug images to identify poorly detected frames or insufficient checkerboard coverage.

The calibration set can now be used with recordings from the eight cameras for multi-camera tracking.

## Troubleshooting

### Recording name

If the AcquisitionTool reports that a recording name is already taken, check the recording location. On the lab desktop, this can happen when the tool defaults to the `/media` SSD and the drive is not mounted. Open the SSD in the Linux file manager to mount it, then try again.
