Docker Compose Drupal 9 advanced - php7.4, nginx, mariadb, solr, redis
======================================================================

This is a docker-compose version of the Lando example tests:

Start up tests
--------------

Run the following commands to get up and running with this example.

```bash
# Should remove any previous runs and poweroff
sed -i -e "/###/d" docker-compose.yml
docker network inspect amazeeio-network >/dev/null || docker network create amazeeio-network
docker-compose down

# Should start up our Lagoon Drupal 9 site successfully
docker-compose build && docker-compose up -d

# Ensure mariadb pod is ready to connect
docker run --rm --net amazeeio-ddev_default jwilder/dockerize dockerize -wait tcp://mariadb:3306 -timeout 1m
```

Verification commands
---------------------

Run the following commands to validate things are rolling as they should.

```bash
# Should be able to site install via Drush
docker-compose exec -T cli bash -c "drush si -y"
docker-compose exec -T cli bash -c "drush cr -y"
docker-compose exec -T cli bash -c "drush status" | grep "Drupal bootstrap" | grep "Successful"

# Should have all the services we expect
docker ps --filter label=com.docker.compose.project=amazeeio-ddev | grep Up | grep amazeeio-ddev_nginx_1
docker ps --filter label=com.docker.compose.project=amazeeio-ddev | grep Up | grep amazeeio-ddev_mariadb_1
docker ps --filter label=com.docker.compose.project=amazeeio-ddev | grep Up | grep amazeeio-ddev_php_1
docker ps --filter label=com.docker.compose.project=amazeeio-ddev | grep Up | grep amazeeio-ddev_cli_1
docker ps --filter label=com.docker.compose.project=amazeeio-ddev | grep Up | grep amazeeio-ddev_solr_1
docker ps --filter label=com.docker.compose.project=amazeeio-ddev | grep Up | grep amazeeio-ddev_redis_1

# Should ssh against the cli container by default
docker-compose exec -T cli bash -c "env | grep LAGOON=" | grep cli-drupal

# Should have the correct environment set
docker-compose exec -T cli bash -c "env" | grep LAGOON_ROUTE | grep amazeeio-ddev.docker.amazee.io
docker-compose exec -T cli bash -c "env" | grep LAGOON_ENVIRONMENT_TYPE | grep development

# Should be running PHP 7.4
docker-compose exec -T cli bash -c "php -v" | grep "PHP 7.4"

# Should have composer
docker-compose exec -T cli bash -c "composer --version"

# Should have php cli
docker-compose exec -T cli bash -c "php --version"

# Should have drush
docker-compose exec -T cli bash -c "drush --version"

# Should have npm
docker-compose exec -T cli bash -c "npm --version"

# Should have node
docker-compose exec -T cli bash -c "node --version"

# Should have yarn
docker-compose exec -T cli bash -c "yarn --version"

# Should have a running Drupal 9 site served by nginx on port 8080
docker-compose exec -T cli bash -c "curl -kL http://nginx:8080" | grep "Welcome to Drush Site-Install"

# Should be running Redis v5.0
docker-compose exec -T redis sh -c "redis-server --version" | grep v=5.

# Should be able to see Redis databases
docker-compose exec -T redis sh -c "redis-cli CONFIG GET databases"

# Redis databases should be initialized
docker-compose exec -T redis sh -c "redis-cli dbsize"

# Should have a "drupal" Solr core
docker-compose exec -T cli bash -c "curl solr:8983/solr/admin/cores?action=STATUS&core=drupal"

# Should be able to reload "drupal" Solr core
docker-compose exec -T cli bash -c "curl solr:8983/solr/admin/cores?action=RELOAD&core=drupal"

# Check Solr has 7.x config in "drupal" core
docker-compose exec -T solr bash -c "cat /opt/solr/server/solr/mycores/drupal/conf/schema.xml | grep solr-7.x"

# Should be able to db-export and db-import the database
docker-compose exec -T cli bash -c "drush sql-dump --result-file /app/test.sql"
docker-compose exec -T cli bash -c "drush sql-drop -y"
docker-compose exec -T cli bash -c "drush sql-cli < /app/test.sql"
docker-compose exec -T cli bash -c "rm test.sql*"

# Should be able to show the drupal tables
docker-compose exec -T cli bash -c "drush sqlq \'show tables;\'" | grep users

# Should be able to rebuild and persist the database
docker-compose build && docker-compose up -d
docker-compose exec -T cli bash -c "drush sqlq \'show tables;\'" | grep users
```

Destroy tests
-------------

Run the following commands to trash this app like nothing ever happened.

```bash
# Should be able to destroy our Drupal 9 site with success
docker-compose down --volumes --remove-orphans
```
