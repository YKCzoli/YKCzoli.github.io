---
layout: post
title:  "Dead Pixels and Georeferencing"
date:   2017-03-25
categories: written_thoughts
---

When working with scanned aerial imagery, I often come across a portion of the image that isn't map friendly. This area is typically the border around the scanned image. This post will step through how to remove this unwanted area by isolating pixels based on their value. I assume you are familiar with [QGIS](http://www.qgis.org/en/site/), [GDAL](http://www.gdal.org/), and the command line.

### Download and view imagery

The University of Toronto Map & Data Library holds a [great collection](https://mdl.library.utoronto.ca/collections/air-photos/air-photo-collection-listing) of aerial imagery from all over Canada. We will work with some imagery covering [Toronto from 1947](https://mdl.library.utoronto.ca/air-photos/1947).

Pick any four connected scenes to download. I chose these four:

![1947_city_of_toronto_air_photos___map_and_data_library](https://cloud.githubusercontent.com/assets/7583912/24066664/f0a07ede-0b4a-11e7-8ca1-1cede292dcd4.jpg)

Open up the .jpg files in each scene within QGIS. Depending on which files you opened in which order, you will see something similar to this:

![screen shot 2017-03-17 at 7 56 32 pm](https://cloud.githubusercontent.com/assets/7583912/24066782/e043f0b0-0b4b-11e7-89ac-de09bddf15b1.png)

The problem is that the border has been scanned in along with the image. If we try to merge these images, the border will be included in the merge and will cover a part of the merged image, similar to how the the images are rendered in QGIS with the overlap:

![qgis_2_14_3-essen](https://cloud.githubusercontent.com/assets/7583912/24066840/3809b596-0b4c-11e7-8a86-e00ecca57a61.jpg)

Almost always, the border is a different colour than the information within the image. A good way to think about the process of removing this border is to think of the image as a collection of individual pixels, each of which hold a value. Different pixel values render different colours. We can get a sense of the values of the image by viewing the metadata and raster stats of the image.

Within the Metadata tab in QGIS we can find the Properties area. Let's take two pieces of information here to breakdown.

### No Data

Any pixel in the image that whose value matches the no data value will be transparent. We can set the no data value on an image, making unwanted pixels transparent. Currently the no data value is 0.


### Data Type

The data type tells us the range of values we can expect in our data. In the GIS industry you may see this referred to as [radiometric resolution](http://www.nrcan.gc.ca/node/9379). These images are 8 bit unsigned integer. For our purposes this means the values will range from 0 to 255.

### Identifying unwanted pixels

Select an image in the table of contents, and activate the Identify Features tools. By clicking around the image you can get a sense of the values of the pixels.
Click on various parts of the border in the image to get a sense of the range of values we would like to remove from the image.

![qgis_2_14_3-essen_and_2017-03-12-removingunwantedpixels_md_ ___ykczoli_github_io__posts](https://cloud.githubusercontent.com/assets/7583912/24067438/f41f0618-0b52-11e7-9573-3c8e6823b7f1.jpg)

The border values consist of pixels with values 240 and higher. You may notice that this value of 240 is also present within the image, and also that there are outlier values within the border, such as scale labels, or slightly darker areas from an uneven scan. Unfortunately these artifacts will remain in the final image, but are much less noticeable than the border area since the backdrop can simply be set to white. Additionally they do add a rustic look to imagery, sometimes to the benefit of the map.

### Raster Calculator

With the raster calculator we can change values that meet a condition in an image. From investigating the image I chose 240 as a threshold. We will set all values above 240 to 0:

![screen shot 2017-03-18 at 12 32 29 pm](https://cloud.githubusercontent.com/assets/7583912/24073982/0e542bb0-0bd7-11e7-86c8-ecc0fb91d7de.png)

After we run the raster calculator:

![screen shot 2017-03-18 at 1 21 10 pm](https://cloud.githubusercontent.com/assets/7583912/24074377/c858c75e-0bdd-11e7-83fa-1bca7f73a880.png)


And set the no data value to 0:

![screen shot 2017-03-18 at 1 21 54 pm](https://cloud.githubusercontent.com/assets/7583912/24074380/def4cd64-0bdd-11e7-8258-1ad26d5b0d79.png)


And finally an improvement on our original image:

![screen shot 2017-03-18 at 1 22 17 pm](https://cloud.githubusercontent.com/assets/7583912/24074385/eeb84528-0bdd-11e7-803a-ea7021842d5f.png)

This process can be replicated for the other images and finally merged into one image. However at this point you will want to automate this process as much as possible, as depending on the number of images it may take an unreasonable amount of time to process your data with the raster calculator.


### Batch processing with GDAL

Rather than manually process these images, we can use GDAL and the command line to batch process all of our images.

Here is an example of a bash script that uses GDAL to apply the above process on all jpg files within a directory.

``` bash
#!/bin/bash
output_location="~/nodata_"
for f in ~/*.jpg;
do
  if $f ==
  gdal_calc.py -A $f --A_band=1 --outfile=$output_location${f:(-11)} --calc="A*(A<240)" --NoDataValue=0
done
```

The variable 'f' is shorthand for 'file'. We are looping through all jpg file types, and running [gdal_calc.py](http://www.gdal.org/gdal_calc.html) on the file. -A is the flag for the file name. --A is the flag for which band to use(here we explicitly use band 1). The --calc flag is our raster calculator expression, and we set the noData value to 0.

This is the same process we applied with the raster calculator, where we substitute GDAL for our UI based approach in QGIS. The benefit of processing the files this way is that we can process the entire repository of imagery with no additional manual work. For an example of how a stitched image covering a large area might look after this process, see the [1954 Toronto map](https://yuriyczoli.com/written_thoughts/2017/03/06/Toronto1954.html).
