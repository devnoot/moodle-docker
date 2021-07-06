FROM php:8-apache

RUN mkdir /usr/src/php
RUN tar --file /usr/src/php.tar.xz --extract --strip-components=1 --directory /usr/src/php
RUN cp /usr/src/php/php.ini-production /usr/local/etc/php/php.ini

# Fix bad proxy? Seems like a Docker bug
# https://forums.docker.com/t/hash-sum-mismatch-writing-more-data-as-expected/45940/2
COPY ./badproxyfix /etc/apt/apt.conf.d/99-badproxyfix

# Ask apt server to provide uncached files
RUN apt -o Acquire::https::No-Cache=True -o Acquire::http::No-Cache=True update
RUN apt-get update
RUN apt-get -y upgrade

# Install moodle php dependencies
RUN docker-php-ext-install pdo pdo_mysql mysqli
RUN apt-get install -y libzip-dev sendmail libpng-dev libicu-dev apt-utils re2c g++ zlib1g-dev libmcrypt-dev libxml2-dev build-essential wget poppler-utils
RUN apt-get clean
RUN docker-php-ext-install zip
RUN docker-php-ext-install gd
RUN docker-php-ext-install intl
RUN docker-php-ext-install opcache
COPY opcache.ini /usr/local/etc/php/conf.d/opcache.ini
COPY php-overrides.ini /usr/local/etc/php/conf.d/php-overrides.ini

# Install composer.
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
RUN php -r "if (hash_file('sha384', 'composer-setup.php') === '756890a4488ce9024fc62c56153228907f1545c228516cbf63f885e036d37e9a59d27d63f46af1d4d07ee0f76181c7d3') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
RUN php composer-setup.php
RUN php -r "unlink('composer-setup.php');"
RUN mv composer.phar /usr/local/bin/composer

# Install codesniffer.
RUN wget -O /usr/local/bin/phpcs https://squizlabs.github.io/PHP_CodeSniffer/phpcs.phar
RUN wget -O /usr/local/bin/phpcbf https://squizlabs.github.io/PHP_CodeSniffer/phpcbf.phar
RUN chmod +x /usr/local/bin/phpcs
RUN chmod +x /usr/local/bin/phpcbf

# Clone to temp.
RUN wget -O /tmp/moodle-local_codechecker.tar.gz https://github.com/moodlehq/moodle-local_codechecker/archive/refs/tags/v3.0.1.tar.gz

# Unzip to /usr/local/lib.
RUN tar -zvxf /tmp/moodle-local_codechecker.tar.gz -C /usr/local/lib
RUN rm -rf /tmp/moodle-local_codechecker.tar.gz
RUN phpcs --config-set installed_paths /usr/local/lib/moodle-local_codechecker-3.0.1

# Set the default code style.
RUN phpcs --config-set default_standard moodle

# Run the moodle installation script.
# RUN php /var/www/html/admin/cli/install.php --agree-license --dataroot=/var/www/moodledata --wwwroot=http://localhost:8079 --dbtype=mariadb --dbhost=localhost --dbname=moodle --dbuser=root --dbpass=password --fullname=Moodle --shortname=moodle --adminuser=moodle --adminpass=moodle --non-interactive
