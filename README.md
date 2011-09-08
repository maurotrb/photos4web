Photos for Web
==============

What is it?
-----------
I import, select and process my photos with a raw converter and a photo manager. But I prepare the photos for the web with the following work-flow to get better control:

- _Export_
  I export the photos from the photo manager in TIFF format resizing them to the maximum dimension that I put on the web (700 pixels) and renaming them using the *Title* (for SEO reasons) and *Original Time* (to guarantee the uniqueness of the file name) image meta-data: "Senigallia Hills 120455.tif".
  
- _Rename_
  I rename the photos to be more suitable to a web environment, all lowercase and without space: "senigallia\_hills\_120455.tif".

- _Strip Meta-data_
  I strip the photos of a lot of unnecessary meta-data using exiftool.
    
- _Create Thumbnails_
  For each exported photos I create three image in large (700 pixel max), medium (470 pixel max) and small (210 pixel max) dimension. For this step I use Imagemagick.
  
- _Apply Watermark_
  I apply a watermark to the large and medium thumbnails. Small thumbnail is too small to apply a readable watermark.
  
- _Copy Meta-data_
  The thumbnails are created without meta-data. I reapply the meta-data using exiftool and copying them from the exported photos.

I prepared a set of rake tasks to help me with this work-flow.

Quick start
-----------
Puts your images converted to TIFF format in a directory called <code>originals</code> and execute <code>rake</code>. The default task will create the watermark images and then the thumbnails for all the original images. You will find them in the directory called <code>thumbs</code>.

Configuration
-------------
In the first part of the Rakefile, you find a set of constants with the configuration used by the tasks.

Some of the configurations that you will probably want to change are:

- thumbnails dimension in pixels
- metadata to be stripped
- copyright notice and font used (and probably the font size depending on the font choosen)

Prerequisites
-------------
This project depends on the following tools:

- _rake_
  on Debian Linux install with <code>sudo apt-get install rake</code>
  
- _exiftool_
  on Debian Linux install with <code>sudo apt-get install libimage-exiftool-perl</code>
  
- _Imagemagick_
  on Debian Linux install with <code>sudo apt-cache search imagemagick</code>

Licensing
---------
Please see the file called LICENSE.

Contacts
--------
For question and comments:

- [MauroTaraborelli@gmail.com](mailto:MauroTaraborelli@gmail.com)
