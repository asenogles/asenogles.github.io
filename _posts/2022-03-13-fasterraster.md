---
title: Faster Raster
date: 2022-03-13 12:00:00 -0700
categories: [programming]
tags: [raster,dem,python,cython,openmp,lidar]     # TAG names should always be lowercase
---

Recently, I've been working on developing <a href="https://ai.facebook.com/blog/self-supervised-learning-the-dark-matter-of-intelligence/" target="_blank">self-supervised</a> deep learning models for <a href="https://en.wikipedia.org/wiki/Raster_graphics" target="_blank">raster</a> processing (specifically processing <a href="https://www.usgs.gov/faqs/what-digital-elevation-model-dem" target="_blank">DEMs</a>). Generating labelled training data as part of the self-supervised learning process involves the manipulation of 10s to 100s of thousands of rasters. In the Python ecosystem (which is currently the dominant language for developing deep learning methodologies) currently available raster processing libraries (such as <a href="https://gdal.org/api/python_bindings.html" target="_blank">GDAL</a>) are prohibitively slow for performing such processing volumes.

To overcome this, I developed a new Python library for raster manipulation and I/O, named <a href="https://github.com/asenogles/fasterraster" target="_blank">FasterRaster</a>. The goal of <a href="https://github.com/asenogles/fasterraster" target="_blank">FasterRaster</a> was to implement fast parallel raster processing operations via <a href="https://cython.org/" target="_blank">Cython</a> wrapped <a href="https://www.openmp.org/" target="_blank">OpenMP</a> C functions. Memory allocation, I/O, and data objects are handled in <a href="https://numpy.org/" target="_blank">NumPy</a> allowing for straightforward memory management and easy addition of processing operations.

You can check out the github repository <a href="https://github.com/asenogles/fasterraster" target="_blank">here</a>.
