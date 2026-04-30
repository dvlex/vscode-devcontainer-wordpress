FROM php:8.4-apache

RUN mv "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"
# Enable .htaccess files & mod_rewrite
RUN sed -i '/<Directory \/var\/www\/>/,/<\/Directory>/ s/AllowOverride None/AllowOverride all/' /etc/apache2/apache2.conf
RUN a2enmod rewrite

#Most of this comes directly from the WordPress docker image
#We don't inherit from it becase it has a VOLUME that causes issues
RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends ghostscript git less ssh-client mariadb-client nodejs npm sudo; \
    rm -rf /var/lib/apt/lists/*

# install the PHP extensions, including XDebug
RUN set -ex; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		libfreetype6-dev \
		libjpeg-dev \
		libmagickwand-dev \
		libpng-dev \
		libzip-dev \
	;

RUN	docker-php-ext-configure gd --with-freetype --with-jpeg; \
	docker-php-ext-install -j "$(nproc)" \
		bcmath \
		exif \
		gd \
		mysqli \
		opcache \
		zip \
	;

RUN	pecl install imagick; \
  	pecl install xdebug; \
	docker-php-ext-enable imagick xdebug;\
	rm -rf /var/lib/apt/lists/*

#XDebug settings
RUN echo "[XDebug]\nxdebug.remote_enable = 1\nxdebug.remote_autostart = 1" > $PHP_INI_DIR/conf.d/xdebug.ini

#WP recommended PHP settings
RUN { \
	echo 'opcache.memory_consumption=128'; \
	echo 'opcache.interned_strings_buffer=8'; \
	echo 'opcache.max_accelerated_files=4000'; \
	echo 'opcache.revalidate_freq=2'; \
	echo 'opcache.fast_shutdown=1'; \
} > /usr/local/etc/php/conf.d/opcache-recommended.ini

RUN echo "memory_limit=512M" > /usr/local/etc/php/conf.d/memory-limit.ini;

#install WP-CLI
RUN curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar \
    && chmod +x wp-cli.phar \
    && mv wp-cli.phar /usr/local/bin/wp

#install Composer
WORKDIR /tmp
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
		&& php composer-setup.php --quiet --install-dir=/usr/local/bin --filename=composer

ARG HOST_UID=1000
ARG HOST_GID=1000
ARG HOST_USER=vscode

#add user using host UID/GID and username when provided
RUN groupadd -g ${HOST_GID} hostgroup || true && \
	getent group ${HOST_GID} >/dev/null 2>&1 || groupadd -g ${HOST_GID} hostgroup || true && \
	id -u ${HOST_USER} >/dev/null 2>&1 || useradd -m -u ${HOST_UID} -g ${HOST_GID} -s /bin/bash ${HOST_USER} || true && \
	usermod -aG www-data ${HOST_USER} || true && \
	# add to sudo group and allow passwordless sudo for the host user
	usermod -aG sudo ${HOST_USER} || true && \
	echo "${HOST_USER} ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/host_user && \
	chmod 0440 /etc/sudoers.d/host_user || true

WORKDIR /var/www/html

USER www-data

#Download WP
RUN wp core download

# add dirs for plugin and theme dev
RUN mkdir -p /var/www/html/wp-content/plugins/plugin-dev; \
	mkdir -p /var/www/html/wp-content/themes/theme-dev;

USER root
RUN chown -R www-data:www-data /var/www/html/
RUN chmod g+w -R /var/www/html/
RUN find /var/www/html/ -type d -exec chmod g+s {} \;