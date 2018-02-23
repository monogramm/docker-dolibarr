[![Build Status](https://travis-ci.org/madmath03/docker-dolibarr.svg)](https://travis-ci.org/madmath03/docker-dolibarr)

# Dolibarr on Docker

Docker image for Dolibarr.

Provides full database configuration, production mode, HTTPS enforcer (SSL must be provided by reverse proxy), handles upgrades, and so on...

## What is Dolibarr ?

Dolibarr ERP & CRM is a modern software package to manage your organization's activity (contacts, suppliers, invoices, orders, stocks, agenda, ...).

> [More informations](https://github.com/dolibarr/dolibarr)

## Supported tags

* `7.0.0-apache` `7.0.0` `7.0-apache` `7.0` `7-apache` `7` `apache` `latest`
* `7.0.0-fpm` `7.0-fpm` `7-fpm` `fpm`
* `6.0.5-apache` `6.0.5` `6.0-apache` `6.0` `6-apache` `6`
* `6.0.5-fpm` `6.0-fpm` `6-fpm`
* `5.0.7-apache` `5.0.7` `5.0-apache` `5.0` `5-apache` `5`
* `5.0.7-fpm` `5.0-fpm` `5-fpm`

## How to run this image ?

This image is based on the [officiel PHP repository](https://registry.hub.docker.com/_/php/) and inspired from [tuxgasy/docker-dolibarr](https://github.com/tuxgasy/docker-dolibarr).

This image does not contain the database for Dolibarr. You need to use either an existing database or a database container.

This image is designed to be used in a micro-service environment. There are two versions of the image you can choose from.

The `apache` tag contains a full Dolibarr installation including an apache web server. It is designed to be easy to use and gets you running pretty fast. This is also the default for the `latest` tag and version tags that are not further specified.

The second option is a `fpm` container. It is based on the [php-fpm](https://hub.docker.com/_/php/) image and runs a fastCGI-Process that serves your Dolibarr page. To use this image it must be combined with any webserver that can proxy the http requests to the FastCGI-port of the container.

## Using the apache image
The apache image contains a webserver and exposes port 80. To start the container type:

```console
$ docker run -d -p 8080:80 madmath03/docker-dolibarr
```

Now you can access Dolibarr at http://localhost:8080/ from your host system.


## Using the fpm image
To use the fpm image you need an additional web server that can proxy http-request to the fpm-port of the container. For fpm connection this container exposes port 9000. In most cases you might want use another container or your host as proxy.
If you use your host you can address your Dolibarr container directly on port 9000. If you use another container, make sure that you add them to the same docker network (via `docker run --network <NAME> ...` or a `docker-compose` file).
In both cases you don't want to map the fpm port to you host. 

```console
$ docker run -d madmath03/docker-dolibarr:fpm
```

As the fastCGI-Process is not capable of serving static files (style sheets, images, ...) the webserver needs access to these files. This can be achieved with the `volumes-from` option. You can find more information in the docker-compose section.

## Using an external database
By default this container does not contain the database for Dolibarr. You need to use either an existing database or a database container.

The Dolibarr setup wizard (should appear on first run) allows connecting to an existing MySQL/MariaDB or PostgreSQL database. You can also link a database container, e. g. `--link my-mysql:mysql`, and then use `mysql` as the database host on setup. More info is in the docker-compose section.

## Persistent data
The Dolibarr installation and all data beyond what lives in the database (file uploads, etc) are stored in the [unnamed docker volume](https://docs.docker.com/engine/tutorials/dockervolumes/#adding-a-data-volume) volume `/var/www/html` and  `/var/www/documents`. The docker daemon will store that data within the docker directory `/var/lib/docker/volumes/...`. That means your data is saved even if the container crashes, is stopped or deleted.

To make your data persistent to upgrading and get access for backups is using named docker volume or mount a host folder. To achieve this you need one volume for your database container and Dolibarr.

Dolibarr:
- `/var/www/html/` folder where all Dolibarr data lives
- `/var/www/documents/` folder where all Dolibarr documents lives
```console
$ docker run -d \
    -v dolibarr_html:/var/www/html \
    -v dolibarr_docs:/var/www/documents \
    madmath03/docker-dolibarr
```

Database:
- `/var/lib/mysql` MySQL / MariaDB Data
- `/var/lib/postgresql/data` PostgreSQL Data
```console
$ docker run -d \
    -v db:/var/lib/mysql \
    mariadb
```

If you want to get fine grained access to your individual files, you can mount additional volumes for config, your theme and custom modules. 
The `conf` is stored in subfolder inside `/var/www/html/`. The modules are split into core `apps` (which are shipped with Dolibarr and you don't need to take care of) and a `custom` folder. If you use a custom theme it would go into the `theme` subfolder.

Overview of the folders that can be mounted as volumes:

- `/var/www/html` Main folder, needed for updating
- `/var/www/html/custom` installed / modified modules
- `/var/www/html/conf` local configuration
- `/var/www/html/theme/<YOUR_CUSTOM_THEME>` theming/branding

If you want to use named volumes for all of these it would look like this
```console
$ docker run -d \
    -v dolibarr:/var/www/html \
    -v apps:/var/www/html/custom \
    -v config:/var/www/html/conf \
    -v theme:/var/www/html/theme/<YOUR_CUSTOM_THEME> \
    madmath03/docker-dolibarr
```

## Auto configuration via environment variables

The Dolibarr image supports auto configuration via environment variables. You can preconfigure nearly everything that is asked on the install page on first run. To enable auto configuration, set your database connection via the following environment variables. ONLY use one database type!

See [conf.php.example](https://github.com/Dolibarr/dolibarr/blob/develop/htdocs/conf/conf.php.example) and [install.forced.sample.php](https://github.com/Dolibarr/dolibarr/blob/develop/htdocs/install/install.forced.sample.php) for more details on install configuration.


### DOLI_DB_TYPE

*Default value*: 

*Possible values*: `mysqli`, `pgsql`

This parameter contains the name of the driver used to access your Dolibarr database.

Examples:
```
DOLI_DB_TYPE=mysqli
DOLI_DB_TYPE=pgsql
```

### DOLI_DB_HOST

*Default value*: 

This parameter contains host name or ip address of Dolibarr database server.

Examples:
```
DOLI_DB_HOST=localhost
DOLI_DB_HOST=127.0.0.1
DOLI_DB_HOST=192.168.0.10
DOLI_DB_HOST=mysql.myserver.com
```

### DOLI_DB_PORT

*Default value*: 

This parameter contains the port of the Dolibarr database.

Examples:
```
DOLI_DB_PORT=3306
DOLI_DB_PORT=5432
```

### DOLI_DB_NAME

*Default value*: 

This parameter contains name of Dolibarr database.

Examples:
```
DOLI_DB_NAME=dolibarr
DOLI_DB_NAME=mydatabase
```

### DOLI_DB_USER

*Default value*: 

This parameter contains user name used to read and write into Dolibarr database.

Examples:
```
DOLI_DB_USER=admin
DOLI_DB_USER=dolibarruser
```

### DOLI_DB_PASSWORD

*Default value*: 

This parameter contains password used to read and write into Dolibarr database.

Examples:
```
DOLI_DB_PASSWORD=myadminpass
DOLI_DB_PASSWORD=myuserpassword
```

### DOLI_DB_PREFIX

*Default value*: `llx_`

This parameter contains prefix of Dolibarr database.

Examples:
```
DOLI_DB_PREFIX=llx_
```

### DOLI_ADMIN_LOGIN

*Default value*: `admin`

This parameter contains the admin's login used in the first install.

Examples:
```
DOLI_ADMIN_LOGIN=admin
```

### DOLI_ADMIN_PASSWORD

**NOT YET WORKING**

*Default value*: 

This parameter contains the admin's password used in the first install.
It requires `DOLI_AUTO_INSTALL` to be enabled in order to work.

Examples:
```
DOLI_ADMIN_PASSWORD=myadminpass
```

### DOLI_AUTO_INSTALL

**NOT YET WORKING**

*Default value*: `0`

*Possible values*: `0` or `1`

The installation will be automatically executed on first boot. If set to `0`, first install wizard will be initialized based on environment variables.

Examples:
```
DOLI_AUTO_INSTALL=0
DOLI_AUTO_INSTALL=1
```

### DOLI_URL_ROOT

*Default value*: `http://localhost`

This parameter defines the root URL of your Dolibarr index.php page without ending "/".
It must link to the directory htdocs.
In most cases, this is autodetected but it's still required 
* to show full url bookmarks for some services (ie: agenda rss export url, ...)
* or when using Apache dir aliases (autodetect fails)
* or when using nginx (autodetect fails)

Examples:
```
DOLI_URL_ROOT=http://localhost
DOLI_URL_ROOT=http://mydolibarrvirtualhost
DOLI_URL_ROOT=http://myserver/dolibarr/htdocs
DOLI_URL_ROOT=http://myserver/dolibarralias
```


### DOLI_AUTH

**NOT YET WORKING**

*Default value*: `dolibarr`

*Possible values*: Any values found in files in htdocs/core/login directory after the `function_` string and before the `.php` string, **except forceuser**. You can also separate several values using a `,`. In this case, Dolibarr will check login/pass for each value in order defined into value. However, note that this can't work with all values.

This parameter contains the way authentication is done.
It requires `DOLI_AUTO_INSTALL` to be enabled to be used. Otherwise, install wizard will use ignore 

If value `ldap` is used, you must also set parameters `DOLI_LDAP_*` and `DOLI_MODULES` must contain `modLdap`.

Examples:
```
DOLI_AUTH=http
DOLI_AUTH=dolibarr
DOLI_AUTH=ldap
DOLI_AUTH=openid,dolibarr
```

### DOLI_LDAP_HOST

**NOT YET WORKING**

*Default value*: `127.0.0.1`

You can define several servers here separated with a comma.

Examples:
```
DOLI_LDAP_HOST=localhost
DOLI_LDAP_HOST=ldap.company.com
DOLI_LDAP_HOST=ldaps://ldap.company.com:636,ldap://ldap.company.com:389
```

### DOLI_LDAP_PORT

**NOT YET WORKING**

*Default value*: `389`

### DOLI_LDAP_VERSION

**NOT YET WORKING**

*Default value*: `3`

### DOLI_LDAP_SERVERTYPE

**NOT YET WORKING**

*Default value*: `openldap`
*Possible values*: `openldap`, `activedirectory` or `egroupware`

### DOLI_LDAP_DN

**NOT YET WORKING**

*Default value*: 

Examples:
```
DOLI_LDAP_DN=ou=People,dc=company,dc=com
```

### DOLI_LDAP_LOGIN_ATTRIBUTE

**NOT YET WORKING**

*Default value*: `uid`

Ex: uid or samaccountname for active directory

### DOLI_LDAP_FILTER

**NOT YET WORKING**

*Default value*: 

If defined, the two previous parameters are not used to find a user into LDAP.

Examples:
```
DOLI_LDAP_FILTER=(uid=%1%)
DOLI_LDAP_FILTER=(&(uid=%1%)(isMemberOf=cn=Sales,ou=Groups,dc=company,dc=com))
```

### DOLI_LDAP_ADMIN_LOGIN

**NOT YET WORKING**

*Default value*: 

Required only if anonymous bind disabled.

Examples:
```
DOLI_LDAP_ADMIN_LOGIN=cn=admin,dc=company,dc=com
```

### DOLI_LDAP_ADMIN_PASS

**NOT YET WORKING**

*Default value*: 

Required only if anonymous bind disabled. Ex: 

Examples:
```
DOLI_LDAP_ADMIN_PASS=secret
```

### DOLI_LDAP_DEBUG

**NOT YET WORKING**

*Default value*: `false`


### DOLI_PROD

*Default value*: `0`

*Possible values*: `0` or `1`

When this parameter is defined, all errors messages are not reported.
This feature exists for production usage to avoid to give any information to hackers.

Examples:
```
DOLI_PROD=0
DOLI_PROD=1
```

### DOLI_HTTPS

*Default value*: `0`

*Possible values*: `0`, `1`, `2` or `'https://my.domain.com'`

This parameter allows to force the HTTPS mode.
0 = No forced redirect
1 = Force redirect to https, until SCRIPT_URI start with https into response
2 = Force redirect to https, until SERVER["HTTPS"] is 'on' into response
'https://my.domain.com' = Force redirect to https using this domain name.

*Warning*: If you enable this parameter, your web server must be configured to
respond URL with https protocol. 
According to your web server setup, some values may work and other not. Try 
different values (1,2 or 'https://my.domain.com') if you experience problems.

Examples:
```
DOLI_HTTPS=0
DOLI_HTTPS=1
DOLI_HTTPS=2
DOLI_HTTPS=https://my.domain.com
```


### PHP_INI_DATE_TIMEZONE

*Default value*: `UTC`

Default timezone on PHP.

### WWW_USER_ID

*Default value*: `33`

ID of user www-data. ID will not change if left empty. During development, it is very practical to put the same ID as the host user.

### WWW_GROUP_ID

*Default value*: `33`

ID of group www-data. ID will not change if left empty.


# Running this image with docker-compose

## Base version - apache with MariaDB/MySQL

This version will use the apache image and add a [MariaDB](https://hub.docker.com/_/mariadb/) container (you can also use [MySQL](https://hub.docker.com/_/mysql/) if you prefer). The volumes are set to keep your data persistent. This setup provides **no ssl encryption** and is intended to run behind a proxy. 

Make sure to set the variables `MYSQL_ROOT_PASSWORD`, `MYSQL_PASSWORD` and `DOLI_DB_PASSWORD` before you run this setup.

Create `docker-compose.yml` file as following:

```yml
version: '2'

volumes:
  dolibarr_html:
  dolibarr_docs:
  dolibarr_db:

mariadb:
    image: mariadb:latest
    restart: always
    volumes:
      - dolibarr_db:/var/lib/mysql
    environment:
        - "MYSQL_ROOT_PASSWORD="
        - "MYSQL_PASSWORD="
        - "MYSQL_DATABASE=dolibarr"
        - "MYSQL_USER=dolibarr"

dolibarr:
    image: madmath03/docker-dolibarr
    restart: always
    depends_on:
        - mariadb
    ports:
        - "8080:80"
    environment:
        - "DOLI_DB_HOST=mariadb"
        - "DOLI_DB_NAME=dolibarr"
        - "DOLI_DB_USER=dolibarr"
        - "DOLI_DB_PASSWORD="
    volumes:
        - dolibarr_html:/var/www/html
        - dolibarr_docs:/var/www/documents
```

Then run all services `docker-compose up -d`. Now, go to http://localhost:8080/install to access the new Dolibarr installation wizard.

## Base version - FPM with PostgreSQL
When using the FPM image you need another container that acts as web server on port 80 and proxies the requests to the Dolibarr container. In this example a simple nginx container is combined with the madmath03/docker-dolibarr-fpm image and a [PostgreSQL](https://hub.docker.com/_/postgres/) database container. The data is stored in docker volumes. The nginx container also need access to static files from your Dolibarr installation. It gets access to all the volumes mounted to Dolibarr via the `volumes_from` option. The configuration for nginx is stored in the configuration file `nginx.conf`, that is mounted into the container.

As this setup does **not include encryption** it should to be run behind a proxy. 

Make sure to set the variables `POSTGRES_PASSWORD` and `DOLI_DB_PASSWORD` before you run this setup.

Create `docker-compose.yml` file as following:

```yml
version: '2'

volumes:
  dolibarr_html:
  dolibarr_docs:
  dolibarr_db:

postgres:
    image: postgres:latest
    restart: always
    environment:
        - "POSTGRES_DB=dolibarr"
        - "POSTGRES_USER=dolibarr"
        - "POSTGRES_PASSWORD="
    volumes:
        - dolibarr_db:/var/lib/postgresql/data

dolibarr:
    image: madmath03/docker-dolibarr
    depends_on:
        - postgres
    ports:
        - "80:80"
    environment:
        - "DOLI_DB_TYPE=pgsql"
        - "DOLI_DB_HOST=postgres"
        - "DOLI_DB_PORT=5432"
        - "DOLI_DB_NAME=dolibarr"
        - "DOLI_DB_USER=dolibarr"
        - "DOLI_DB_PASSWORD="
    volumes:
        - dolibarr_html:/var/www/html
        - dolibarr_docs:/var/www/documents

web:
    image: nginx
    ports:
        - 8080:80
    links:
        - dolibarr
    volumes:
        - ./nginx.conf:/etc/nginx/nginx.conf:ro
    volumes_from:
        - dolibarr
    restart: always
```

Then run all services `docker-compose up -d`. Now, go to http://localhost:8080/install to access the new Dolibarr installation wizard.


# Make your Dolibarr available from the internet
Until here your Dolibarr is just available from you docker host. If you want you Dolibarr available from the internet adding SSL encryption is mandatory.

## HTTPS - SSL encryption
There are many different possibilities to introduce encryption depending on your setup. 

We recommend using a reverse proxy in front of our Dolibarr installation. Your Dolibarr will only be reachable through the proxy, which encrypts all traffic to the clients. You can mount your manually generated certificates to the proxy or use a fully automated solution, which generates and renews the certificates for you.


# First use
When you first access your Dolibarr, you need to access the install wizard at `http://localhost/install/`.
The setup wizard will appear and ask you to choose an administrator account, password and the database connection. For the database use the name of your database container as host and `dolibarr` as table and user name. Also enter the password you chose in your `docker-compose.yml` file.


# Update to a newer version
Updating the Dolibarr container is done by pulling the new image, throwing away the old container and starting the new one. Since all data is stored in volumes, nothing gets lost. The startup script will check for the version in your volume and the installed docker version. If it finds a mismatch, it automatically starts the upgrade process. Don't forget to add all the volumes to your new container, so it works as expected. 

```console
$ docker pull madmath03/docker-dolibarr
$ docker stop <your_dolibarr_container>
$ docker rm <your_dolibarr_container>
$ docker run <OPTIONS> -d madmath03/docker-dolibarr
```
Beware that you have to run the same command with the options that you used to initially start your Dolibarr. That includes volumes, port mapping.

When using docker-compose your compose file takes care of your configuration, so you just have to run:

```console
$ docker-compose pull
$ docker-compose up -d
```


# Adding Features
If the image does not include the packages you need, you can easily build your own image on top of it.
Start your derived image with the `FROM` statement and add whatever you like.

```yaml
FROM madmath03/docker-dolibarr:apache

RUN ...

```

You can also clone this repository and use the [update.sh](update.sh) shell script to generate a new Dockerfile based on your own needs.

For instance, you could build a container based on Dolibarr develop branch and  by setting the `update.sh` versions like this:
```bash
versions=( "latest" )
```
Then simply call [update.sh](update.sh) script.

```console
bash update.sh
```
Your Dockerfile(s) will be generated in the `images` folder.

If you use your own Dockerfile you need to configure your docker-compose file accordingly. Switch out the `image` option with `build`. You have to specify the path to your Dockerfile. (in the example it's in the same directory next to the docker-compose file)

```yaml
  app:
    build: .
    links:
      - db
    volumes:
      - data:/var/www/html/data
      - config:/var/www/html/config
      - apps:/var/www/html/apps
    restart: always
```

**Updating** your own derived image is also very simple. When a new version of the Dolibarr image is available run:

```console
docker build -t your-name --pull . 
docker run -d your-name
```

or for docker-compose:
```console
docker-compose build --pull
docker-compose up -d
```

The `--pull` option tells docker to look for new versions of the base image. Then the build instructions inside your `Dockerfile` are run on top of the new image.

# Questions / Issues
If you got any questions or problems using the image, please visit our [Github Repository](https://github.com/madmath03/docker-dolibarr) and write an issue.  
