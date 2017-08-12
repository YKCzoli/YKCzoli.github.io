thought process.
Ive always wanted to make an elevation map of all of canada, like the ones you see in your elementary school geography rooms. so im going to step through the process, including all of my mistakes on a week by week basis until i get there.
i want to process a lot of data. and i want to process all of it. what do i need to know?

Visualizing canadian elevation data.

the data: cded located at:


http://geogratis.gc.ca/api/en/nrcan-rncan/ess-sst/3A537B2D-7058-FCED-8D0B-76452EC9D01F.html#distribution

http://ftp.geogratis.gc.ca/pub/nrcan_rncan/elevation/geobase_cded_dnec/50k_dem/

and

http://ftp.geogratis.gc.ca/pub/nrcan_rncan/elevation/geobase_cded_dnec/250k_dem/

note about 50k and 250k comparing to zoom levels

now what i want to know. im going to go with the 50 k. why? Because im going to process this all on a cloud service, so in theory it shouldnt make a whole big difference between the two

use cyber duck to know the size of all the data.

connect to the ftp. and check the folder size in finder.

ok so its like 20 gb. but how big is that when uncompressed?

connect to ftp using bash:

ftp

open ftp.geogratis.gc.ca

user: anonymous

I installed lftp and ran:

echo "du -hs ." | lftp http://ftp.geogratis.gc.ca/pub/nrcan_rncan/elevation/geobase_cded_dnec/50k_dem/ 2>&1

to get uncompressed file size. probably an easier way, but hey, were making a map.

download NTS index file:

http://geogratis.gc.ca/api/en/nrcan-rncan/ess-sst/dc8449c6-242b-4892-b0ee-dca406de1f66.html#distribution

pick and area of interest, perhaps 4-6 tiles and download only those files. we will work with these files locally to test our scripts before pushing to a cloud environment to process the data.

I chose 022n01, 022n02, 022n03, 022n06, 022n07, 022n08, 022n09, 022n10, 022n11, which correspond with the Manicouagan crater.

Were going to want to use bash scripts in our cloud enviorment. i think. so replicate the folder structre with our 6 folders

unzip

find <path_of_your_zips> -type f -name "*.zip" -exec unzip {} -d <out> \;

cd unzippped

mkdir dem

find . -type f -name '*.dem' -exec mv -i {} ./dem \;

cd dem

gdalbuildvrt vrt.vrt *.dem

gdalwarp -t_srs epsg:4326 -r bilinear -of GTiff vrt.vrt vrt_4326.vrt

gdaldem hillshade vrt_4326.vrt hillshade.tif -z 1.0 -s 111120.0 -az 315.0 -alt 45.0 -of GTiff
