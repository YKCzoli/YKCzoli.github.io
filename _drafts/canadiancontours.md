1. reproject OR delete columns.

get info
ogrinfo -so contour_1.shp -sql "SELECT * FROM contour_1"

create new shp with only 'e'
ogr2ogr -sql "SELECT e FROM contour_1 WHERE e=40 OR e=80 OR e=120 OR e=160 OR e=200 OR e=240 OR e=280 OR e=320 OR e=360 OR e=480 OR e=520 OR e=580 OR e=620 OR e=660" contour_1_slim.shp contour_1.shp

reproject

ogr2ogr contour_102008.shp -t_srs "EPSG:102008" contour_1_slim.shp

to geojson

ogr2ogr -f "GeoJSON" -lco COORDINATE_PRECISION=2 contour.json contour_1_slim.shp
