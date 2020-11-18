# How to set up Redash on a Mac 

## Preliminaries

If you don't already have these, 
```
brew install wget
brew install pwgen
```
## Create directories

I've selected these because my laptop is locked down, so I can't use `/opt/redash`, which is the default:
```
mkdir -p ~/opt/redash
touch ~/opt/redash/env
mkdir ~/opt/redash/postgres-data
```
## Set up your environment
```
export REDASH_BASE_PATH=~/opt/redash
COOKIE_SECRET=$(pwgen -1s 32)
SECRET_KEY=$(pwgen -1s 32)
POSTGRES_PASSWORD=$(pwgen -1s 32)
REDASH_DATABASE_URL="postgresql://postgres:${POSTGRES_PASSWORD}@postgres/postgres"
```
## Create the environment file
```
echo "PYTHONUNBUFFERED=0" >> $REDASH_BASE_PATH/env
echo "REDASH_LOG_LEVEL=INFO" >> $REDASH_BASE_PATH/env
echo "REDASH_REDIS_URL=redis://redis:6379/0" >> $REDASH_BASE_PATH/env
echo "POSTGRES_PASSWORD=$POSTGRES_PASSWORD" >> $REDASH_BASE_PATH/env
echo "REDASH_COOKIE_SECRET=$COOKIE_SECRET" >> $REDASH_BASE_PATH/env
echo "REDASH_SECRET_KEY=$SECRET_KEY" >> $REDASH_BASE_PATH/env
echo "REDASH_DATABASE_URL=$REDASH_DATABASE_URL" >> $REDASH_BASE_PATH/env
```
## Start getting files
```
REQUESTED_CHANNEL=stable
LATEST_VERSION=`curl -s "https://version.redash.io/api/releases?channel=$REQUESTED_CHANNEL"  | json_pp  | grep "docker_image" | head -n 1 | awk 'BEGIN{FS=":" {print $3}' | awk 'BEGIN{FS="\""}{print $1}'
cd $REDASH_BASE_PATH
GIT_BRANCH="${REDASH_BRANCH:-master}"
wget --no-check-certificate http://raw.githubusercontent.com/getredash/setup/${GIT_BRANCH}/data/docker-compose.yml
sed -iE "s/image: redash\/redash:([A-Za-z0-9.-]*)/image: redash\/redash:$LATEST_VERSION/" docker-compose.yml
sed -iE "s/\/opt\/redash/~\/opt\/redash/" docker-compose.yml
echo "export COMPOSE_PROJECT_NAME=redash" >> ~/.zshrc                   # or .bashrc
echo "export COMPOSE_FILE=${REDASH_BASE_PATH}/docker-compose.yml" >> ~/.zshrc  # or .bashrc
export COMPOSE_PROJECT_NAME=redash
export COMPOSE_FILE=$(REDASH_BASE_PATH}/docker-compose.yml
docker-compose run --rm server create_db
```
Use `docker logs redash_postgres_1` to ensure that Postgres initialised completed before continuing. You should see something like this at the end of the log:
```
LOG:  database system was shut down at 2020-11-18 11:16:02 UTC
LOG:  MultiXact member wraparound protections are now enabled
LOG:  database system is ready to accept connections
LOG:  autovacuum launcher started
```
Now run `docker-compose up -d`
