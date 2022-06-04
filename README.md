# Magento 2.4.3 Docker Compose for ARM
###### Working on M1 Apple Silicon

## Containers
### Mysql
- Name: `magento-mysql`
- Image: `mysql/mysql-server:8.0.26`
- Port: `3306:3306`
- Volume: `magento-mysql-data`

#### Config
Replace next variables into `.env`: 

- `MYSQL_ROOT_PASSWORD`
- `MYSQL_DATABASE`
- `MYSQL_USER`
- `MYSQL_USER_PASSWORD`

### ElasticSearch 
- Name: `magento-elastic`
- Image: `docker.elastic.co/elasticsearch/elasticsearch:7.9.3-arm64`
- Ports:
  - `9200:9200`
  - `9300:9300`


### Web container
###### PHP 7.4 and modules, Composer, Xdebug preinstalled 
- Name: `magento-web`
- Image: Build from `ubuntu`
- Port: `5001:5001`
- Volume: `${MAGENTO_ABSOLUTE_PATH}:/workspaces/magento`

#### Config
Replace `MAGENTO_ABSOLUTE_PATH` into `.env`.

This absolute path is where Magento will be installed or already is installed.

## Containers Network
`magento-network` is created to communicate containers between with next network alias for containers:
- mysql
- elasticsearch
- web

## Usage
1. Copy `.env.template` to `.env` and replace default variables
2. Run `docker-compose up -d` to set up images and containers
3. Go inside web container bash running `docker-compose exec web bash` or `docker exec -it magento-web bash`

#### New Project
- Run `composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition ./install` to download Magento Community Edition
- Move generated files into root directory location with `(shopt -s dotglob; mv -v ./install/* .)`
- Install Magento via CLI (replace {{vars}} with the ones written on `.env`):
```
bin/magento setup:install \
  --base-url=http://magentodev.com:5001 \
  --db-host=mysql \
  --db-name={{dbName}} \
  --db-user={{dbUser}} \
  --db-password={{dbPassword}} \
  --admin-firstname=admin \
  --admin-lastname=admin \
  --admin-email=admin@admin.com \
  --admin-user=admin \
  --admin-password=admin123 \
  --language=en_US \
  --currency=USD \
  --timezone=America/New_York \
  --use-rewrites=1 \
  --elasticsearch-host=elasticsearch \
  --elasticsearch-port=9200
```
- Fire up via the PHP built in server: `php -S 0.0.0.0:5001 -t ./pub/ ./phpserver/router.php`
- Go to http://magentodev.com:5001 in browser

#### Existing Project - BYODB (Bring Your Own DB)

- Import your existing DB from out of containers with
`mysql -h localhost -P 3306 --protocol=tcp -u{{dbUser}} -p magento < /path/to/file.sql`
- Fire up via the PHP built in server: php -S 0.0.0.0:5001 -t ./pub/ ./phpserver/router.php
- Go to http://yourdomain.com:5001 in browser

##  Magento CLI

```
docker exec -it magento-web bin/magento command
```

