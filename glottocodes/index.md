# Finding Glottocodes

In this tutorial, we will find the Glottocodes for the language areas shown on the Alor-Pantar map (Schapper, 2020).  We georeferenced the map in the [Georeferencing tutorial](../georeferencing/index.md), digitised the language areas in the [Digitising tutorial](../digitising/index.md) and recorded attributes and metadata in the [Attributes and Metadata tutorial](../metadata/index.md). 


## Requirements 

**Software:** [Python 3](https://www.python.org/) is a high-level free and open-source programming language. This tutorial uses version 3.12 with the `guess_glottocode` package installed. For installation instructions, see below.

**Data:** A GeoPackage file containing the language polygons (see the [Digitising tutorial](../digitising/index.md)) and attributes (see the [Attributes and Metadata tutorial](../metatdata/index.md)). The Alor–Pantar language polygons, including attribute data, can be downloaded [here](../digitising/out/schapper2020papuan.gpkg).  Note that this file already includes a `glottocode` column, which we will ignore for the purposes of this tutorial.

**API keys:** The `guess_glottocode` package sends requests to a large language model (LLM) provider to find Glottocodes. Currently supported providers are [Google Gemini](https://aistudio.google.com/apikey) and [Anthropic](https://console.anthropic.com/settings/keys). To use either service, you must first create an API key from the provider (see below).


## What is a Glottocode? 

Glottocodes are unique identifiers for languages, dialects, and language families, maintained by [Glottolog](https://glottolog.org).  
The simplest way to find a Glottocode is to look it up manually:

1. Go to Glottolog.  
2. Enter the language name into the search box.  
3. Copy the Glottocode from the resulting page.  

This works fine for a few languages, but it quickly becomes tedious when you need dozens or hundreds of Glottocodes.  
Instead, we can add Glottocodes programmatically using the [`guess_glottocode` package] (https://github.com/derpetermann/guess_glottocode) in Python. The package can guess a language's Glottocode either using a large language model (LLM) via an API or by querying Wikipedia. This tutorial focuses on finding Glottocodes with an LLM.  


## Install the `guess_glottocode` package

The package requires Python 3.12+ and depends on several other packages, including:

- `geopandas` (for handling geospatial data)
- `wptools` (for querying Wikipedia)
- `anthropic` and `google-genai` (for LLM-based lookups)
- `keyring` (for handling API credentials)

All dependencies are listed in `pyproject.toml` and will be installed automatically. To install the package, run:

```bash
pip install git+https://github.com/derpetermann/guess_glottocode.git
```

Full installation guidelines are available in the project's [README file](https://github.com/derpetermann/guess_glottocode/blob/main/README.md).

## API keys

When using a large language model (LLM) to find a Glottocode, the package sends a request to an LLM provider.  Currently supported providers are **Google Gemini** and **Anthropic**. To use either service, you must first create an API key.

- **Google Gemini** - Create an API key at [https://aistudio.google.com/apikey](https://aistudio.google.com/apikey).  
  You'll need a Google account and must be logged in.
- **Anthropic** - Create an API key at [https://console.anthropic.com/settings/keys](https://console.anthropic.com/settings/keys).  
  You'll need to sign up for and log in to an Anthropic account.

Moderate use of the package should not incur third-party API costs, but heavy usage may.

The first time you call `llm.guess_glottocode` with Gemini or Anthropic, the package will prompt you to enter your API key.  The key is stored securely on your local machine via the `keyring` package, so you won't need to enter it again in future sessions.

## Load the data

First, we load the Alor-Pantar GeoPackage file using GeoPandas' `read_file()` function. GeoPandas is a Python library for working with geospatial data in tabular form. Its `read_file()` function imports spatial data into a GeoDataFrame, preserving both attribute data and geometry, including the coordinate reference system (CRS).


```python
import geopandas as gpd
polygons = gpd.read_file("schapper2020papuan.gpkg")
```

We can quickly inspect the GeoDataFrame by displaying the first 10 entries, confirming that it includes a geometry as well as all attributes and metadata recorded in the [Attributes and Metadata tutorial](../metadata/index.md). Note that it already contains a `glottocode` column with entries for most languages; for the purposes of this tutorial, we will ignore it and generate Glottocodes using a large language model.

```python
print(polygons.head(10))
```

       id        name map_name_full  year note                                          geometry 
    0   1        Abui          Map3  2020      MULTIPOLYGON (((124.7554 -8.14784, 124.7538 -8...  
    1   2       Adang          Map3  2020      MULTIPOLYGON (((124.50134 -8.13276, 124.49954 ...
    2   3      Blagar          Map3  2020      MULTIPOLYGON (((124.28606 -8.47974, 124.28184 ... 
    3   4  Bukalabang          Map3  2020      MULTIPOLYGON (((124.24741 -8.25698, 124.27363 ...
    4   5      Di'ang          Map3  2020      MULTIPOLYGON (((124.16262 -8.36901, 124.17733 ...  
    5   6       Hamap          Map3  2020      MULTIPOLYGON (((124.51194 -8.25331, 124.51783 ... 
    6   7      Kabola          Map3  2020      MULTIPOLYGON (((124.50134 -8.13276, 124.50632 ... 
    7   8       Kaera          Map3  2020      MULTIPOLYGON (((124.30453 -8.3351, 124.28255 -...
    8   9       Kafoa          Map3  2020      MULTIPOLYGON (((124.43155 -8.33392, 124.44223 ...  
    9  10      Kamang          Map3  2020      MULTIPOLYGON (((124.8879 -8.16338, 124.8897 -8... 
    
                                                
## Finding a Suitable Glottocode Using an LLM

While large language models (LLMs) can sometimes guess a language's Glottocode from its name, this approach is unreliable. LLMs may hallucinate nonexistent codes or confuse languages with similar names.  A more reliable approach is to 

1. Generate a set of candidate Glottocodes based on a geographic filter
2. Use the LLM to identify the most suitabele Glottocode of these.

The `geo_filter_glottocodes` function searches Glottolog for entries located within a defined spatial area.  In our case, theese are language polygons digitised earlier including a user-defined buffer. The buffer ensures that spatial uncertainty between the map and Glottolog is taken into account when filtering candidates. It is key to finding a matching Glottocode. If the buffer is too broad, it will return too many candidate Glottocodes, leading to a long prompt, higher token costs, and potentially suboptimal results. A small buffer risks excluding the correct Glottocode. Users can also specify the classification level, excluding Glottocodes that refer to dialects or (sub-)families as needed.

The `llm.guess_glottocode` function then sends the candidate list to and LLM via an API, either Google Gemini or Anthropic, and receives the best match.

The helper function `glottocode_per_row` below combines both steps. It iterates over all rows in the GeoDataFrame, applying a 200 km buffer around each language polygon to gather all Glottocodes in that area, regardless of classification level (`level="all"`). Glottolog's classification may not align exactly with that of the Alor-Pantar map. For example, what the map considers a language might be classified as a dialect or subfamily in Glottolog. The function then sends the candidates to the Google Gemini LLM via its API and receives the most suitable Glottocode.


```python
import guess_glottocode.llm as llm
from guess_glottocode.utils import geo_filter_glottocodes

def glottocode_per_row(row):
    # Geographic filter to find candidate Glottocodes
    candidates = geo_filter_glottocodes(language_location=row.geometry,
                                        buffer=200,
                                        level="all")
    # LLM to find the most suitable candidate
    return llm.guess_glottocode(language=row['name'],
                                candidates=candidates,
                                api="gemini")
# Iterate over all rows and apply helper function to each
polygons['unverified_glottocode'] = polygons.apply(glottocode_per_row, axis=1)
```

## Verify the Glottocde match

We can verify Glottocode matches using additional information maintained by Glottolog. Each Glottocode is linked to a GitHub page containing the language's primary name and any alternative names. The `verify_glottocode_guess` function queries that page and checks whether the language name appears as the primary name or among the alternatives. If it does, the function returns `True`; otherwise, it returns `False`.  

The helper function `verify_per_row` below iterates over all rows in the GeoDataFrame with tentative, unverified Glottocodes. It confirms the Glottocode if verified or sets it to `None` if not.

```python
from guess_glottocode.utils import verify_glottocode_guess

def verify_per_row(row):
    return row.unverified_glottocode if (
            verify_glottocode_guess(row['name'],
                                    row['unverified_glottocode'])) else None

polygons['glottocode'] = polygons.apply(verify_per_row, axis=1)
```

After verifying the Glottocodes, we can display the results for each language to see which have confirmed matches.

```python
print(polygons.head[['name', 'glottocode']])
```
                  name   glottocode    
    0             Abui   abui1241
    1            Adang   adan1251
    2           Kabola   kabo1247
    3         Nedebang   nede1245
    4             Reta   rett1240
    5           Kamang   kama1365
    6            Kafoa   kafo1240
    7           Di'ang       None
    8             Kula   kula1280
    9             Klon   kelo1247
    10      Bukalabang   blag1240
    11          Blagar   blag1240
    12           Hamap   hama1240
    13           Manet   mane1265
    14             Sar   sarr1247
    15          Papuna   papu1257
    16           Teiwa   teiw1235
    17         Kiraman   kira1248
    18           Kaera   kaer1234
    19          Sawila   sawi1256
    20             Kui   kuii1254
    21            Tiee   tiay1238
    22           Suboo   sibo1242
    23             Moo       None
    24  Western Pantar   lamm1241
    25         Wersing   wers1238



The approach successfully identified and verified Glottocodes for 18 out of 25 languages. The entries with `None` indicate that no verified Glottocode was found automatically, so these will need to be added manually or through further refinement, such as a larger buffer size.


## Export to file 

Once the Glottocodes are verified, we remove the column `unverified_glottocode` and export the GeoDataFrame to a `GeoPackage` file.

```python
polygons = polygons.drop(columns=["unverified_glottocode"])
polygons.to_file("schapper2020papuan.gpkg", driver="GPKG")
```

We also export the attribute information as a `CSV` file. For details on why this file is needed, see the [Attributes and metadata tutorial](../metadata/index.md).

```python
polygons.drop(columns="geometry").to_csv("schapper2020papuan.csv", index=False)
```

## Output

A GeoPackage file containing the language polygons (see the [Digitising tutorial](../digitising/index.md)), attributes, and Glottocodes (see also the [Attributes and metadata tutorial](../metadata/index.md)).  The Alor–Pantar language polygons, including attribute data and Glottocodes, can be downloaded [here](../digitising/out/schapper2020papuan.gpkg). Note that in this file some Glottocodes were added manually.  

A CSV file containing the attribute and Glottocode data, linked to the digitised polygons via the `id` column. The CSV file for the Alor–Pantar language polygons can be downloaded [here](../metadata/out/schapper2020papuan.csv).
