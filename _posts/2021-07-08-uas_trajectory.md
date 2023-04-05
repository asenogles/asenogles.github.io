---
title: UAS Trajectory Reconstruction
date: 2021-07-08 12:00:00 -0700
categories: [UAS]
tags: [rtklib,python,uas,gnss,sfm,ppk]     # TAG names should always be lowercase
---

Accurately estimating the trajectory of a sensor is essential when trying to construct a geospatial model using data captured by the sensor. In the case of UAS (unmanned aircraft systems) imagery, the position (X,Y,Z) and rotation (roll, pitch, yaw) of the camera for each image should be estimated. This is most commonly conducted using GNSS measurements using a PPK approach. IMU measurements can also be added if available. This post contains a simple practical workflow for estimating the camera positions of UAS imagery using <a href="https://rtklib.com/" target="_blank">rtklib</a>. The workflow is primarily designed for processing and extracting camera orientation parameters from the DJI phantom 4 pro for Agisoft Metashape using Python.

# Workflow

## Step 1: data organization and setup

1. Download the latest version of rtklib. I am using the <a href="http://rtkexplorer.com/downloads/rtklib-code/" target="_blank">demo5 fork</a>.
1. Make a local copy of <a href="https://github.com/asenogles/uas-trajectory" target="_blank">my github repo</a> that goes along with this post.
1. In the PROJECT_TEMPLATE directory, insert the following files:
    - 01_BASE - Base rinex observation and navigation file/s
    - 02_ROVER - Create a new directory for each flight containing the rover rinex observation file/s and the image timestamp file (.MRK).
    - (***OPTIONAL***) 03_EPHEMERIS - precise ephemeris (.sp3) and clock (.clk) data, this can be downloaded from <a href="https://cddis.nasa.gov/archive/gnss/products/" target="_blank">NASA</a>.
    - (***OPTIONAL***) 04_GEOID - The geoid grid file (.bin) with geoid heights for the survey location. In North America these are available to download from <a href="https://www.ngs.noaa.gov/GEOID/" target="_blank">NGS</a>.
    - 05_ANTENNAS - Antenna Calibration file (.atx) containing the antenna offset for the base and/or rover.
    - 06_OUT - directory to contain output files
1. If you haven't already, derive the position of the base station in geodetic coordinates using OPUS or similar.


## Step 2: Running the RTKPOST GUI

1. Run the RTKPOST GUI.
1. Select the rover, base, navigation, and ephemeris (optional) data to use, and select the 06_OUT directory as the output directory.
1. Select the options button and then under the setting1 tab enter the following:
    - Positioning mode: Kinematic
    - Frequencies: L1/L2
    - Filter type: Combined
    - Elevation Mask: 15 (should be default)
    - Rec Dynamics: on (should be default)
    - Satellite ephemeris: Precise (only if using precise ephemeris, else set to broadcast)
    - Constellations: GPS, GLONASS, GALILEO (dependent on antenna/receivers used)
1. Move to the setting2 tab and enter the following:
    - Integer Ambiguity: Fix and hold
1. Move to the output tab and select the following:
    - Solution Format: Lat/Lon/Height
    - Output Header: Off
    - Output Processing Options: Off
    - Time Format: ww ssss GPST
    - Latitude Longitude format: ddd.dddddd
    - Field Separator: comma (,)
1. Move to the files tab
    - Select the Antenna calibrations file in the second Satellite/Receiver box (for the base), and in the first box (for the rover).
1. Save the options for future use.
1. Move to the positions tab
    - Enter the coordinates to use for the base reference station.
    - Enter the antenna height offset height.
    - Select the Antenna Type.
1. Select ok to close the options, then click the execute button to start the trajectory processing.
1. Click the plot button, and make sure all/most of the ambiguities are fixed (Q=1).
1. This should have generated a *.pos file in the output folder containing the processed trajectory coordinates. This will be used in the next step.

## Step 3: Extracting the camera exterior orientation rotation parameters.

The exif data in the images contains valuable rotation parameter data (roll, pitch, yaw). The easiest way to get this information is using Agisoft Metashape as described below:

1. Load the images into Agisoft Metashape.
1. Under the "reference" pane click export
    - enter a filename ({flightName}_inclination.txt)
    - Under items select "cameras"
    - Under delimiter select "comma"
    - Under columns select "save rotation"
    - Click ok
1. This should have generated a file containing the rotation (in Yaw, Roll, Pitch, see the <a href="https://www.agisoft.com/downloads/user-manuals/" target="_blank">Metashape user manual</a> for details on rotation convention) parameters for each image.

## Step 4: Interpolating the camera exterior orientation translation parameters in a projected coordinate system.

Next we need to interpolate the position of the camera at each image within a projected coordinate system. This can be done using the trajectory.py code in the <a href="https://github.com/asenogles/uas-trajectory" target="_blank">github repo</a> as described below:  
1. Install the dependencies from this repo if not already (pip install -r requirements.txt)
1. Run the trajectory.py script and select the following files:
    - Select the rtklib *.pos file(s) generated in step 2.
    - Select the rotation *.txt file(s) from agisoft metashape generated in step 3.
    - Select the timestamp *.MRK file(s) that contains the timestamp of each image.
    - *(optional)* Select the geoid *.bin file covering the survey area. For the US, these are available from <a href="https://www.ngs.noaa.gov/GEOID/" target="_blank">NGS</a>.
1. Enter the EPSG code of the projected coordinate system you would like to use, if unknown, these can be found on the <a href="https://epsg.io/" target="_blank">EPSG website</a>.
1. Enter a multiplier to scale the standard deviation values reported by rtklib.
    - The standard deviation values can be used as uncertainty estimates in agisoft metashape to constrain the camera origins. The standard deviation values outputted by rtklib (and other GNSS post-processing software) are often overly optimistic of the actual uncertainty and thus scaling these values by a constant factor can provide a more reasonable uncertainty estimate. In my experience, 10 is a good default multiplier.

1. This should generate an output text file ({filename}_cameras.txt) containing the exterior camera orientation parameters (rotation and translation) for each image in a local projection where Z represents the orthometric height.

## Step 5: Next Steps

The outputted text file can be imported into Agisoft Metashape by clicking the import button under the "reference" pane or any other data processing software. As always control points/data should be used to verify any products resulting from the processed trajectory, it is the responsibility of the user to make sure this is suitable for their own workflow. 