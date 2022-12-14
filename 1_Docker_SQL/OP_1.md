docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v "D:/Work/GIT Bash/Docker_SQL_0/ny_taxi_postgres_data":/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:13

notepad ~/.bashrc

export PATH=$PATH:"/C/Users/7asanen999/AppData/Local/Packages/PythonSoftwareFoundation.Python.3.9_qbz5n2kfra8p0/LocalCache/local-packages/Python39/Scripts"

pgcli -h localhost -p 5432 -u root -d ny_taxi


wget https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2021-01.parquet

>>> import pyarrow.parquet as pq
>>> yellow_tripdata = pq.read_table("yellow_tripdata_2021-01.parquet")
>>> yellow_tripdata = yellow_tripdata.to_pandas()
>>> yellow_tripdata.to_csv("yellow_tripdata_2021-01.csv",index=False)

then notebook used

docker run -it \
  -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
  -e PGADMIN_DEFAULT_PASSWORD="root" \
  -p 8080:80 \
  dpage/pgadmin4

Link postgres container with pgadmin container:
(docker network)

docker network create pg_network

docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v "D:/Work/GIT Bash/Docker_SQL_0/ny_taxi_postgres_data":/var/lib/postgresql/data \
  -p 5432:5432 \
  --network=pg_network \
  --name pgdatabase \ # name to discover this in network
  postgres:13

docker run -it \
  -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
  -e PGADMIN_DEFAULT_PASSWORD="root" \
  -p 8080:80 \
  --network=pg_network \
  --name pgadmin \ # name to discover this in network
  dpage/pgadmin4

work on browser pdadmin to connect server to localhost docker of postgres
### Data ingestion
 ### Multiline cursor
ingest_data.py

URL="https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2021-01.parquet"
python ingest_data.py \
  --user=root \
  --password=root \
  --host=localhost \
  --port=5432 \
  --db=ny_taxi \
  --table_name=yellow_taxi \
  --url=${URL}

# Dockerize:
docker build -t taxi_ingest:v001 .

docker run -it \
  --network=pg_network \
  taxi_ingest:v001 \
    --user=root \
    --password=root \
    --host=pgdatabase \
    --port=5432 \
    --db=ny_taxi \
    --table_name=yellow_taxi \
    --url=${URL}

# Docker-compose:
#create: code docker-compose.yml
docker-compose up
// docker-compose up -d # get terminal back
docker-compose down

docker run -it \
  --network=docker_sql_0_default \
  taxi_ingest:v001 \
    --user=root \
    --password=root \
    --host=docker_sql_0_pgdatabase_1 \
    --port=5432 \
    --db=ny_taxi \
    --table_name=yellow_taxi \
    --url=${URL}

# Terrafomr & GCP

#Key on GCP in services permissions

export GOOGLE_APPLICATION_CREDENTIALS="<path/to/your/service-account-authkeys>.json"

gcloud auth application-default login

# add Permissions for service account
// Storage admin to create, update, delete.... buckets and files for terraform

Enable these APIs for your project:
   * https://console.cloud.google.com/apis/library/iam.googleapis.com
   * https://console.cloud.google.com/apis/library/iamcredentials.googleapis.com

advisable to destroy then apply terraform to save resources and money

# Set-up environment on GCP

SSH access & Cloud VM
ssh-keygen -t rsa -f gcp -C 7asanen999 -b 2048

// take .pub key and add it to VM
// create ur instance as desired then copy IP and
ssh -i ~/.ssh/gcp 7asanen999@***IP***

touch config // in .ssh:
Host # NAME
    HostName # IP of engine
    User # User in SSH
    IdentityFile #SSH
//can access env directly by: ssh #NAME 

// add docker
sudo apt-get install docker.io
//search run it w/o sudo then run commands
//wget right version docker compose search online run commands github
// add it to folder with -O docker-compose
run: chmod +x docker-compose
//ADD to PATH:
nano .bashrc:
export PATH="${HOME}/bin:${PATH}"
// forward ports on ur VS ssh run for ease access
// add terraform
//sftp for file transfer to ur VM
// in ur ssh apply in the terraform directory:
export GOOGLE_APPLICATION_CREDENTIALS=~/.gc/*FILE*.json
gcloud auth activate-service-account --key-file $GOOGLE_APPLICATION_CREDENTIALS
terraform init
terraform plan
terraform apply

sudo shutdown now


make exe executable to system
and add it to path in bashrc
///
chmod +x EXE
export PATH=....
///

sftp: file transfer protocol


SQL Homework: 

SELECT count(*)
FROM
	yellow_taxi t
WHERE CAST(t.tpep_pickup_datetime AS DATE) = '2021-01-15'

SELECT CAST(t.tpep_pickup_datetime AS DATE) pick_up_day,max(tip_amount)
FROM
	yellow_taxi t
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1

SELECT coalesce(zo."Zone", 'Unknown') zoning,count(*) counting
FROM
	yellow_taxi t INNER JOIN zones zpu
	ON CAST(t."PULocationID" as int) = zpu."LocationID"
	LEFT JOIN zones zo
	ON CAST(t."DOLocationID" as int) = zo."LocationID"
WHERE CAST(t.tpep_pickup_datetime AS DATE) = '2021-01-14'
AND zpu."Zone" ILIKE '%central park%'
GROUP BY zo."Zone"
ORDER BY counting DESC
lIMIT 1

SELECT CONCAT(coalesce(zpu."Zone", 'Unknown'),' / ',coalesce(zo."Zone", 'Unknown')) AS Pick_Drop,
	Avg(t.total_amount) most_popular_trip
FROM
	yellow_taxi t INNER JOIN zones zpu
	ON CAST(t."PULocationID" as int) = zpu."LocationID"
	LEFT JOIN zones zo
	ON CAST(t."DOLocationID" as int) = zo."LocationID"
GROUP BY Pick_Drop
ORDER BY most_popular_trip DESC
lIMIT 1

# Remove all stopped containers
docker rm $(docker ps --filter status=exited -q)
