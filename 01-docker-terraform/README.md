## Module 1 Homework – Docker & SQL

This repository contains my solutions for **Module 1 (docker-terraform)** of
Data Engineering Zoomcamp 2026.

### Question 1. Understanding Docker images

I ran the following command to check the pip version inside the image:

```
docker run -it --entrypoint=bash python:3.13
pip --version
```

**Answer:** `24.3.1`

### Question 2. Understanding Docker networking

PgAdmin should connect to Postgres using the internal Docker network.

**Answer:** `db:5432`

### Data Preparation

The following datasets were used:

- `green_tripdata_2025-11.parquet`
- `taxi_zone_lookup.csv*`

The queries were executed directly on the Parquet and CSV files using DuckDB.

### Question 3. Counting short trips

Trips with `trip_distance <= 1` mile between
`2025-11-01` (inclusive) and `2025-12-01` (exclusive).

```sql
SELECT COUNT(*)
FROM read_parquet('green_tripdata_2025-11.parquet')
WHERE lpep_pickup_datetime >= TIMESTAMP '2025-11-01'
  AND lpep_pickup_datetime <  TIMESTAMP '2025-12-01'
  AND trip_distance <= 1;
```

**Answer:** `8,007`

### Question 4. Longest trip for each day

Only trips with `trip_distance < 100` miles were considered.

```sql
SELECT DATE(lpep_pickup_datetime) AS pickup_day,
       MAX(trip_distance) AS max_distance
FROM read_parquet('green_tripdata_2025-11.parquet')
WHERE trip_distance < 100
  AND lpep_pickup_datetime >= TIMESTAMP '2025-11-01'
  AND lpep_pickup_datetime <  TIMESTAMP '2025-12-01'
GROUP BY pickup_day
ORDER BY max_distance DESC
LIMIT 1;
```

**Answer:** `2025-11-14`

### Question 5. Biggest pickup zone

Pickup zone with the largest sum of `total_amount` on November 18, 2025.

```sql
SELECT z."Zone", SUM(g.total_amount) AS total_amount
FROM read_parquet('green_tripdata_2025-11.parquet') g
JOIN read_csv_auto('taxi_zone_lookup.csv') z
  ON g.PULocationID = z.LocationID
WHERE DATE(g.lpep_pickup_datetime) = DATE '2025-11-18'
GROUP BY z."Zone"
ORDER BY total_amount DESC
LIMIT 1;
```

**Answer:** `East Harlem North`

### Question 6. Largest tip

For passengers picked up in East Harlem North in November 2025,
the drop-off zone with the largest tip.

```sql
SELECT z2."Zone", MAX(g.tip_amount) AS max_tip
FROM read_parquet('green_tripdata_2025-11.parquet') g
JOIN read_csv_auto('taxi_zone_lookup.csv') z1
  ON g.PULocationID = z1.LocationID
JOIN read_csv_auto('taxi_zone_lookup.csv') z2
  ON g.DOLocationID = z2.LocationID
WHERE z1."Zone" = 'East Harlem North'
  AND g.lpep_pickup_datetime >= TIMESTAMP '2025-11-01'
  AND g.lpep_pickup_datetime <  TIMESTAMP '2025-12-01'
GROUP BY z2."Zone"
ORDER BY max_tip DESC
LIMIT 1;
```

**Answer:** `Yorkville West`

### Question 7. Terraform Workflow

The correct Terraform workflow is:

`terraform init`

`terraform apply -auto-approve`

`terraform destroy`
