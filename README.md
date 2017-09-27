[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

# Coastal studio
Online, open educational materials for a coastal landscape architecture studio.

<p align="center"><img src="images/sea_level_rise.gif" height="500"></p>

**Resources** [Geospatial data sources](geospatial-data-sources.md)

**Software** | [GRASS GIS](https://grass.osgeo.org) |
[Rhino](https://www.rhino3d.com/) |

---
## Contents
1. [**Assignments**](#terrain-modeling)
    1. [**Site analysis**](#site-analysis)
        1. [**Site visit**](#site-visit)
        2. [**Research**](#research)
        3. [**Laser cut model**](#laser-cut-model)
        4. [**Geospatial analysis and mapping**](#geospatial-analysis-and-mapping)
        5. [**Sections**](#sections)
        6. [**Diagrams**](#diagrams)
    2. [**Site design**](#site-design)
2. [**Tutorials**](#tutorials)
    1. [**Terrain analysis**](#terrain-analysis)
    2. [**Hydrological modeling**](#hydrological-modeling)
---

# Assignments

## Site analysis

### Site visit
For our site visit to Elmer's Island on **10/02**
please bring a camera, your sketchbook,
and as a class a 32"x36" plot with contours
and another 32"x36" plot with orthoimagery.
Include a north arrow, scale, and cartographic grid on your plots.

### Research
Read the
[management plan](http://www.wlf.louisiana.gov/sites/default/files/pdf/refuge/32508-elmers-island-wildlife-refuge/elmers_island_management_plan_final.pdf)
and its [addendum](http://www.wlf.louisiana.gov/sites/default/files/pdf/refuge/32508-elmers-island-wildlife-refuge/draft_elmers_island_management_plan_addendum_031017.pdf).
As a class compile a list of plants, birds, and terrestrial and marine life
on Elmer's Island.
Then create a library of cutout *png* images
for at least 10 key species in each category.  
**Due: 10/06**
* Library of plants
* Library of birds
* Library of terrestrial and marine life

### Laser cut model
As a class build a laser cut contour model of Elmer's Island.
**Due: 10/06**
* Material: museum board (4-ply) or chipboard (1/16" thick)
* Contour interval: 0.25 m
* Tiles: grid of 6 square tiles

### Geospatial analysis and mapping
As groups of 2 compute geospatial analyses for our site.
Work through the
[terrain analysis](README.md#terrain-analysis)
and [hydrological modeling](README.md#hydrological-modeling)
tutorials.
Then design 3 *beautiful* maps
about topics like topography, vegetation, hydrology,
sea level rise, and flood risk.
Include cartographic elements like
legends, labels, scale bars, north arrows, and cartographic grids.
**Due: 10/13**
* 3 maps
* Layout and print all 3 maps on a 36"x72" plot
* Submit digital and print versions

### Sections
Each group will
draw at least 3 sections or section-elevations.
Make the section cuts in GRASS GIS or Rhino
Based on your site visit, the orthoimagery, and your research
illustrate your sections in Photoshop and/or Illustrator
to show plant, bird, and marine communities.
**Due: 10/18**
* 3+ illustrated section-elevations

### Diagrams
Each group will draw an environmental process diagram and a concept diagram.
The first diagram should illustrate how an environmental process
like dune morphology, coastal deposition and erosion,
land loss, or bird migration
functions on Elmer's Island.
The concept diagram should highlight
what aspect of the site interests you most,
explain why it's interesting,
and suggest a solution or intervention
laying the framework for your first design moves.
**Due: 10/18**
* Environmental process diagram
* Concept diagram

## Site design
...

### Concept model
...

# Tutorials

## Terrain analysis
Start GRASS GIS in the `nad83_utm15n_barataria` location
and create a new mapset called `terrain_analysis`.

Set your region to our study area with 2 meter resolution
using the module
[g.region](https://grass.osgeo.org/grass72/manuals/g.region.html)
by specifying a reference raster map.
```
g.region raster=elmers_elevation_2m_2012 res=2
```

### Contours
*Under development*
...

```
r.contour input=elmers_elevation_2m_2012@PERMANENT output=contours_25cm step=0.25 minlevel=0
```

```
r.neighbors -c input=elmers_elevation_2m_2012@PERMANENT output=smoothed_elevation size=9
```

```
r.contour --overwrite input=smoothed_elevation@PERMANENT output=contours_25cm step=0.25 minlevel=0
```
Right click on the `contour_25cm` map layer and select `change opacity level`.
Set the opacity to 30%.

```
r.contour --overwrite input=smoothed_elevation@PERMANENT output=contours_1m step=1 minlevel=0
```
Right click on the `contour_25cm` map layer and select `change opacity level`.
Set the opacity to 60%.


## Hydrological modeling
Start GRASS GIS in the `nad83_utm15n_barataria` location
and create a new mapset called `hydrology`.

Set your region to our study area with 2 meter resolution
using the module
[g.region](https://grass.osgeo.org/grass72/manuals/g.region.html)
by specifying a reference raster map.
```
g.region raster=elmers_elevation_2m_2012 res=2
```

### Inundation modeling
Model current sea level using the module
[r.lake](https://grass.osgeo.org/grass72/manuals/r.lake.html).
Set the water level to 0 meter, i.e. sea level.
In the seed tab use the ![pointer](images/grass-gui/pointer.png)
to pick coordinates somewhere out in the ocean on the map display
for the starting point.
```
r.lake elevation=elmers_elevation_2m_2012@PERMANENT water_level=0 lake=sea_level
```
In the layer manager move the sea level map
above the elevation and contour maps.

### Flow accumulation
Model flow accumulation for our study area using the module
[r.watershed](https://grass.osgeo.org/grass72/manuals/r.watershed.html).
```
r.watershed -a elevation=elmers_elevation_2m_2012@PERMANENT accumulation=flow_accumulation
```

Use map algebra with the GRASS GIS Raster Map Calculator
[r.mapcalc](https://grass.osgeo.org/grass72/manuals/r.mapcalc.html)
to remove cells beneath sea level.
```
r.mapcalc "accumulation = if(isnull(sea_level@PERMANENT),flow_accumulation@hydrology,null())"
```
This expression means
if there are cells where the sea level map is null (ie. has no value),
then write the flow accumulation values,
else write null values.


Copy the color table from the `flow accumulation` map using
[r.colors](https://grass.osgeo.org/grass72/manuals/r.colors.html)
```
r.colors map=accumulation@hydrology raster=flow_accumulation@hydrology
```

To see only the concentrated flow accumulation
hide the cells with values less than `60`
by either
double clicking on the `accumulation` map in the layer manager
and setting the list of values to display to `60-58849.4460216767`
or running the command:
```
d.rast map=accumulation@hydrology values=60-58849.4460216767
```
Experiment to find the right minimum value.

In the layer manager move the `accumulation` map layer
above the `elmers_elevation_2m_2012` and `sea_level` map layers.

Display the flow accumulation legend
by either pressing the
![legend](images/grass-gui/legend-add.png)
`Add raster legend` button
or running the command
[d.legend](https://grass.osgeo.org/grass72/manuals/d.legend.html).
Use the `-n` flag to hide categories
that are not represented in the data.

### Sea level rise animation
Install the `r.lake.series` add-on with
[g.extension](https://grass.osgeo.org/grass72/manuals/g.extension.html).
```
g.extension extension=r.lake.series
```

Model sea level rise over time using the add-on module
[r.lake.series](https://grass.osgeo.org/grass72/manuals/addons/r.lake.series.html).
In the water tab
set the starting water level to 0 m, the end water level to 2 m,
and the water level step to 0.1 m.
Use the ![pointer](images/grass-gui/pointer.png)
to pick coordinates somewhere out in the ocean on the map display
for the starting point.
In the time tab set the time step to 5 years
to model sea level rise over 100 years.
This module will create a time series of raster maps
named `sea_level_rise_0.0`, `sea_level_rise_0.1`, `sea_level_rise_0.2`, etc...
that will all be registered in a space time raster dataset.
```
r.lake.series elevation=elmers_elevation_2m_2012@PERMANENT output=sea_level_rise start_water_level=0 end_water_level=2 water_level_step=0.1 coordinates=787832.669323,3232533.86454 time_step=5 time_unit=years
```

To animate this sea level rise time series launch the GRASS Animation Tool
[g.gui.animation](https://grass.osgeo.org/grass72/manuals/g.gui.animation.html).
If you launch the Animation Tool from the File menu in the GUI
follow the these instructions.
Create a new animation,
add a space-time dataset layer,
choose `space time raster dataset` as the input data type,
and choose the `sea_level_rise` space time raster dataset.
Then add a raster map layer
and choose `elmers_elevation_2m_2012@PERMANENT`.
Move this raster map layer beneath the space time raster dataset.
Check the `Show raster legend` button
and choose `elmers_elevation_2m_2012@PERMANENT`
as the raster map in the d.legend dialog.
Press `Ok` to create to the animation.
Press `Play` to run the animation.
Export your animation as an animated gif.
Press the `Export` button,
select export to `animated GIF`,
then browse and name your file,
and press `Export`.
```
g.gui.animation strds=sea_level_rise
```

















# Submission
Upload digital copies of your work to the class network drive
at `\\desn-knox.lsu.edu\Landscape-Classes` on Windows
or `smb://desn-knox.lsu.edu/Landscape-Classes` on Mac.

# License
CC BY-SA 4.0 by Brendan Harmon. The license does not apply to logos, fonts, linked material, quotations, or reprinted images by other authors, which may have different licenses. The fonts used in this repository are licensed under the SIL Open Font License by their authors. The syllabus is based on a latex template by Kieran Healy hosted at https://github.com/kjhealy/latex-custom-kjh.
