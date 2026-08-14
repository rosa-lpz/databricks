Absolutely. This SQL finds the **nearest site for every geographic point** in `point_table`.

### 1. The overall idea

Suppose you have:

**`point_table`**

| longitude | latitude |
| --------: | -------: |
|   -112.05 |    33.45 |
|   -111.90 |    33.50 |

and:

**`sites`**

| site_name | longitude | latitude |
| --------- | --------: | -------: |
| Site A    |   -112.00 |    33.40 |
| Site B    |   -111.80 |    33.60 |
| Site C    |   -112.20 |    33.30 |

The query essentially does:

> For each point → calculate its distance to **every site** → rank the sites from closest to farthest → keep only rank 1.

---

## The CTE

```sql
WITH DistanceCalculations AS (
```

`WITH` creates a **Common Table Expression (CTE)**.

Think of `DistanceCalculations` as a temporary result set that exists only while this query runs.

The CTE calculates:

* the point's longitude
* the point's latitude
* the site name
* distance between point and site
* ranking of the sites by distance

---

## Getting the coordinates

```sql
SELECT 
    p.longitude,
    p.latitude,
    s.site_name AS site_column,
```

There are two tables:

```sql
point_table p
sites s
```

`p` and `s` are aliases.

So:

```sql
p.longitude
```

means longitude from `point_table`.

And:

```sql
s.site_name
```

means site name from `sites`.

The alias:

```sql
s.site_name AS site_column
```

renames `site_name` to `site_column` in the output.

---

# Calculating the distance

The most important part is:

```sql
ST_DistanceSphere(
    ST_Point(p.longitude, p.latitude),
    ST_Point(s.longitude, s.latitude)
) AS distance_meters
```

### `ST_Point()`

This converts longitude and latitude into a geographic point.

For example:

```sql
ST_Point(-112.05, 33.45)
```

represents a location at:

```text
longitude = -112.05
latitude  = 33.45
```

So this:

```sql
ST_Point(p.longitude, p.latitude)
```

creates the geographic point from `point_table`.

And:

```sql
ST_Point(s.longitude, s.latitude)
```

creates the geographic point from `sites`.

### `ST_DistanceSphere()`

Then:

```sql
ST_DistanceSphere(point1, point2)
```

calculates the approximate distance between those two geographic coordinates **along the Earth's surface**.

The result is in **meters**.

For example:

```text
Point → Site A = 2,350 meters
Point → Site B = 7,820 meters
Point → Site C = 4,100 meters
```

---

# The `CROSS JOIN`

This is a very important part:

```sql
FROM point_table p
CROSS JOIN sites s
```

A `CROSS JOIN` creates **every possible combination** of points and sites.

For example, if you have:

```text
10,000 points
100 sites
```

you get:

```text
10,000 × 100 = 1,000,000 combinations
```

Conceptually:

| Point   | Site   | Distance |
| ------- | ------ | -------: |
| Point 1 | Site A |  2,350 m |
| Point 1 | Site B |  7,820 m |
| Point 1 | Site C |  4,100 m |
| Point 2 | Site A |  5,200 m |
| Point 2 | Site B |  1,800 m |
| Point 2 | Site C |  3,600 m |

This is necessary because you need to compare each point against **all sites** to determine which site is closest.

---

# Ranking the sites

This is the key SQL window function:

```sql
ROW_NUMBER() OVER (
    PARTITION BY p.longitude, p.latitude 
    ORDER BY ST_DistanceSphere(
        ST_Point(p.longitude, p.latitude),
        ST_Point(s.longitude, s.latitude)
    ) ASC
) AS rank
```

Let's break it down.

### `PARTITION BY`

```sql
PARTITION BY p.longitude, p.latitude
```

means:

> Treat each unique coordinate as a separate group.

For example:

```text
Point 1: (-112.05, 33.45)
Point 2: (-111.90, 33.50)
Point 3: (-112.20, 33.30)
```

Each point gets its own ranking.

---

### `ORDER BY`

```sql
ORDER BY ST_DistanceSphere(...) ASC
```

means:

> Within each point, sort the sites from closest to farthest.

For example:

```text
Point 1

Site A → 2,350 m → rank 1
Site C → 4,100 m → rank 2
Site B → 7,820 m → rank 3
```

`ASC` means ascending order.

---

### `ROW_NUMBER()`

Then SQL assigns:

```text
rank = 1
rank = 2
rank = 3
...
```

So the intermediate result might look like:

| longitude | latitude | site   | distance | rank |
| --------: | -------: | ------ | -------: | ---: |
|   -112.05 |    33.45 | Site A |     2350 |    1 |
|   -112.05 |    33.45 | Site C |     4100 |    2 |
|   -112.05 |    33.45 | Site B |     7820 |    3 |
|   -111.90 |    33.50 | Site B |     1800 |    1 |
|   -111.90 |    33.50 | Site C |     3600 |    2 |
|   -111.90 |    33.50 | Site A |     5200 |    3 |

---

# The final query

```sql
SELECT 
    longitude,
    latitude,
    site_column,
    distance_meters
FROM DistanceCalculations
WHERE rank = 1;
```

This is where you throw away everything except the closest site.

```sql
WHERE rank = 1
```

means:

> Give me only the first/closest site for each point.

The final result becomes:

| longitude | latitude | site_column | distance_meters |
| --------: | -------: | ----------- | --------------: |
|   -112.05 |    33.45 | Site A      |            2350 |
|   -111.90 |    33.50 | Site B      |            1800 |

---

# In plain English

The entire query can be translated to:

> **Take every point, compare it with every site, calculate the geographic distance between them, rank the sites from closest to farthest for each point, and return only the closest site.**

### Data flow

```text
point_table
     │
     │ CROSS JOIN
     ▼
  Every point
  × every site
     │
     ▼
Calculate distance
     │
     ▼
Rank sites for each point
     │
     ▼
Keep rank = 1
     │
     ▼
Nearest site for each point
```

### One important caveat

The query partitions by:

```sql
p.longitude, p.latitude
```

rather than by a unique **point ID**.

If two different records have exactly the same longitude/latitude, they'll be treated as the same point group. If `point_table` has something like `point_id`, it is usually safer to use:

```sql
PARTITION BY p.point_id
```

Also, because `CROSS JOIN` creates `points × sites` rows, this can become expensive with large datasets. For millions of points/sites, a spatial nearest-neighbor approach with a spatial index is generally much more efficient.
