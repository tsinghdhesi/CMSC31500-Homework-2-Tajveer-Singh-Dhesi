# India Earthquake Explorer

**Live Dashboard:** [India-Earthquake-Dashboard](https://india-earthquake-dashboard.netlify.app/)

## Overview

This dashboard provides an interactive exploration of seismic activity across India from 2000 to 2025, using data from the USGS Earthquake Catalog (M2.5+). Over 1,200 earthquakes that fall within Indian state boundaries are visualized across four linked views arranged in a 2×2, grid. The dashboard is built with Svelte 4 and D3.js.

Each earthquake in the USGS CSV is assigned to an Indian state at load time using `d3.geoContains()` against a simplified GeoJSON of Indian state boundaries. Earthquakes that fall outside all state polygons (offshore events or those in neighboring countries captured by the USGS bounding box query) are excluded.

## Interactive Elements

### Choropleth Map — Earthquake Count by State
The map displays Indian states colored by earthquake count using a sequential yellow-to-red color scale (`d3.interpolateYlOrRd`). The projection is `d3.geoMercator()` fitted to mainland India. State borders are rendered as a separate always-visible overlay layer (black strokes, no fill) so that borders remain visible regardless of filtering. **Interaction:** clicking a state toggles it as selected. Multiple states can be selected simultaneously. Selected states are highlighted; unselected states are dimmed. The color scale domain is fixed to the full (unfiltered) dataset so the legend does not rescale during filtering.

### Magnitude Histogram
A histogram of earthquake magnitudes using `d3.bin()` with approximately 20 bins. A grey background layer shows the full dataset distribution for reference, while the orange foreground bars reflect the currently filtered data. **Interaction:** a `d3.brushX()` on the x-axis lets the user drag to select a magnitude range. Brushing filters all other views to only show earthquakes within that magnitude window.

### Depth vs. Magnitude Scatter Plot
Each earthquake is plotted as a small semi-transparent circle with magnitude on the x-axis and depth (km) on the y-axis, inverted so that deeper earthquakes appear lower on screen. **Interaction:** a 2D `d3.brush()` rectangle can be drawn to select earthquakes by both magnitude and depth simultaneously, filtering the other three views. Hovering over a point shows a tooltip with magnitude, depth, and location.

### Earthquakes by Year Bar Chart
Vertical bars show the count of earthquakes per year from 2000 to 2025. A grey background layer shows the full dataset counts for reference. **Interaction:** clicking a bar toggles that year as selected. Multiple years can be selected. Selected bars are highlighted with a dark stroke; unselected bars are dimmed during an active selection.

## Crossfiltering

All four views are linked through a shared crossfilter. Filters combine as an intersection (AND logic): if the user brushes a magnitude range, selects two states, and clicks a year, only earthquakes matching all three criteria appear. Each view displays data filtered by the other three views' active selections but renders its own selection state independently. A "Reset All Filters" button appears when any filter is active and clears all selections at once.

## Data

- **Earthquake data:** USGS Earthquake Catalog API, bounding box around India, M2.5+, 2000–2025
- **GeoJSON:** Indian state-level boundaries (simplified from original, with clockwise winding order preserved for D3 spherical geometry). State name property is `NAME_1`.

## Collaborators
Shreeya Upadhyay - This is my girlfriend in the Geophysical Sciences Department. She is a Seismology PhD student here at UChicago. She inspired me to use this dataset due to its interesting features variables and geospatial element. She also assisted in advising what might be some interesting aspect to look at for the figures and advised to do count per state over mean magnitude of earthquake by state

Claude.ai - debugging, basic syntax, aestheitic features. 

