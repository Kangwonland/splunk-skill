---
title: geostats and geom Commands
impact: MEDIUM
tags: geo, geostats, geom, choropleth, map, lat, lon
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-g/geostats"
---

## geostats and geom Commands

`geostats` and `geom` power Splunk's cluster map and choropleth map
visualizations.

**Cluster map (geostats):**
```spl
index=web_logs
| iplocation src_ip
| geostats latfield=lat longfield=lon count by status
```
Renders a cluster map: circles sized by count, colored by status breakdown.

**Choropleth map (geom):**
```spl
index=web_logs
| iplocation src_ip
| stats count by Country
| geom geo_countries featureIdField=Country
```
Colors countries on a world map by count.

**Filter by bounding box (geomfilter):**
```spl
index=web_logs
| iplocation src_ip
| geomfilter min_lat=30 max_lat=60 min_lon=-10 max_lon=40
| geostats latfield=lat longfield=lon count
```

**Notes:**
- `geostats globallimit=N` — max distinct values of `by` field rendered (default 100).
- `geostats maxzoomlevel=N` — controls aggregation granularity at zoom levels.
- `geom geo_countries` — built-in country geometry lookup.
  Other built-ins: `geo_us_states`, `geo_us_counties`.
- Custom KML/KMZ geometry files can be added via lookup configuration.
- `geomfilter` clips events outside a lat/lon bounding box.
