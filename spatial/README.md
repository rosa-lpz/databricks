# Databricks SQL

## Get site_column using longitude and latitude columns of a table 

To look up or match a `site_column` (like a site name or site ID) based on latitude and longitude coordinates in Databricks SQL, the approach depends on how your site reference data is stored.

Here are the two standard ways to do this in Databricks SQL:

### Spatial Join with Polygon Geometries (Most Precise)

If you have a reference table (`sites_geo`) containing site boundaries/polygons and you want to match points that fall inside those polygons, use Databricks' native ST functions (`ST_Point` and `ST_Contains`):  

```SQL
SELECT 
  p.longitude,
  p.latitude,
  s.site_name AS site_column
FROM point_table p
LEFT JOIN sites_geo s
  ON ST_Contains(
       s.geometry,                            -- Polygon geometry column in sites_geo
       ST_Point(p.longitude, p.latitude)      -- Constructs a Point geometry (X=lon, Y=lat)
     );
```

Note: In `ST_Point(x, y)`, **longitude** comes first as $X$ and **latitude** as $Y$.



### Nearest Point / Minimum Distance Search

If your site reference table (`sites`) has site locations defined as single latitude/longitude points (e.g., store centers or towers), you can find the closest site using `ST_DistanceSphere` or `ST_DistanceSpheroid`:

```sql
WITH DistanceCalculations AS (
  SELECT 
    p.longitude,
    p.latitude,
    s.site_name AS site_column,
    -- Calculates spherical distance in meters between the two points
    ST_DistanceSphere(
      ST_Point(p.longitude, p.latitude),
      ST_Point(s.longitude, s.latitude)
    ) AS distance_meters,
    ROW_NUMBER() OVER (
      PARTITION BY p.longitude, p.latitude 
      ORDER BY ST_DistanceSphere(
        ST_Point(p.longitude, p.latitude),
        ST_Point(s.longitude, s.latitude)
      ) ASC
    ) AS rank
  FROM point_table p
  CROSS JOIN sites s
)
SELECT 
  longitude,
  latitude,
  site_column,
  distance_meters
FROM DistanceCalculations
WHERE rank = 1;
```



### Fast Point Indexing via H3 Hexagons (For Scalability)

If you are running performance-heavy operations over large datasets (millions of rows), use Databricks' built-in **H3 grid functions**:



```sql
SELECT 
  p.longitude,
  p.latitude,
  s.site_name AS site_column
FROM point_table p
LEFT JOIN sites s
  -- Converts (lat, lon) to a hexagonal H3 cell ID at resolution 8 (~0.7 km²)
  ON h3_longlatash3(p.longitude, p.latitude, 8) = h3_longlatash3(s.longitude, s.latitude, 8);
```

