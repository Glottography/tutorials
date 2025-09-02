# Glottography Tutorials

This tutorial series walks you through the five main steps to create **language speaker area polygons** from a language map image, ready for upload to [**Glottography**](https://github.com/Glottography)*. 

**Glottography is an open-source initiative to collect and share the geographic areas of the world’s languages as digital open data.*  

## What you need

The only essential input is **a digital raster image of a language map** (e.g., a PNG, JPG, TIFF) from a citable scientific publication. For software, you need [QGIS](https://qgis.org), an open-source geographic information system (GIS), and [Python 3](https://www.python.org/), a free and open-source programming language.

## Output

A set of fully digitised **language polygons** in [Cross-Linguistic Data Format (CLDF)](https://cldf.clld.org/), including attributes and metadata, ready for upload to **Glottography**.  


## From a language map image to a Glottography dataset in five steps

1. [Georeferencing the language map](georeferencing/index.md)  
   Georeferencing assigns geographic coordinates to the map image, allowing it to be accurately placed and displayed in a GIS.

2. [Digitising language polygons in QGIS](digitising/index.md)  
   Digitising traces the language areas in the georeferenced map and converts them into digital polygons. 

3. [Collecting attributes and metadata](metadata/index.md)  
   Learn how to record the attributes and metadata required when digitising language areas from source publications. 
   
4. [Finding Glottocodes](glottocodes/index.md)  
   Glottocodes are unique identifiers for languages maintained by Glottolog. When the source map does not include them, this tutorial shows how to programmatically add Glottocodes to language polygons based on language names.  

5. [Data curation](curation/index.md)  
   Combine the digitised language polygons with their attributes and metadata to create a CLDF dataset ready for upload to Glottography.  


