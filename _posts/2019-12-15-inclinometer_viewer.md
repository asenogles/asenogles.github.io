---
title: 3D Inclinometer Viewer
date: 2019-12-15 12:00:00 -0700
categories: [landslide]
tags: [webgl,potree,three.js,inclinometer,lidar,pointcloud,landslide,web,visualization]     # TAG names should always be lowercase
---

In geotechnical engineering real time <a href="https://en.wikipedia.org/wiki/Inclinometer" target="_blank">inclinometers</a> are a commonly used tool in landslide exploration, monitoring and mitigation. This consists of the installation of an array of accelerometers into a borehole drilled 10s to 100s of feet deep on an active landslide. By connecting this system to a wireless modem real time landslide displacement information across depth can be reported. See <a href="https://measurand.com/" target="_blank">measurand</a> as an example.

As lidar and other remote sensing data becomes increasingly used for landslide monitoring applications, there is a need to integrate the visualization of both remote sensing and geotechnical sensor data into a single viewer. For this project, I created a web viewer using <a href="https://github.com/potree/potree" target="_blank">potree</a> and <a href="https://threejs.org/" target="_blank">three.js</a> that enables viewing the inclinometer data (including the geologic bore log data) along with the lidar pointcloud.

![incinometer potree](/assets/incinometer_viewer/potree_inclinometer.png)
_Screenshot showing visualization of the inclinometer within a webGL web viewer_


A demo of the viewer using data from the Hooskanaden landslide can be found <a href="https://research.engr.oregonstate.edu/geomatics/projects/oregon-coast/spr807/hooskanaden/inclinometer-view.html" target="_blank">here</a>.
