 **FITS image decoder and encoder**
==============

> Organization: FFmpeg

Student information
-------------------

### Personal details ###
* **Name**: Adeel Ahmad
* **Email**: adeelahmadadl1995@gmail.com
* **GitHub**: adl1995
* **IRC (#OpenAstronomy)**: adl1995
* **Timezone**: UTC +05:00
* **Blog**: http://adl1995.github.io/
* **IRC (#ffmpeg-devel):** adeel

### Academic details ###
* **University**: National University of Computer and Emerging Sciences, Islamabad
* **Degree**: Computer Science 
* **Graduation year**: 2018 

Background
------

In my previous semester, I enrolled myself in Digital Image Processing course. During the tenure of this course I implemented filters (Gaussian, Sobel, Prewitt, Laplacian), edge detectors (Canny, Marr Hildreth), morphological operators (Convex hulling, Erosion, Dilation), object detectors (Generalized Hough transform, simple convolution), and seam carvers. These were implemented purely in Python, making no use of any library other than ``Numpy`` and ``Matplotlib``. This attenuated my interest in Computer Vision and I have been an active researcher since then. I have showcased these project on my [GitHub profile](https://github.com/adl1995). The most interesting algorithm I implemented was [Generalized Hough Tranfrom](https://github.com/adl1995/generalised-hough-transform). I was amazed as to how something as simple as an equation of line could lead to detection of objects in images. Although my implementation was only scale invariant, I plan on extending this invariancy to orientation as well. Even though this project requires to build a FITS image decoder and encoder and has not direct application in computer vision, I still believe that my previous experience will help me get up to speed. 

Apart from this, I have also been a PHP web developer for almost two years, and I have worked on multitude of project ranging for SaaS to e-commerce websites. I work remotely on [Upwork](https://www.upwork.com/freelancers/~018e56b8591046f889) and [Fiverr](https://www.fiverr.com/adl1995). My clients are either IT organizations or independent contractors. I have also worked on web automation and data scraping using ``Selenium`` and ``BeautifulSoup``. Although these skills are not required for this organization and have no direct links in my project, it shows that I am persistent and hard-working. The skills that I acquire are through self learning and consistency. Currently I am building an application using [Google Application Engine](https://github.com/adl1995/zoho-portal).

I started collaborating with Open Source organizations for only about a month now, and my experience has been excellent so far. The mentors have been very kind, welcoming and have provided assistance along the way. Given my background in Computer Vision and being good at problem solving, I firmly believe this prestigious organization and its users will benefit from the FITS client for FFmpeg.

Project Details
---------------

### Mentors ###
> * [Paul B Mahol](https://github.com/richardpl)
> * [Rostislav Pehlivanov](https://github.com/atomnuker)

### Abstract ###
> Design and develop a decoder / encoder for Flexible Image Transport System (FITS) digital file format. Being designed specifically for astronomical purposes, this format supports functionality for describing photometric and spatial calibration information. The software would be written in C.

### Detailed description ###

#### Overview of FITS ####
Flexible Image Transport System (FITS) digital file format was developed by IAU FITS Working Group and was standardised in 1981. This format was designed to provide long-term archival storage. FITS allows to write image metadata in human readable ASCII, this allows users to examine headers of a file of unknown provenance. The keyword / value pairs provide information such as size, origin, coordinates, binary data format, free-form comments, history of the data, and anything else the creator desires.

This file format can also be used for non-images purposes, such as spectra, photon lists, data cubes, or even structured data such as multi-table databases. But, in the context of this project, I will only work with FITS images.

Currently, there are a multitude of options available for viewing FITS images, such as:

* Aladin Lite
* APLpy
* GAIA

#### The HEALPix framework ####

HEALPix, an acronym of 'Hierarchical Equal Area isoLatitude Pixelization of a sphere', is a framework for discretizing high resolution data. The software is available in C, C++, Fortran90, IDL, Java, and Python. It extends a data structure (with a library), for each language. The main features provided by this software are:

  * Pixel manipulation
  * Spherical Harmonics Transforms
  * Visualization
  * Input / Output (supports FITS files)

In a nutshell, the pixelization procedure subdivides a spherical sphere in which each pixel is equidistant from the origin - meaning it covers the same surface area. This produces a HEALPix grid, whose interesting property is that pixels are distributed on lines of constant latitude. Due to this iso-latitude distribution of pixels the complexity for computing integrals over each harmonics is N<sup>1/2</sup>.

#### HEALPix coordinate system ####
HEALPix header files contain one of the following three letters, each depicting the coordinate system being used:

* C:Celestial=ICRS=RA/DEC(equatorial)=FK5 J2000
* G:Galactic
* E:Ecliptic

HiPS uses the Celestial coordinate system by default. Also, HiPS does not support Ecliptic coordinate system.

#### HiPS images & catalogues ####
The way HiPS represents images is by resampling them on a HEALPix grid at the maximum desired order, say k<sup>max</sup>. Then it generates tile images for tile orders. When mosaicking / stitching images, the angular resolution is taken into account. There are various methods for filling the data region when stitching images and dealing with background difference. The k<sup>max</sup> chosen earlier determines minimum pixel size which is near to the angular pixel size or the resolution of original data.

Next important thing is whether to emphasize on ``display quality`` or ``photometric accuracy``, which depends on our use case. Image encoding can be done either in **FITS**, **PNG**, **JPG** file format. For most cases it is enough to only generate FITS and PNG files. The lowest order pixel values correspond to a large area of the sky. The HiPS indexing structure takes care of mapping correct tiles onto a display.

HiPS generation for huge amounts of data such as the Hubble Space Telescope requires planning of system growth.

The same way a tile in HiPS image survey contains a 512x512 image, a tile catalogue contains the RA / DEC coordinates stored in a TSV file. The data is ASCII tab separated and is organized in various directories the same way as HiPS images. 

## Benefits ##
This project would allow FFmpeg to extend its services to astronomical domain. It would add support for decoding and encoding FITS images from through the command line interface.

Qualification task
---------
I frequently conversed with Paul B Mahol, the mentor of this project on the ``#ffmpeg-devel`` IRC channel. He assigned me with the qualification task to decode an XPM (X PixMap) image in ``libavcodec``. The work revolved around working with a patch (https://ffmpeg.org/pipermail/ffmpeg-devel/2012-June/126476.html). It contained incomplete code for encoding an XPM image. This had to be transformed for decoding. After this, support for >256 colors had to be added. For this characters per pixels ``cpp`` had to be set to 3. The pixel format also had to be modified to ``AV_PIX_FMT_RGBA``, as this provides an extra channel.

I made numerous commits on my local fork of ``FFmpeg`` (https://github.com/adl1995/FFmpeg/commits/new_branch).

To make the code generic and to allocate memory accordingly for ``cpp =  1, 2, or 3``, I wrote this piece of code

```c
size = 1;
for (j = 0; j < cpp; ++j)
{
  t = 94;
  for (k = j; k > 0; --k)
  t *= t; 
  size += t; 
}
```

Finally a check had to be added to verify if the characters in pixel were supported or not.

```c
av_log(avctx, AV_LOG_ERROR, "unsupported number of chars per pixel\n");
```

The next step was to deal with C styled comments in the ``.xpm`` file. Comments could be either multiline ``/**/`` or single line ``//``. I wrote a function utilizing the ``strcspn`` method for skipping specific patterns.

```c
static size_t skip_chars_comments(uint8_t **cpixel, char *delimiter)
{
    int len;

    len = strcspn(*cpixel, delimiter) + 1;

    *cpixel += strcspn(*cpixel, "//");
    *cpixel += strcspn(*cpixel, "\n");
    *cpixel += strcspn(*cpixel, "/*");
    *cpixel += strcspn(*cpixel, "*/");

    return len;
}
```

For testing purposes I used ``GIMP`` to a ``2048x1356`` image to ``XPM`` format. Afterwards, I used ffplay to decoded the image

```bash
./ffplay  ~/Desktop/screen.xpm
```

The image was now decoded, however, the alignment was a bit skewed.

## Timeline ##

| Time Period        | Plan           |
| ------------- | ------------- |
| May 04, 2017 - May 30, 2017 **(Community Bonding Period)**      |   <ul><li>Finalize on the deliverables.</li><li>Discuss with mentors and get a final idea on how to approach the problem.</li></ul>|
| | **Part 1 starts** |
| May 30, 2017 - June 15, 2017 ( 2 weeks ) | <ul><li>Write a skeleton layout of the algorithm / code to implement.</li></ul> |
| June 16, 2017 - June 30, 2017 ( 2 weeks ) | <ul><li>Document and write test cases for the added part.</li><li>**Update 1 : Push code.**</li></ul>|
| | **Part 1 completed**<br />**Part 2 starts** |
| July 01, 2017 - July 14, 2017 ( 2 weeks ) | <ul><li>Add test cases and complete decoding functionality.</li></ul> |
| July 15, 2017 - July 28, 2017 ( 2 weeks ) | <ul><li>Add optimizations, write test cases.</li><li>Add support for FITS image encoding.</li></ul> |
| | **Part 2 completed** |
| August 21, 2017 - August 29, 2017 **(Students Submit Code and Evaluations)** | <ul><li>Clean up code.</li><li>Improve documentation.</li><li>Add further test cases.</li><li>Code refactoring (if required).</li><li>Resolve merge conflicts (if any).</li></ul> |
| August 29, 2017 - September 05, 2017 | Mentors submit final student evaluations. |
| September 06, 2017 | Results Announced. |

## Availability ##

My final exams end on May 20<sup>th</sup>. So, I will have ample time for the community bonding period. After that, I will be free during the whole summer i.e. almost three months. I do not have any other commitments, so I can focus all my attention to GSoC.

