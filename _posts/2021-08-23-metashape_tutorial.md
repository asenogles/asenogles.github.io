---
title: Processing UAS imagery in Agisoft Metashape
date: 2021-08-23 12:00:00 -0700
categories: [UAS]
tags: [agisoft metashape,uas,sfm,ppk,pointcloud]     # TAG names should always be lowercase
---

Agisoft Metashape is a popular software program for processing digital imagery into pointclouds, digital elevation models (DEMs), and orthomosaic images using photogrammetric and structure from motion (SfM) techniques. This post contains a straightforward workflow for using Agisoft Metashape to process a <a href="https://en.wikipedia.org/wiki/Nadir" target="_blank">nadir</a> UAS (unmanned aircraft systems) imagery dataset. This post does not discuss any of the underlying theory or principles used in photogrammetric processing, rather contains a practical guide and starting point for using the software.

Before starting however, it is worth noting that the best workflow/parameter values will vary across datasets depending on, but not limited to:
 - UAS platform used
 - Camera Used
 - Flying height
 - Land Cover type (terrain, vegetation, urban vs rural etc.)
 - Utilization of direct georeferencing of the images
 - Number of GCP’s used
 - Lightning conditions

As a result, the guide presented can be used as a starting point, however it is the responsibility of the individual pracitioner to determine the best workflow/parameters for their specific project.

## Pre-processing

Before proceeding with processing the imagery in Agisoft Metashape, there are a few pre-processing steps which can be used to increase the accuracy of any resulting models primarily direct georefencing of the images, and the use of GCP's (Ground Control Points).

### Step 1: Estimate the camera coordinates/orientation


If GNSS/IMU data of the UAS platform is available, then the user should estimate the coordinates of the camera positions using PPK method or similar. An example of this using <a href="https://rtklib.com/" target="_blank">RTKLIB</a> with the DJI Phantom 4 Pro can be found in a <a href="../uas_trajectory" target="_blank">previous blog post</a>. Note, this was developed specifically for the DJI P4P system and adaptations may be necessary for other UAS platforms.

### Step 2: Derive GCP coordinates (if available)

If using GCP’s (Ground Control Points), derive their coordinates from Total Station/GNSS measurements and export to a text file in your preferred coordinate system.

## Processing in Agisoft Metashape

### Step 1: Project setup and data cleaning

This section covers some project setup and data cleaning the user may wish to perform prior to performing any computation in Agisoft Metashape.

1. Create a suitable directory template, note Metashape does not copy the image files, but rather references them from their import location using relative file paths. Therefore, it is advised to copy the images to a local directory that will contain both the metashape project, and the images. An example template could be as follows:
    - {Project Name}
        - 00_IMAGES
            - FLIGHT_0001
                - FLIGHT_0001_{image Number}.jpg
        - 01_METASHAPE
            - {project name}.files
            - {project name}.psx
        - 02_Export
            - {project name}_dense.las
            - {project name}_ortho.tif
            - etc.
1. Import the images for each of the flights.
1. (**recommended**) Set up the GPU
    - Click Tools > Preferences, then click on the GPU tab
    - Make sure a suitably powerful GPU is selected under the “GPU devices”.
    - This will dramatically speed up several of the processes.
1. (**recommended**) If using direct georeferencing, import camera exterior orientation.
    - This should contain:
        - X, Y, Z coordinate for each image from PPK/RTK processing
        - X, Y, Z coordinate accuracy estimate for each image
        - Rotation (yaw, pitch, roll) from exif data for each image
    - Be sure to select the correct Coordinate system/projection for the coordinates used.
    - Make sure the accuracy estimates for the coordinates/angle seem reasonable for the processing method used.
1. (**recommended**) if using GCP’s, import the ground control points
    - This should contain:
        - X, Y, Z coordinate for each GCP
        - X, Y, Z coordinate accuracy estimate for each GCP
    - Make sure this is the same coordinate system/project as used for the image coordinates.
    - Make sure the accuracy estimates for the coordinates seem reasonable for the processing method used.
1. (**recommended**) Estimate the image quality:
    - In the photos tab, right-click a photo, then select estimate image quality.
    - At the top of the photos tab, sort by image quality.
    - Go through images below around 0.8 and disable any blurry or otherwise bad images.
1. (***optional***) Add Image masks:
    - Go through each of the photos, and for any moving objects (cars, people etc.), use one of the select tools to draw a fence around the object.
    - Then right click, and select “Add selection”.
    - Once complete, export the image masks by right-clicking on a photo in the photos tab, then click masks > export masks.
    - This step is optional and not necessary, but should help clean up output data, especially for any orthomosaic images generated.

### Step 2: Image alignment, adjustment, and GCP registration

1. **Generate the sparse pointcloud**
    - Select workflow > Align Photos and input the following:
        - Quality - High
        - Generic Preselection - checked
        - Reference preselection - checked, source
        - Key point limit - 60,000
        - Tie Point limit - 4,000
        - Adaptive Camera model fitting - No
        - Exclude stationary tie points: No
        - Guided image matching: No
        - (***optional***) Apply Masks to: Key Points
    - This could take several hours depending on the size of project, and resources available.
    - Once complete, if any photos fail to align, evaluate the photos to determine whether the photo alignment should be re-run, or whether the photos should be disabled. This could be due to the photo consisting of: ocean, dense tall timber, shadows/bad lighting, moving objects etc. 
1. **Run the initial Bundle Adjustment**
    - In the Reference pane, Select the magic wand “optimize cameras” button.
    - Check the following:
        - f
        - cx/cy
        - k1
        - k2
        - k3
        - k4
        - p1
        - p2
1. **Set up the GCP’s**
    - In the Reference pane, for each control point, right-click the control point then click filter by marker and complete the following:
        - Open each image and move/verify the marker is in the center of the target.
        - Cycle through the images using the pg up/down keys.
        - Markers colored green will be used in future processes, markers colored grey are for representation only.
        - Verify/move control points in images where target is clear/undistorted only.
    - Uncheck 2 or 3 Control points to use as checkpoints, these will not be used in the georeferencing process and should give a relative idea of model accuracy.
    - At the top of the reference pane, click the “update” button to apply the settings.
1. **Re-run the bundle adjustment**
    - Re-run the bundle adjustment using the same settings as above.
1. **Save the project**
    - Save the project, then save a copy of the project to {projectName}_gradual and open up the copied project.
    - This will allow you to come back to this step in the future if you wish to re-run the gradual selection steps.
1. **Gradual Selection**
    - Select model > gradual selection
    - Slowly whittle down the number of points in the sparse pointcloud until you reach the following approximate values:
        - Reprojection error - 0.3
        - Reconstruction uncertainty - 15
        - Projection accuracy - 5
    - After each selection, you should delete the selected points, then re-run the bundle adjustment.
    - It may take 3-4 cycles to reach the desired values.
    - As a general rule of thumb, you want to delete no more than 50% of the points.
    - The optimal values will vary for each project depending on the factors discussed in the introduction section above. It is recommended to play around with these settings.
    - Once complete, run the bundle adjustment one final time.

### Step 3: Further data products generation

Now that the camera exterior/interior orientation has been determined, and the images have successfully been aligned, the user can generate further data products such as the dense pointcloud, mesh, DEM, orthomosaic photo etc. The following is a recommendation of starting settings/workflow, but as always ideal settings may vary depending on the specifics of the project.

1. (**Recommended**) Generate Dense Pointcloud
    - Workflow > Build Dense pointcloud then select the following:
        - Quality: High
        - Depth filtering: Aggressive
        - Calculate Point colors: Checked
        - Calculate Point confidence: Checked
    - This may take a long time depending on project size and computing resources available. For faster results it may be necessary to reduce the quality.
1. (***optional***) Filter by point confidence
    - Tools > Dense cloud > Filter by confidence.
    - Gradually remove low confidence points.
    - Remove enough points to reduce noise, but not so much as to create holes.
1. (***optional***) Generate Mesh
    - Workflow > Build Mesh then select the following:
        - Source Data: Dense Cloud
        - Surface type: Height field (2.5D)
        - Face Count: High
        - Interpolation: Enabled
        - Calculate vertex colors: checked
1. (**Recommended**) Generate DEM
    - Workflow > Build DEM then select the following:
    - Source Data: Dense Pointcloud
    - Interpolation: Enabled
1. (**Recommended**) Calibrate colors
    - Tools > Calibrate Colors then select the following:
        - Source Data: DEM
        - Calibrate white balance: not checked
1. (**Recommended**) Generate Orthomosaic
    - Workflow > Build Orthomosaic then select the following:
        - Surface: Mesh
        - Blending mode: Mosaic
        - Refine Seamlines: yes
        - Enable hole filling: Yes
        - Enable ghost filtering: Yes (prevents oblique/non nadir imagery from being used)
        - Color calibration


## References

The following materials were referenced when developing this workflow: 

 - <a href="https://www.agisoft.com/downloads/user-manuals/" target="_blank">Agisoft Metashape Manual</a>
 - <a href="https://agisoft.freshdesk.com/support/solutions/articles/31000153696" target="_blank">Agisoft Metashape Aerial data processing (with GCPs)</a>
 - <a href="https://doi.org/10.3133/ofr20211039" target="_blank">USGS Processing Coastal Imagery With Agisoft Metashape Professional Edition, Version 1.6—Structure From Motion Workflow Documentation</a>