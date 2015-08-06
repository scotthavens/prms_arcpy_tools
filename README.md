gsflow-arcpy-tools
==================

Series of Python/ArcPy (ArcGIS) scripts for developing inputs for a GSFLOW model.

#####Script Execution Order
- fishnet_generator.py 
- hru_parameters.py 
- dem_parameters.py 
- veg_parameters.py 
- soil_raster_pre.py 
- soil_parameters.py 
- impervious_parameters.py 
- prism_4km_normals.py / prism_800m_normals.py 
- ppt_ratio_parameters.py 
- *Iterate to define the stream network*
  - dem_2_streams.py 
  - crt_fill_parameters.py 
- stream_parameters.py 
- prms_template_fill.py 

####Ancillary Data
Almost all of the following data can be downloaded for a study area using the [USGS Geospatial Data Gateway](http://datagateway.nrcs.usda.gov/).  They could probably also be downloaded using the [National Map Viewer](http://viewer.nationalmap.gov/viewer/), but I haven't tried this.  Specific download instructions are provided for each dataset below.
#####Elevation
Elevation data is set using the 10m (1/3 arc-second) or 30m (1 arc-second) National Elevation Dataset (NED) rasters.  These can be easily downloaded in 1x1 degree tiles for the CONUS from the [USGS FTP](rockyftp.cr.usgs.gov) in the folder vdelivery/Datasets/Staged/Elevation.
#####LANDFIRE
Vegetation type and cover percent is set using [LANDFIRE](http://www.landfire.gov/).  The data can be downloaded in [tiles](http://www.landfire.gov/viewer/) or for the entire [CONUS](http://www.landfire.gov/lf_mosaics.php) (but only version 1.3.0?).  The default remap files were developed and tested for version 1.2.0 LANDFIRE 2010 version 1.2.0.
#####Soils
Available water capacity (AWC), percent sand, percent clay, and saturated hydraulic conductivity (Ksat) can be set using SSURGO or STATSGO.  The easiest way to acquire these data are through the [USGS Geospatial Data Gateway](http://datagateway.nrcs.usda.gov/).  Shapefiles of the soil properties can be extracted using the [NRCS Soil Data Viewer](http://www.nrcs.usda.gov/wps/portal/nrcs/detailfull/soils/home/?cid=nrcs142p2_053620).  The shapefiles then need to be converted to raster.
#####PRISM
PRISM precipitation, minimum temperature, and maximum temperature 30 year normals for the CONUS can be downloaded from the [PRISM site](http://www.prism.oregonstate.edu/normals/).
#####CRT
User must have [Cascade Routing Tool](http://water.usgs.gov/ogw/CRT/) (CRT) version 1.1.1.
#####Remap files
Example ASCII remap files are provided, although it may be necessary to modify these to include new or missing LANDFIRE vegetation types.  In versions of ArcGIS before 10.2, you could have comments after the values (indicated by a /*) but this was removed in 10.2.  Now, comments must be on a separate line and begin with the "#" symbol.  The convert_remap_10p2.py script will convert the ArcGIS 10.1 style ASCII remap files to the ArcGIS 10.2 style.


###Example
Create the project folder if necessary (i.e. D:\Projects\sagehen) and create an hru_params sub-folder (i.e. D:\Projects\sagehen\hru_params).
####Clone the repository
If you already have a local repository of the gsflow-arcpy-tools, pull the latest version from GitHub.  If you don't already have a local repository, either clone the repository locally or download a zip file of the scripts from Github.  For this example, the local copy will be cloned directly in the project folder (i.e. D:\Projects\sagehen\scripts and D:\Projects\sagehen\remaps)
####Study Area
Create a "shapefiles" sub-folder (i.e. D:\Projects\sagehen\shapefiles).
Place the study area shapefile in the shapefiles sub-folder  (ie.e. D:\Projects\sagehen\shapefiles\watershed.shp).  The study area is typically a single watershed (like a HUC8) or a collection of watersheds that will be modeled together.  Check the projection of the study area shapefile and make sure it is in a projected coordinate system (like NAD83 UTM Zone 10N).
#####Input File
Copy the template input file (scripts\template_parameters.ini) to the hru_params folder and rename (i.e. D:\Projects\sagehen\hru_params\example_params.ini)
Within the input file, all of the file and folder paths need to be set to the project folder.  A simple find and replace of "D:\Projects\gsflow-arcpy-example" to your project folder (i.e. "D:\Projects\sagehen") should work, but doublecheck all of the paths.  Eventually, all of the paths will be set as relative paths to the main project folder, but for now they must be absolute paths.
##### Fishnet / Model Grid
The following parameters must be explicitly set whether you are building a new fishnet/grid shapefile or reading in an existing one.  If reading an existing grid, the paramters must match the grid properties exactly.  
- hru_cellsize - cellsize (units will depend on projection)
- hru_ref_x: snap x coordinate (units will depend on projection, 0 is a good choice for a new grid)
- hru_ref_x: snap y corodinate (units will depend on projection, 0 is a good choice for a new grid)
- hru_projection: EPSG code (see spatialreference.org)
- study_area_path: The full filepath of your study area shapefile

##### Running the Scripts
Currently, the scripts must be run from the windows command prompt so that the input file can be passed as an argument directly to the script.  Eventually, the scipts will be modified so they can be executed by double clicking on the script in Explorer and the input file will be set through a dialog box.<br>
First open the windows command prompt by pressing the "windows" key and "r" or clicking the "Start" button, "Accessories", and then "Command Prompt".

Within the command prompt, change to the target drive if necessary:
```
> D:
```

Navigate to the project folder:
```
> cd D:\Projects\sagehen
```
Run the fisnet_generator.py script if building a new fishnet.  Note, this will overwrite an existing fishnet shapefile if one already exists at the path specified in the input file.
```
> python scripts\fishnet_generator.py -i hru_params\example_params.ini
```

##### Digitial Elevation Model (DEM)
If you don't already have a DEM raster of the study area, download the DEM tiles using the download_ned.py script.  You must set the study area shapefile path using the "--extent" argument and the output folder using the "--output" argument.  The script will project the shapefile to a geographic coordinate system and download all NED 1x1 degree tiles that intersect the study area.
```
> python tools\download_ned.py --extent shapefiles\watershed.shp --output dem\tiles
```
After downloading the tiles, they need to be merged together into a single image and then projected to the fishnet (or watershed) spatial reference.  This could be done in ArcGIS or using the GDAL utilities (http://www.gdal.org/gdal_utilities.html) at the command line.  

First, the gdal_merge tool can be used to merge the tiles into a single raster.
```
> gdal_merge.py -of HFA -co COMPRESSED=YES -o dem\ned_30m_merge.img  dem\tiles\imgn39w107_1.img  dem\tiles\imgn39w108_1.img  dem\tiles\imgn40w107_1.img  dem\tiles\imgn40w108_1.img
```

It is important that the elevation units match the linear unit of the coordinate system.  If the fishnet is in a coordinate system with feet as the linear unit, scale the NED rasters by 0.3048 to convert meters to feet.  This can be done using the gdal_calc utility
```
gdal_calc.py -A dem\ned_30m_merge.img --outfile=dem\ned_nad83_feet.img --calc="0.3048*A" --format HFA --co COMPRESSED=YES
```

Project the raster to the fishnet (or study area) spatial reference using the "gdalwarp" utility.  
```
gdalwarp -r "bilinear" -tr 30 30 -s_srs "EPSG:4269" -t_srs "EPSG:26910" -overwrite -ot Float32 -srcnodata None -dstnodata -3.4028234663852886e+38 -of HFA -co COMPRESSED=YES dem\ned_30m_merge.img dem\ned_30m.img
```
The output cellsize may need to be adjusted using "-tr" argument for coordinate systems that are not in meters, otherwise 30 should work well for most applications.  The output EPGS code (set with "-t_srs") needs to match the fishnet EPSG set in the input file.  To subset the raster at this step, use the "-te xmin ymin xmax ymax" argument with coordinates in the projected coordinate system of the fishnet.  
#####
