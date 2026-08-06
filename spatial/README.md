use databricks sql to get site_column using longitude and latitute of a table

To look up or match a `site_column` (like a site name or site ID) based on latitude and longitude coordinates in Databricks SQL, the approach depends on how your site reference data is stored.

Here are the two standard ways to do this in Databricks SQL:

## Method 1: Spatial Join with Polygon Geometries (Most Precise)

If you have a reference table (`sites_geo`) containing site boundaries/polygons and you want to match points that fall inside those polygons, use Databricks' native ST functions (`ST_Point` and `ST_Contains`):  



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