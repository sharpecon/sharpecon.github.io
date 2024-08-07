```python
# Load libraries
import pandas as pd
import numpy as np

import datetime
from tqdm import tqdm 
import time
import os

import matplotlib.pyplot as plt
import seaborn as sns

import ee
import hydrafloods as hf
from hydrafloods import corrections
import folium
import geemap
```


```python
# Initialize Google Earth Engine
try:
    ee.Initialize()
except Exception as e:
    ee.Authenticate()
    ee.Initialize()
```



<style>
    .geemap-dark {
        --jp-widgets-color: white;
        --jp-widgets-label-color: white;
        --jp-ui-font-color1: white;
        --jp-layout-color2: #454545;
        background-color: #383838;
    }

    .geemap-dark .jupyter-button {
        --jp-layout-color3: #383838;
    }

    .geemap-colab {
        background-color: var(--colab-primary-surface-color, white);
    }

    .geemap-colab .jupyter-button {
        --jp-layout-color3: var(--colab-primary-surface-color, white);
    }
</style>




```python
# Load shapefiles
ama = ee.FeatureCollection('projects/ee-chinmaya/assets/AMA_shapefile')
gama = ee.FeatureCollection('projects/ee-chinmaya/assets/GAMA_shapefile')

# Create a geometry out of the polygon 
ama_geom = ama.geometry()
gama_geom = gama.geometry()
```


```python
# Define start and end times
year1 = "2021"
year2 = "2021"
month1 = "01" 
month2 = "03" 
start_time = f"{year1}-{month1}-01"
end_time = f"{year2}-{month2}-31"
```


```python
# Convert dates to date range
date_range = ee.DateRange(ee.Date(start_time), ee.Date(end_time))

# Getting GPM IMERG images
temp = ee.ImageCollection("NASA/GPM_L3/IMERG_V07") \
            .filterDate(date_range) \
            .filterBounds(regiongeom) \
            .select("precipitation")

# Calculating mean of the image over the area
bandmean = temp.map(calcmean)
# Removing data for dates which have NULL values
bandmean = bandmean.filter(ee.Filter.notNull(['precipitation']))
# Making time series of the region mean
bandmean_df = pd.DataFrame({ key:pd.Series(value) for key, value in fc_to_dict(bandmean).getInfo().items() })
```


```python
plt.plot(bandmean_df['precipitation'])
plt.show()
```


```python
# Define the center of our map.
lat, lon = 5.62, -0.19759

my_map = folium.Map(location=[lat, lon], zoom_start=12)
my_map
```


```python
# Define start and end times
year1 = "2021"
year2 = "2021"
month1 = "01" 
month2 = "01" 
start_time = f"{year1}-{month1}-01"
end_time = f"{year2}-{month2}-31"
```


```python
# Define the elevation dataset we want to calculate corrections from
elv = ee.Image("JAXA/ALOS/AW3D30/V3_2").select("AVE_DSM")
```


```python
# Use MERIT Hydro dataset for elevation data
merit = ee.Image("MERIT/Hydro/v1_0_1");

# Extract out the DEM and HAND bands
dem = merit.select("elv").unmask(0)
hand = merit.select("hnd").unmask(0)
```


```python
# Load Sentinel data
s1D = ee.ImageCollection("COPERNICUS/S1_GRD") \
        .filterBounds(ama_geom) \
        .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VH')) \
        .filter(ee.Filter.eq('instrumentMode', 'IW')) \
        .filterDate(start_time, end_time) \
        .filter(ee.Filter.eq('orbitProperties_pass', 'ASCENDING'))
```


```python
def clip_to_roi(image):
  return image.clip(ama_geom)
    
clipped_s1D = s1D.map(clip_to_roi)
```


```python
s1 = hf.Dataset.from_imgcollection(s1D)
# s1 = hf.datasets.Sentinel1(regiongeom, start_time, end_time)
# Conduct analysis
water = s1.pipe((
             # Apply speckle filter
             (hf.lee_sigma, dict(window = 5)),
             # Apply slope correction 
             (corrections.slope_correction, dict(elevation = dem, buffer = 50, model = "surface")),
             # Apply thresholding algorithm to classify water
             (hf.fuzzy_otsu, dict(band='VH', initial_threshold=-20, scale = 30))
             ))
        
# Undo hydrafloods wrapper
watercollection = water.collection
```


```python
m = geemap.Map()
m.set_center(-0.19759, 5.62, 12)
m.add_layer(watercollection.first().clip(ama_geom), {'min': 0, 'max': 1}, 'Multi-T Mean ASC', True)
# m.add_layer(desc_change, {'min': -25, 'max': 5}, 'Multi-T Mean DESC', True)
m
```


```python
### Example code from hydrafloods documentation


# specify start and end time as well as geographic region to process
start_time = "2019-10-05"
end_time =  "2019-10-12"

# get the Sentinel-1 collection
# the hf.dataset classes performs the spatial-temporal filtering for you
s1 = hf.datasets.Sentinel1(regiongeom, start_time, end_time)

# apply a water mapping function to the S1 dataset
# this applies the "Edge Otsu" algorithm from https://doi.org/10.3390/rs12152469
water_imgs = s1.apply_func(
    hf.thresholding.edge_otsu,
    initial_threshold=-14,
    edge_buffer=300
)

# take the mode from multiple images
# since this is just imagery from one day, it will simply mosaic the images
water_map = ee.Image(water_imgs.collection.mode())

# export the water map
# hf.geeutils.export_image(
#     water_map,
#     regiongeom,
#     "users/chinmayaxavier/water_map_example",
#     scale=30,
# )

m.add_layer(water_map, {'min': 0, 'max': 1}, 'Multi-T Mean ASC', True)
m
```


```python

```
