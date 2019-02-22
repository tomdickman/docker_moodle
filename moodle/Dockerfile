FROM ubuntu:18.04

ARG DEBIAN_FRONTEND=noninteractive

# Install all of our dependencies
RUN apt-get update -y \
    && apt-get install -y nginx \
    php \
    php-bcmath \
    # Dev for running XHProf profiler
    php7.2-dev \
    php7.2-fpm \
    # Mongo DB extention for running XHGui UI
    php-mongodb \
    php-pgsql \
    php-xdebug \
    php-cli \
    curl \
    php-xml \
    php-curl \
    php-zip \
    php-gd \
    php-mbstring \
    php-soap \
    php-intl \
    php-xmlrpc \
    php-ldap \
    locales \
    unzip \
    git \
    php-mysql \
    cron \
    mongodb \
    vim

COPY mongodb.ini /etc/php/7.2/mods-available/mongodb.ini
RUN pecl install mongodb

RUN locale-gen en_AU.UTF-8

# Install composer, we'll need this for packages
RUN cd /usr/local/lib/ && curl -sS https://getcomposer.org/installer | php && mv composer.phar /usr/local/bin/composer

# Install Tideways/XHProf
RUN cd / && git clone https://github.com/tideways/php-xhprof-extension.git xhprof
RUN cd /xhprof && phpize && ./configure && make && make install
COPY tideways_xhprof.ini /etc/php/7.2/mods-available/tideways_xhprof.ini
RUN ln -s /etc/php/7.2/mods-available/tideways_xhprof.ini /etc/php/7.2/cli/conf.d/10-tideways_xhprof.ini
RUN ln -s /etc/php/7.2/mods-available/tideways_xhprof.ini /etc/php/7.2/fpm/conf.d/10-tideways_xhprof.ini

# Install XHGui
RUN cd / && mkdir /xhgui && chown -R www-data:www-data /xhgui
RUN git clone https://github.com/perftools/xhgui.git /xhgui && chmod -R 0777 /xhgui
RUN cd /xhgui && curl -sS https://getcomposer.org/installer | php && composer install
RUN php /xhgui/install.php
COPY xhgui.conf /etc/nginx/sites-available/xhgui.conf
COPY xhgui-config.php /xhgui/config/config.php
RUN ln -s /etc/nginx/sites-available/xhgui.conf /etc/nginx/sites-enabled/xhgui.conf

# Make our Moodle folder to mount our Moodle repo to
RUN mkdir /siteroot

# Install PHP Unit
RUN cd /siteroot && composer require phpunit/phpunit

# Make our Moodledata folder and datafolder for testing
RUN mkdir /var/lib/sitedata && chmod 0777 /var/lib/sitedata
RUN mkdir /var/lib/testsitedata && chmod 0777 /var/lib/testsitedata

# Copy our local certificates into the container
COPY moodle.crt /etc/nginx/conf.d/moodle.crt
COPY moodle.pem /etc/nginx/conf.d/moodle.pem

# Replace the Nginx default server files with our own
RUN rm /etc/nginx/sites-available/default
COPY nginx-site.conf /etc/nginx/sites-available/default

RUN rm /etc/nginx/nginx.conf
COPY nginx.conf /etc/nginx/nginx.conf

# Add our xdebug for debugging
COPY 99-xdebug.ini /etc/php/7.2/mods-available/xdebug.ini
COPY php.ini /etc/php/7.2/fpm/php.ini

# Start up the php-fpm server and nginx - away we go!
CMD service php7.2-fpm start && nginx