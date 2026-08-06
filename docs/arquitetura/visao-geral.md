Smart-15 v2.0 (Geo-Analytics Metrics)

Smart-15 revolutionizes the "15-Minute City" concept by moving beyond simple Euclidean distance calculations. Instead, this tool introduces a humanized movement physics model, combining the road network with satellite environmental data and artificial intelligence algorithms to calculate the real effort and thermal comfort of pedestrians and cyclists.

 What’s New in This Version

In version 2.0, the routing engine and raster processing have been completely rebuilt to deliver surgical precision and cloud automation:

️ Cloud-Native Integration (LST): Direct connection to Microsoft Planetary Computer for automatic and free extraction of Surface Temperature (Landsat 9 Band 10).

⛰️ Terrain and Tobler’s Equation (DEM): Rigorous geodetic scaling (degrees to real meters) using Copernicus data. Actual slope dynamically slows or speeds up individuals on the OpenStreetMap network.

 AI Engine (Smart OBIA): New object-based classification algorithm (Decision Tree Classifier) that rasterizes thousands of pixels from user samples. It classifies asphalt, buildings, and trees at scale to inject the Shading Factor (FIS) into the route.

️ Anti-NaN Shield: Advanced mathematical protection that prevents routing mesh failure in areas of missing satellite data at the edges.

️ Highlight Features

 Multi-Profile Simulation: Inclusive accessibility. The algorithm respects different speeds and thermal tolerance limits for Adults, Children, and Seniors (walking or cycling).

️ Tessellation and Isochrones: Simultaneous export of continuous time areas (Isochrones) and hexagonal statistical aggregations (Uber H3 Grid).

 Penalty Fill: Areas beyond the 15-minute radius are not left empty; they are actively mapped as red exclusion/penalty zones (> 15 min).

 High-Level Interactive Visualization:

X-Ray Identifier: Hover to read AI classes, POI categories, and routing times.

Comparison Curtain (Swipe): Analyze scenarios by splitting the screen in half.

Dynamic Legends: Exact values (vmin/vmax) extracted directly from satellite data for Plasma ramps (Slope) and Nipy Spectral (Elevation).

Secure Export: Native generation of .ZIP (SHP, GeoJSON, KML) and integrated Screenshot capture tool.

 Technology Stack

Backend (Python Engine): FastAPI, GeoPandas, OSMnx, NetworkX, Rasterio, Scikit-learn (Decision Trees), pystac-client, and stackstac.

Frontend (Panel): Plain JavaScript, WebGL/MapLibre rendering for 3D extrusion and GeoJSON source handling.

 Quick Installation (Backend)

To ensure the cloud engine works properly, install the dependencies:

Bash

pip install fastapi uvicorn geopandas osmnx networkx rasterio scikit-learn pystac-client planetary-computer stackstac rioxarray h3

Start the server:

Bash

uvicorn main:app --reload --port 8000

Developer Note: This project combines cartography, physics, and urban health.
