# Module 1 Homework: Docker & SQL

In this homework we'll prepare the environment and practice
Docker and SQL

When submitting your homework, you will also need to include
a link to your GitHub repository or other public code-hosting
site.

This repository should contain the code for solving the homework.

When your solution has SQL or shell commands and not code
(e.g. python files) file format, include them directly in
the README file of your repository.


## Question 1. Understanding Docker images

Run docker with the `python:3.13` image. Use an entrypoint `bash` to interact with the container.

What's the version of `pip` in the image?

- 25.3

`
PS G:\Zoomcamp\Week 1> docker run -it --entrypoint bash taxi_ingest:v001
root@61b3c934b22e:/code# pip --version
pip 25.3 from /usr/local/lib/python3.13/site-packages/pip (python 3.13)
`

## Question 2. Understanding Docker networking and docker-compose

Given the following `docker-compose.yaml`, what is the `hostname` and `port` that pgadmin should use to connect to the postgres database?

```yaml
services:
  db:
    container_name: postgres
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: 'postgres'
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_DB: 'ny_taxi'
    ports:
      - '5433:5432'
    volumes:
      - vol-pgdata:/var/lib/postgresql/data

  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: "pgadmin@pgadmin.com"
      PGADMIN_DEFAULT_PASSWORD: "pgadmin"
    ports:
      - "8080:80"
    volumes:
      - vol-pgadmin_data:/var/lib/pgadmin

volumes:
  vol-pgdata:
    name: vol-pgdata
  vol-pgadmin_data:
    name: vol-pgadmin_data
```

- db:5433
- db:5432

If multiple answers are correct, select any 


## Prepare the Data

Download the green taxi trips data for November 2025:

```bash
wget https://d37ci6vzurychx.cloudfront.net/trip-data/green_tripdata_2025-11.parquet
```

You will also need the dataset with zones:

```bash
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv
```

## Question 3. Counting short trips

For the trips in November 2025 (lpep_pickup_datetime between '2025-11-01' and '2025-12-01', exclusive of the upper bound), how many trips had a `trip_distance` of less than or equal to 1 mile?

- 8,007

`
SELECT COUNT(*) AS short_trips
FROM green_taxi_trips
WHERE lpep_pickup_datetime >= '2025-11-01'
  AND lpep_pickup_datetime < '2025-12-01'
  AND trip_distance <= 1;
`

## Question 4. Longest trip for each day

Which was the pick up day with the longest trip distance? Only consider trips with `trip_distance` less than 100 miles (to exclude data errors).

Use the pick up time for your calculations.

- 2025-11-14

`
WITH daily_max AS (
    SELECT 
        DATE(lpep_pickup_datetime) AS pickup_day,
        MAX(trip_distance) AS max_trip_distance
    FROM green_taxi_trips
    WHERE trip_distance < 100
    GROUP BY DATE(lpep_pickup_datetime)
)
SELECT *
FROM daily_max
ORDER BY max_trip_distance DESC
LIMIT 1;
`

## Question 5. Biggest pickup zone

Which was the pickup zone with the largest `total_amount` (sum of all trips) on November 18th, 2025?

- East Harlem North

`
SELECT
    z."Borough",
    z."Zone",
    SUM(t.total_amount) AS total_revenue
FROM green_taxi_trips t
JOIN taxi_zone_lookup z
    ON t."PULocationID" = z."LocationID"
WHERE t.lpep_pickup_datetime >= '2025-11-18'
  AND t.lpep_pickup_datetime <  '2025-11-19'
GROUP BY z."Borough", z."Zone"
ORDER BY total_revenue DESC
LIMIT 1;
`

## Question 6. Largest tip

For the passengers picked up in the zone named "East Harlem North" in November 2025, which was the drop off zone that had the largest tip?

Note: it's `tip` , not `trip`. We need the name of the zone, not the ID.

- Yorkville West

`
SELECT
    zdo."Borough" AS dropoff_borough,
    zdo."Zone" AS dropoff_zone,
    MAX(t.tip_amount) AS max_tip
FROM green_taxi_trips t
JOIN taxi_zone_lookup zpu
    ON t."PULocationID" = zpu."LocationID"
JOIN taxi_zone_lookup zdo
    ON t."DOLocationID" = zdo."LocationID"
WHERE zpu."Zone" = 'East Harlem North'
  AND t.lpep_pickup_datetime >= '2025-11-01'
  AND t.lpep_pickup_datetime <  '2025-12-01'
GROUP BY zdo."Borough", zdo."Zone"
ORDER BY max_tip DESC
LIMIT 1;
`

## Terraform

In this section homework we'll prepare the environment by creating resources in GCP with Terraform.

In your VM on GCP/Laptop/GitHub Codespace install Terraform.
Copy the files from the course repo
[here](../../../01-docker-terraform/terraform/terraform) to your VM/Laptop/GitHub Codespace.

Modify the files as necessary to create a GCP Bucket and Big Query Dataset.


## Question 7. Terraform Workflow

Which of the following sequences, respectively, describes the workflow for:
1. Downloading the provider plugins and setting up backend,
2. Generating proposed changes and auto-executing the plan
3. Remove all resources managed by terraform`

Answers:

- terraform init, terraform apply -auto-approve, terraform destroy

