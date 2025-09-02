This tutorial series walks you through the five main steps to create **language speaker area polygons** from a language map image, ready for upload to [**Glottography**](https://github.com/Glottography)*. 

**Glottography is an open-source initiative to collect and share the geographic areas of the world’s languages as digital open data.*  

### What you need

The only essential input is **a digital raster image of a language map** (e.g., a PNG, JPG, TIFF) from a citable scientific publication. For software, you need [QGIS](https://qgis.org), an open-source geographic information system (GIS), and [Python 3](https://www.python.org/), a free and open-source programming language.

### Output

A set of fully digitised **language polygons** in [Cross-Linguistic Data Format (CLDF)](https://cldf.clld.org/), including attributes and metadata, ready for upload to **Glottography**.  


### Tutorials

1. [Georeferencing](georeferencing/index.md) – Assign geographic coordinates to the language map image so it can be accurately placed and displayed in a GIS.

2. [Digitising](digitising/index.md) – Trace language areas on the georeferenced map and convert them into digital polygons.

3. Adding [Attributes and Metadata](metadata/index.md) – Record language attributes and information from the source publications.

4. [Glottocodes](glottocodes/index.md) – Unique identifiers for languages maintained by Glottolog. This tutorial shows how to programmatically add Glottocodes to language polygons when they are missing from the source map.

5. [Data Curation](curation/index.md) – Combine the digitised language polygons with their attributes and metadata to create a CLDF dataset ready for upload to Glottography.



