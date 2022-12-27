# Geospatial Graph Demos

Map-based demos to showcase geospatial functionality in Neo4j.

* Spatial search
* Routing

## Spatial Search With Neo4j

![](img/spatialsearch.png)

See [`src/index.html`](src/index.html)

These examples use points of interest from the **Daylight Earth Table OpenStreetMap** distribution. [This Python notebook](https://github.com/johnymontana/daylight-earth-graph/blob/main/POI_import.ipynb) has the code to import this data into Neo4j.

[![](img/daylight_notebook.png)](https://github.com/johnymontana/daylight-earth-graph/blob/main/POI_import.ipynb)


### Radius Distance Search

![](img/radius_search.png)

See [`src/index.html`](src/index.html)

### Bounding Box Search

![](img/bounding_box.png)

See [`src/index.html`](src/index.html)

### Point In Polygon Search

![](img/point_in_polygon_search.png)

See [`src/index.html`](src/index.html)

### Line Geometry Search

![](img/linesearch.png)

See [`src/strava.html`](src/strava.html)

![](img/bloom2.png)

Working with Line geometries in Neo4j using Strava data. To import data, first export user data from Strava then to add activities:

```Cypher
// Create Activity Nodes
LOAD CSV WITH HEADERS FROM "file:///activities.csv" AS row
MERGE (a:Activity {activity_id: row.`Activity ID`})
SET a.filename = row.Filename,
    a.activity_type = row.`Activity Type`,
    a.distance = toFloat(row.Distance),
    a.activity_name = row.`Activity Name`,
    a.activity_data = row.`Activity Date`,
    a.activity_description = row.`Activity Description`,
    a.max_grade = toFloat(row.`Max Grade`),
    a.elevation_high = toFloat(row.`Elevation High`),
    a.elevation_loss = toFloat(row.`Elevation Loss`),
    a.elevation_gain = toFloat(row.`Elevation Gain`),
    a.elevation_low = toFloat(row.`Elevation Low`),
    a.moving_time = toFloat(row.`Moving Time`),
    a.max_speed = toFloat(row.`Max Speed`),
    a.avg_grade = toFloat(row.`Average Grade`)

// Parse geojson geometries and create Geometry:Line nodes
MATCH (a:Activity) 
WITH a WHERE a.filename IS NOT NULL AND a.filename CONTAINS ".gpx"
MERGE (n:Geometry {geom_id:a.activity_id })
MERGE (n)<-[:HAS_FEATURE]-(a)
WITH n,a
CALL apoc.load.json('file:///' + replace(a.filename, '.gpx', '.geojson')) YIELD value
UNWIND value.features[0].geometry.coordinates AS coord
WITH n, collect(point({latitude: coord[1], longitude: coord[0]})) AS coords
SET n.coordinates = coords
SET n:Line
```

## Routing

### Airport Routing

![](img/airportrouting.png)

See [`src/airports.html`](src/airports.html)

### OpenStreetMap Road Network Routing

See [`src/osm_routing.html`](src/osm_routing.html)