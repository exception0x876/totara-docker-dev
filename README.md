## A Docker setup for local Totara LMS development

This project aims to provide an easy way to start developing for Totara by providing a Docker setup.

This setup was created and tested on a MAC. It should work on Windows and Linux as well but it still needs to be tested.

### What you get:
 * NGINX as a webserver
 * PHP 5.6, 7.0, 7.1, 7.2RC to test for different versions
 * PostgreSQL, MariaDB (MySQL), Microsoft SQL Server support
 * A PHPUnit and Behat setup to run tests (including Selenium)
 * A [mailcatcher](https://mailcatcher.me/) instance to inspect mails
 * [XDebug](https://xdebug.org/) installed, ready for debugging with your favorite IDE

### Requirements:
 * Totara source code: https://help.totaralearning.com/display/DEV/Getting+the+code
 * Docker: https://www.docker.com
 * Docker-compose: https://docs.docker.com/compose/install (included in Docker for Mac/Windows)
 * Docker-sync: http://docker-sync.io/ (optional, for more speed on Mac and Windows)
 * At least 3.25GB of RAM for MSSQL

## Warning
Please note that there's a current [issue with docker-sync](https://github.com/EugenMayer/docker-sync/issues/517) opn Mac and Docker versions newer than [17.09.1-ce-mac42](https://docs.docker.com/docker-for-mac/release-notes/#docker-community-edition-17091-ce-mac42-2017-12-11-stable). To be on the safe side I recommend download and installing this version.

### Todo

 * Get mssql working with PHP 5.6 (driver is installed but throws error on connect)
 * Get mssql working with PHP 7.2RC (dependency problem in mssql extension)

### Installation:
 1. Clone the Totara source code (see requirements) 
 1. Clone this project
 1. Install docker-sync
 1. Copy the file __.env.dist__ to __.env__ and change at least the path to your local Totara source folder (LOCAL_SRC)
 1. In your totara-docker-env folder run:

__with docker-sync__
```bash
# use helper file provided
./totara-docker.sh build

# or call it directly
docker-compose -f docker-compose.yml -f docker-compose-dev.yml build
```

__without docker-sync__
```bash
docker-compose build
```

#### /etc/hosts
Make sure you have all the hosts in your /etc/hosts file to be able to access them via the browser.

__Example:__
```bash
127.0.0.1   localhost totara71 totara56 totara70 totara72 totara71.behat totara56.behat
```

You can change the hostnames in the nginx configuration file (/nginx/config/totara.conf) to your needs.

### Run

__with docker-sync__

```bash
# 1. fire up docker-sync as a daemon in the background
docker-sync start
# or alternatively once in the foreground
docker-sync-stack start
```

```bash
# use helper file provided
./totara-docker.sh up
# run in background
./totara-docker.sh up -d 

# or call it directly
docker-compose -f docker-compose.yml -f docker-compose-dev.yml up
# run in background
docker-compose -f docker-compose.yml -f docker-compose-dev.yml up -d 
```

__without docker-sync__
```bash
docker-compose up
```

Now make sure you have configured Totara and created the databases you need.

### Config & Database

Modify your Totara __config.php__ and create the databases. You can connect to the databases from your host using any tools you prefer.

The host will be always __localhost__ and the ports are the default ports of the database systems.

To use the command line clients provided by the containers you can use the following commands:

```bash
# PostgreSQL
docker exec -ti docker_pgsql_1 psql -U postgres
# MySQL / MariaDB
docker exec -ti docker_mariadb_1 mysql -u root -p"root"
# Microsoft SQL Server
docker exec -ti docker_php-7.1_1 /opt/mssql-tools/bin/sqlcmd -S mssql -U SA -P "Totara.Mssql1"
```

Create a database schema for each Totara version you would like to develop on.

### Run unit tests

Make sure your config file contains the PHPUnit configuration needed and the database is ready.

Log into one of the test containers
```bash
./totara-docker.sh exec php-5.6 bash
./totara-docker.sh exec php-7.1 bash
# or if you need xdebug support
./totara-docker.sh exec php-5.6-debug bash
./totara-docker.sh exec php-7.1-debug bash
```

Go to the project folder
```bash
# replace version
cd /var/www/totara/src/[version]
```

First time run the init script to initiate the unit tests
```bash
# in the project folder
php composer.phar install
# initiate the test environment
php admin/tool/phpunit/cli/init.php
```

To start running
```bash
vendor/bin/phpunit
```

### Run behat tests

wip

### Switch between different versions

wip

# Mailcatcher

The setup comes with mailcatcher support. Just add the following to your config and all mails will be sent to it:

```php
$CFG->smtphosts = 'mailcatcher:25';
```

Open __http://localhost:8080__ to open the mailcatcher GUI.

If needed modify the local port in the docker-compose.yml file.

# NodeJS, NPM and grunt 

If you want to use grunt or npm you can log into the nodejs container and issue the commands there:

```bash
./totara-docker.sh run nodejs bash
# go to your source directory and
npm install
npm install grunt-cli
./node_modules/.bin/grunt
```

Or you use the shortcut bash script:

```bash
./totara-grunt.sh
# if you have your version in REMOTE_SRC/subfolder then add the subfolder as parameter
./totara-grunt.sh subfolder
``` 