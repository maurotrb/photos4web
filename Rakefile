# Rakefile for photos4web -*- ruby -*-

# Copyright 2011 by Mauro Taraborelli (MauroTaraborelli@gmail.com)
# All rights reserved.

# This file may be distributed under MIT license.
# See LICENSE files for details.

require 'rake/clean'
CLEAN.add('*fgnd.png', '*mask.png')
CLOBBER.add('*.png', 'thumbs')


# Configuration constants ------------------------------------------------------

# Directories
ORIGINALS_DIR     = 'originals'
LARGE_THUMBS_DIR  = 'thumbs/large'
MEDIUM_THUMBS_DIR = 'thumbs/medium'
SMALL_THUMBS_DIR  = 'thumbs/small'

# Thumbnails labels
LARGE_THUMB_LABEL  = '_l'
MEDIUM_THUMB_LABEL = '_m'
SMALL_THUMB_LABEL  = '_s'

# Extension of originals and thumbnails images
ORIGINALS_EXT    = 'tif'
THUMBS_EXT       = 'jpg'

# Thumbnails max dimensions in pixel
LARGE_THUMB_DIM  = 700
MEDIUM_THUMB_DIM = 470
SMALL_THUMB_DIM  = 210

# Exiftool parameters to strip from images
# unnecessary metadata.
ET_STRIP_METADATA = [
                     '-XMP-crs:All=',
                     '-XMP-exif:All=',
                     '-XMP-aux:All=',
                     '-XMP-photoshop:All=',
                     '-XMP-tiff:All=',
                     '-XMP-xmp:All=',
                     '-IFD1:All='
                    ].join(' ')

# Original images
ORIGINALS = FileList[File.join(ORIGINALS_DIR, '*'.ext(ORIGINALS_EXT))]

# Watermark images
LARGE_WATERMARK_WIDTH     = 400
LARGE_WATERMARK_HEIGHT    = 70
LARGE_WATERMARK_FONT      = 'Aller-Regular'
LARGE_WATERMARK_FONTSIZE  = 24
LARGE_WATERMARK_TEXT      = '© maurotaraborelliphoto.com'
LARGE_WATERMARK_NAME      = 'watermark_l'
LARGE_WATERMARK           = LARGE_WATERMARK_NAME.ext('png')
MEDIUM_WATERMARK_WIDTH    = 300
MEDIUM_WATERMARK_HEIGHT   = 50
MEDIUM_WATERMARK_FONT     = 'Aller-Regular'
MEDIUM_WATERMARK_FONTSIZE = 18
MEDIUM_WATERMARK_TEXT     = '© maurotaraborelliphoto.com'
MEDIUM_WATERMARK_NAME     = 'watermark_m'
MEDIUM_WATERMARK          = MEDIUM_WATERMARK_NAME.ext('png')


# Utility methods to create thumbnails -----------------------------------------

# Gets file names more suitable for the web:
# - substitutes spaces and dashes with underscores
# - converts all characters to lower case
def normalize_for_web(name)
  name.downcase.gsub(/[ \-]/, '_')
end

# Creates a thumbnail using Imagemagick "convert"
def create_thumb(original, thumb, dimension)
  sh "convert #{original} -thumbnail #{dimension}x#{dimension} -unsharp 0x.5 -quality 80 #{thumb}"
end

# Applies a watermark to the thumbnail using Imagemagik "composite"
def apply_watermark(thumb, watermark)
  sh "composite -gravity south -geometry +0+10 #{watermark} #{thumb} #{thumb}"
end

# Copy metadata from original to thumb using "exiftool"
def copy_metadata(original, thumb)
  sh "exiftool -overwrite_original -tagsFromFile #{original} #{thumb}"
end


# Tasks to create thumbnails ---------------------------------------------------

directory ORIGINALS_DIR
directory LARGE_THUMBS_DIR
directory MEDIUM_THUMBS_DIR
directory SMALL_THUMBS_DIR

# Strips unnecessary metadata from images using exiftool
task :strip_metadata do
  sh "exiftool #{ET_STRIP_METADATA} -ext #{ORIGINALS_EXT} -overwrite_original #{ORIGINALS_DIR}"
end

# Generate file tasks for large, medium and small thumbnails
ORIGINALS.each do |original|

  # Original and thumbnails file names
  original_file_name   = original.pathmap('%n')
  normalized_file_name = normalize_for_web(original_file_name)
  normalized_original  = File.join(ORIGINALS_DIR,     normalized_file_name).ext(ORIGINALS_EXT)
  large_thumb          = File.join(LARGE_THUMBS_DIR,  normalized_file_name + LARGE_THUMB_LABEL).ext(THUMBS_EXT)
  medium_thumb         = File.join(MEDIUM_THUMBS_DIR, normalized_file_name + MEDIUM_THUMB_LABEL).ext(THUMBS_EXT)
  small_thumb          = File.join(SMALL_THUMBS_DIR,  normalized_file_name + SMALL_THUMB_LABEL).ext(THUMBS_EXT)

  # Normalizes the original for the web
  task :normalize_originals do
    File.rename(original, normalized_original) if original != normalized_original
  end

  # Task for large thumbnail
  desc "Create large thumbnail for #{original_file_name}"
  file large_thumb => [original, LARGE_WATERMARK, LARGE_THUMBS_DIR] do
    Rake::Task['normalize_originals'].invoke
    Rake::Task['strip_metadata'].invoke
    create_thumb(normalized_original, large_thumb, LARGE_THUMB_DIM)
    apply_watermark(large_thumb, LARGE_WATERMARK)
    copy_metadata(normalized_original, large_thumb)
  end

  # Task for medium thumbnail
  desc "Create medium thumbnail for #{original_file_name}"
  file medium_thumb => [original, MEDIUM_WATERMARK, MEDIUM_THUMBS_DIR] do
    Rake::Task['normalize_originals'].invoke
    Rake::Task['strip_metadata'].invoke
    create_thumb(normalized_original, medium_thumb, MEDIUM_THUMB_DIM)
    apply_watermark(medium_thumb, MEDIUM_WATERMARK)
    copy_metadata(normalized_original, medium_thumb)
  end

  # Task for small thumbnail
  desc "Create small thumbnail for #{original_file_name}"
  file small_thumb => [original, SMALL_THUMBS_DIR] do
    Rake::Task['normalize_originals'].invoke
    Rake::Task['strip_metadata'].invoke
    create_thumb(normalized_original, small_thumb, SMALL_THUMB_DIM)
    copy_metadata(normalized_original, small_thumb)
  end

  # Default task creates large, medium and small thumbnails for all the originals
  task :default => [large_thumb, medium_thumb, small_thumb]

end

# For the first run default task create the orginals directory
task :default => ORIGINALS_DIR


# Utility methods to create watermarks -----------------------------------------
# See http://www.imagemagick.org/Usage/annotating/#wmark_text

# Creates watermark foreground image
def create_fg_watermark(width, height, font, fontsize, text, name)
  sh "convert -size #{width}x#{height} xc:grey30 -font #{font} -pointsize #{fontsize} -gravity center -draw \"fill grey70 text 0,0 '#{text}'\" #{name}_fgnd.png"
end

# Creates watermark mask image
def create_mask_watermark(width, height, font, fontsize, text, name)
  sh "convert -size #{width}x#{height} xc:black -font #{font} -pointsize #{fontsize} -gravity center -draw \"fill white text 1,1 '#{text}' text 0,0 '#{text}' fill black text -1,-1 '#{text}'\" +matte #{name}_mask.png"
end

# Composes foreground and mask images
def compose_watermark(name)
  sh "composite -compose CopyOpacity #{name}_mask.png #{name}_fgnd.png #{name}.png"
end

# Trims compose watermark image
def trim_watermark(name)
  sh "mogrify -trim +repage #{name}.png"
end


# Tasks to create watermarks ---------------------------------------------------

desc "Creates watermark for large thumbnails"
file LARGE_WATERMARK_NAME.ext('png') do
  create_fg_watermark(LARGE_WATERMARK_WIDTH, LARGE_WATERMARK_HEIGHT, LARGE_WATERMARK_FONT, LARGE_WATERMARK_FONTSIZE, LARGE_WATERMARK_TEXT, LARGE_WATERMARK_NAME)
  create_mask_watermark(LARGE_WATERMARK_WIDTH, LARGE_WATERMARK_HEIGHT, LARGE_WATERMARK_FONT, LARGE_WATERMARK_FONTSIZE, LARGE_WATERMARK_TEXT, LARGE_WATERMARK_NAME)
  compose_watermark(LARGE_WATERMARK_NAME)
  trim_watermark(LARGE_WATERMARK_NAME)
end

desc "Creates watermark for medium thumbnails"
file MEDIUM_WATERMARK_NAME.ext('png') do
  create_fg_watermark(MEDIUM_WATERMARK_WIDTH, MEDIUM_WATERMARK_HEIGHT, MEDIUM_WATERMARK_FONT, MEDIUM_WATERMARK_FONTSIZE, MEDIUM_WATERMARK_TEXT, MEDIUM_WATERMARK_NAME)
  create_mask_watermark(MEDIUM_WATERMARK_WIDTH, MEDIUM_WATERMARK_HEIGHT, MEDIUM_WATERMARK_FONT, MEDIUM_WATERMARK_FONTSIZE, MEDIUM_WATERMARK_TEXT, MEDIUM_WATERMARK_NAME)
  compose_watermark(MEDIUM_WATERMARK_NAME)
  trim_watermark(MEDIUM_WATERMARK_NAME)
end

