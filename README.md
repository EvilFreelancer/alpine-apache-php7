# Alpine-based LAP Server with PHP extensions

Provides a basic LAP stack using stable Alpine, Apache2 and PHP7 (version from stable repository of Alpine),
loading in the various extensions along the way (see Dockerfile for full list).

Should allow you to get going with a full LAP stack and support for DB via linked container (such as mysql)
with ease, allowing you to fine tune various aspects of the server and php via environment variables.

This project is partially taken from [here](https://github.com/ulsmith/alpine-apache-php7), but with
some stabilization patches (all packages installed only from stable branch of Alpine repository).

<details>
<summary>
  <i>List of packages included in this image</i>
</summary>

bash, apache2, php7, php7-apache2, git, tzdata, openntpd, nano,
curl, ca-certificates, openssl, openssh-keygen, openssh-client

php7-phar, php7-mcrypt, php7-soap, php7-openssl, php7-gmp, php7-pdo_odbc, php7-json, php7-dom, php7-pdo,
php7-zip, php7-mysqli, php7-sqlite3, php7-pdo_pgsql, php7-bcmath, php7-gd, php7-odbc, php7-pdo_mysql,
php7-pdo_sqlite, php7-gettext, php7-xmlreader, php7-xmlrpc, php7-bz2, php7-iconv, php7-pdo_dblib,
php7-curl, php7-ctype, php7-session, php7-redis, php7-fileinfo

</details>

<details>
<summary>
  <i>List of all available environment variables</i>
</summary>

Various env vars can be set at runtime via your docker command or docker-compose environment section:

__APACHE_SERVER_NAME:__ Change server name to match your domain name in httpd.conf

__PHP_SHORT_OPEN_TAG:__ Maps to php.ini 'short_open_tag'

__PHP_OUTPUT_BUFFERING:__ Maps to php.ini 'output_buffering'

__PHP_OPEN_BASEDIR:__ Maps to php.ini 'open_basedir'

__PHP_MAX_EXECUTION_TIME:__ Maps to php.ini 'max_execution_time'

__PHP_MAX_INPUT_TIME:__ Maps to php.ini 'max_input_time'

__PHP_MAX_INPUT_VARS:__ Maps to php.ini 'max_input_vars'

__PHP_MEMORY_LIMIT:__ Maps to php.ini 'memory_limit'

__PHP_ERROR_REPORTING:__ Maps to php.ini 'error_reporting'

__PHP_DISPLAY_ERRORS:__ Maps to php.ini 'display_errors'

__PHP_DISPLAY_STARTUP_ERRORS:__ Maps to php.ini 'display_startup_errors'

__PHP_LOG_ERRORS:__ Maps to php.ini 'log_errors'

__PHP_LOG_ERRORS_MAX_LEN:__ Maps to php.ini 'log_errors_max_len'

__PHP_IGNORE_REPEATED_ERRORS:__ Maps to php.ini 'ignore_repeated_errors'

__PHP_REPORT_MEMLEAKS:__ Maps to php.ini 'report_memleaks'

__PHP_HTML_ERRORS:__ Maps to php.ini 'html_errors'

__PHP_ERROR_LOG:__ Maps to php.ini 'error_log'

__PHP_POST_MAX_SIZE:__ Maps to php.ini 'post_max_size'

__PHP_DEFAULT_MIMETYPE:__ Maps to php.ini 'default_mimetype'

__PHP_DEFAULT_CHARSET:__ Maps to php.ini 'default_charset'

__PHP_FILE_UPLOADS:__ Maps to php.ini 'file_uploads'

__PHP_UPLOAD_TMP_DIR:__ Maps to php.ini 'upload_tmp_dir'

__PHP_UPLOAD_MAX_FILESIZE:__ Maps to php.ini 'upload_max_filesize'

__PHP_MAX_FILE_UPLOADS:__ Maps to php.ini 'max_file_uploads'

__PHP_ALLOW_URL_FOPEN:__ Maps to php.ini 'allow_url_fopen'

__PHP_ALLOW_URL_INCLUDE:__ Maps to php.ini 'allow_url_include'

__PHP_DEFAULT_SOCKET_TIMEOUT:__ Maps to php.ini 'default_socket_timeout'

__PHP_DATE_TIMEZONE:__ Maps to php.ini 'date.timezone'

__PHP_PDO_MYSQL_CACHE_SIZE:__ Maps to php.ini 'pdo_mysql.cache_size'

__PHP_PDO_MYSQL_DEFAULT_SOCKET:__ Maps to php.ini 'pdo_mysql.default_socket'

__PHP_SESSION_SAVE_HANDLER:__ Maps to php.ini 'session.save_handler'

__PHP_SESSION_SAVE_PATH:__ Maps to php.ini 'session.save_path'

__PHP_SESSION_USE_STRICT_MODE:__ Maps to php.ini 'session.use_strict_mode'

__PHP_SESSION_USE_COOKIES:__ Maps to php.ini 'session.use_cookies'

__PHP_SESSION_COOKIE_SECURE:__ Maps to php.ini 'session.cookie_secure'

__PHP_SESSION_NAME:__ Maps to php.ini 'session.name'

__PHP_SESSION_COOKIE_LIFETIME:__ Maps to php.ini 'session.cookie_lifetime'

__PHP_SESSION_COOKIE_PATH:__ Maps to php.ini 'session.cookie_path'

__PHP_SESSION_COOKIE_DOMAIN:__ Maps to php.ini 'session.cookie_domain'

__PHP_SESSION_COOKIE_HTTPONLY:__ Maps to php.ini 'session.cookie_httponly'

__PHP_XDEBUG_ENABLED:__ Turns on xdebug (which is not for production really)

</details>

## Usage

### Docker Compose

You can create a `docker-compose.yml` file to keep things nice and simple:

```yml
version: "3"

services:

  mariadb:
    restart: unless-stopped
    image: mariadb:10
    ports:
    # Database port, will be mapped to localhost interface only
    - 127.0.0.1:3308:3306
    volumes:
    # Path to folder of local filesystem with database files
    - ./databases/mariadb:/var/lib/mysql
    environment:
    # Parameters which will be used in application
    - MYSQL_ROOT_PASSWORD=ive_got_no_roots
    - MYSQL_ROOT_HOST=%
    - MYSQL_DATABASE=somedatabase
    - MYSQL_USER=user
    - MYSQL_PASSWORD=user_pass

  application:
    image: evilfreelancer/alpine-apache-php7
    environment:
    # Database parameters of app
    - DB_HOST=mariadb
    - DB_NAME=somedatabase
    - DB_USER=user
    - DB_PASS=user_pass
    # Settings of php.ini
    - PHP_SHORT_OPEN_TAG=On
    - PHP_ERROR_REPORTING=E_ALL
    - PHP_DISPLAY_ERRORS=On
    - PHP_HTML_ERRORS=On
    - PHP_XDEBUG_ENABLED=true
    ports:
    # Web-server HTTP port, will be mapped to all network interfaces
    - 80:80
    volumes:
    # Folder in which will be "public" folder with index.php
    # and other sources of application
    - ./application:/app
```

Where:

* mariadb - Is a container with SQL database
* application - Container based on `evilfreelancer/alpine-apache-php5` image with your application

In folder `./application` will be your application, it should have `./application/public` subfolder with `index.php` because this container is will use `/app/public` path as webroot of Apache.

Then run command below, it will start composition of containers:

```bash
docker-compose up -d
```

### Docker

If you would like to add some files into this container then you can create your own `Dockerfile` based on this image:

```
FROM evilfreelancer/alpine-apache-php7

ADD ["./public", "/app/public"]
RUN composer update --working-dir=/app/public \
 && chown -R apache:apache /app
```

Probably you want to install some packages via NPM by NodeJS:

```
FROM evilfreelancer/alpine-apache-php7

ADD [".", "/app"]
WORKDIR /app

RUN apk update \
 && apk add --update --no-cache nodejs nodejs-npm \
 && npm install \
 && composer update \
 && chown -R apache:apache /app
```

If you want install some additional extension which not already installed:

```
FROM evilfreelancer/alpine-apache-php7

ADD [".", "/app"]
WORKDIR /app

RUN apk update \
 && apk add --update --no-cache php7-amqp \
 && composer update --working-dir=/app \
 && chown -R apache:apache /app
```

## Where do I put my files?

You can place them in the /app folder, but your main `index.php` file must be in /app/public.
This allows you to have your src files and other outside the public directory.

### Important folders and files inside container

- Apache WebRoot - /app/public/
- Apache Configs - /etc/apache2/
- Apache Logs - /var/log/apache2/
- Composer - /usr/local/bin/composer
- PHP.ini - /etc/php7/php.ini
