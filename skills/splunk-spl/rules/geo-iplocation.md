---
title: iplocation Command
impact: MEDIUM
tags: geo, iplocation, geolocation, country, lat, lon
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-i/iplocation"
---

## iplocation Command

`| iplocation` enriches events with geographic data (country, city,
latitude, longitude) based on an IP address field.

**Basic usage:**
```spl
index=web_logs
| iplocation src_ip
| stats count by Country
```
Adds `Country`, `City`, `Region`, `lat`, `lon` fields for each `src_ip`.

**Get all available fields:**
```spl
index=web_logs
| iplocation allfields=true src_ip
| table src_ip, Country, Region, City, lat, lon, MetroCode, Timezone
```

**Incorrect (wrong field name):**
```spl
index=web_logs
| iplocation ip_address
```
Default field for iplocation is `clientip`. Specify field name explicitly.

**Notes:**
- Default input field: `clientip` (if no field argument given).
- Output fields: `Country`, `Region`, `City`, `lat`, `lon` (always added).
- `allfields=true` — also adds `MetroCode`, `Timezone`, `DmaCode`.
- `lang=en` — language for location names (default English).
- External DB: Configure `iplookups.conf` to point to a custom MaxMind DB.
- Events with RFC1918 (private) or IPv6 addresses return null geo fields.
- See: `geo-geostats.md` for map visualization.
