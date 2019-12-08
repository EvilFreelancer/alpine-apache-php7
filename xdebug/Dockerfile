FROM alpine:3.10

# Install basics first
RUN apk add --update --no-cache \
    bash curl ca-certificates openssl openssh-keygen openssh-client git tzdata openntpd nano \
    apache2 php7-apache2 php7 \
    php7-phar \
    php7-json \
    php7-iconv \
    php7-openssl \
    php7-ftp \
    php7-xdebug \
    php7-mcrypt \
    php7-mbstring \
    php7-soap \
    php7-gmp \
    php7-pdo_odbc \
    php7-dom \
    php7-pdo \
    php7-zip \
    php7-mysqli \
    php7-sqlite3 \
    php7-pdo_pgsql \
    php7-bcmath \
    php7-gd \
    php7-odbc \
    php7-pdo_mysql \
    php7-pdo_sqlite \
    php7-gettext \
    php7-xmlreader \
    php7-xmlwriter \
    php7-tokenizer \
    php7-xmlrpc \
    php7-bz2 \
    php7-pdo_dblib \
    php7-curl \
    php7-ctype \
    php7-session \
    php7-redis \
    php7-exif \
    php7-simplexml \
    php7-fileinfo \
    php7-pecl-xdebug

# Install composer
RUN curl -sS https://getcomposer.org/installer | php && mv composer.phar /usr/local/bin/composer

# Add apache to run and configure
RUN mkdir -p /run/apache2 \
 && sed -i "s/#LoadModule\ rewrite_module/LoadModule\ rewrite_module/" /etc/apache2/httpd.conf \
 && sed -i "s/#LoadModule\ session_module/LoadModule\ session_module/" /etc/apache2/httpd.conf \
 && sed -i "s/#LoadModule\ session_cookie_module/LoadModule\ session_cookie_module/" /etc/apache2/httpd.conf \
 && sed -i "s/#LoadModule\ session_crypto_module/LoadModule\ session_crypto_module/" /etc/apache2/httpd.conf \
 && sed -i "s/#LoadModule\ deflate_module/LoadModule\ deflate_module/" /etc/apache2/httpd.conf \
 && sed -i "s#^DocumentRoot \".*#DocumentRoot \"/app/public\"#g" /etc/apache2/httpd.conf \
 && sed -i "s#/var/www/localhost/htdocs#/app/public#" /etc/apache2/httpd.conf \
 && printf "\n<Directory \"/app/public\">\n\tAllowOverride All\n</Directory>\n" >> /etc/apache2/httpd.conf

ADD ["entrypoint.sh", "/"]

RUN mkdir -p /app/public \
 && chown -R apache:apache /app \
 && chmod -R 755 /app \
 && chmod +x /entrypoint.sh

ADD xdebug.ini /etc/php7/conf.d/

EXPOSE 80
ENTRYPOINT ["/entrypoint.sh"]
