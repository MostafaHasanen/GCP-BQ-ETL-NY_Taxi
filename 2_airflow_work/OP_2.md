from tutorials setup_official
Work on Dockerfile, get docker-compose from airflow initial setup

build, run docker-compose up airflow-init
#============================================
echo -e "AIRFLOW_UID=$(id -u)" > .env
set AIRFLOWUID: ex: AIRFLOW_UID=50000
run docker-compose up airflow-init
run: docker-compose up


# DATA INGESTING


# Local lightweight setup airflow

# changes in docker compose:
# after postgres in services remove:
#   redis:
#     image: redis:latest
#     expose:
#       - 6379
#     healthcheck:
#       test: ["CMD", "redis-cli", "ping"]
#       interval: 5s
#       timeout: 30s
#       retries: 50
#     restart: always
# #
#   airflow-worker:
#     <<: *airflow-common
#     command: celery worker
#     healthcheck:
#       test:
#         - "CMD-SHELL"
#         - 'celery --app airflow.executors.celery_executor.app inspect ping -d "celery@$${HOSTNAME}"'
#       interval: 10s
#       timeout: 10s
#       retries: 5
#     environment:
#       <<: *airflow-common-env
#       # Required to handle warm shutdown of the celery workers properly
#       # See https://airflow.apache.org/docs/docker-stack/entrypoint.html#signal-propagation
#       DUMB_INIT_SETSID: "0"
#     restart: always
#     depends_on:
#       <<: *airflow-common-depends-on
#       airflow-init:
#         condition: service_completed_successfully
# #
#   airflow-triggerer:
#     <<: *airflow-common
#     command: triggerer
#     healthcheck:
#       test: ["CMD-SHELL", 'airflow jobs check --job-type TriggererJob --hostname "$${HOSTNAME}"']
#       interval: 10s
#       timeout: 10s
#       retries: 5
#     restart: always
#     depends_on:
#       <<: *airflow-common-depends-on
#       airflow-init:
#         condition: service_completed_successfully
# #
#   flower:
#     <<: *airflow-common
#     command: celery flower
#     ports:
#       - 5555:5555
#     healthcheck:
#       test: ["CMD", "curl", "--fail", "http://localhost:5555/"]
#       interval: 10s
#       timeout: 10s
#       retries: 5
#     restart: always
#     depends_on:
#       <<: *airflow-common-depends-on
#       airflow-init:
#         condition: service_completed_successfully

# # From depend on remove:
#     redis:
#       condition: service_healthy

# change Core Executor mode from CeleryExecutor to LocalExecutor &
# remove/comment Celery parts in code after (next few lines)
# Option stop and remove all containers:
# docker-compose down --volumes --rmi all
# run as usual :)

#docker rm $(docker ps --filter status=exited -q)
