FROM php:7.0-apache
ENV TERM=xterm
LABEL maintainer="Gary Ritchie <gary.t.ritchie@gmail.com>" \
      version="0.16.1-php7.0"

# Install base requirements & sensible defaults + required PHP extensions
RUN echo "deb http://ftp.debian.org/debian jessie-backports main" >> /etc/apt/sources.list \
    && apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
        less \
        libpng12-dev \
        libjpeg-dev \
        libxml2-dev \
        mariadb-client \
        unzip \
        sudo \
        vim \
        wget \
        zip \
    && DEBIAN_FRONTEND=noninteractive apt-get -t jessie-backports install -y \
        python-certbot-apache \
    && rm -rf /var/lib/apt/lists/* \
    && docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr \
    && docker-php-ext-install \
        exif \
        gd \
        mysqli \
        opcache \
        soap \
        zip \
    # See https://secure.php.net/manual/en/opcache.installation.php
    && echo 'memory_limit = 512M' > /usr/local/etc/php/php.ini \
    && { \
        echo 'opcache.memory_consumption=128'; \
        echo 'opcache.interned_strings_buffer=8'; \
        echo 'opcache.max_accelerated_files=4000'; \
        echo 'opcache.revalidate_freq=2'; \
        echo 'opcache.fast_shutdown=1'; \
        echo 'opcache.enable_cli=1'; \
    } > /usr/local/etc/php/conf.d/opcache-recommended.ini

# install xdebug and enable it. This block of code goes through the installion from source and compiling steps found
# on the xdebug website
# https://xdebug.org/docs/install
RUN cd /tmp \
    && wget https://xdebug.org/files/xdebug-2.5.0.tgz \
    && tar -zxvf xdebug-2.5.0.tgz \
    && cd xdebug-2.5.0 \
    && /usr/local/bin/phpize \
    && ./configure --enable-xdebug --with-php-config=/usr/local/bin/php-config \
    && make \
    && cp modules/xdebug.so /usr/local/lib/php/extensions/no-debug-non-zts-20151012/

# add xdebug configurations
RUN { \
        echo '[xdebug]'; \
        echo 'zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20151012/xdebug.so'; \
        echo 'xdebug.remote_enable=on'; \
        echo 'xdebug.remote_autostart=on'; \
        echo 'xdebug.remote_connect_back=off'; \
        echo 'xdebug.remote_handler=dbgp'; \
        echo 'xdebug.profiler_enable=0'; \
        echo 'xdebug.profiler_enable_trigger=1'; \
        echo 'xdebug.profiler_output_dir="/var/www/html"'; \
        echo 'xdebug.remote_port=9001'; \
        echo 'xdebug.remote_host=172.254.254.254'; \
    } > /usr/local/etc/php/conf.d/xdebug.ini

# add the application to the container outside of the web root
# COPY app /app

# Install wp-cli, configure apache, add scripts, create install directory & symlink
RUN curl -s \
        -o /usr/local/bin/wp https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar \
        -o /run.sh https://raw.githubusercontent.com/luxcentric/wordpress-starter/master/run.sh \
    && chmod +x /usr/local/bin/wp /run.sh \
    && curl -s \
        https://raw.githubusercontent.com/wp-cli/wp-cli/master/utils/wp-completion.bash | \
        sed -e "s/wp cli completions/wp cli completions --allow-root/" > /etc/bash_completion.d/wp-cli \
    && sed -i 's/AllowOverride None/AllowOverride All/g' /etc/apache2/apache2.conf \
    && echo "ServerName localhost" | tee /etc/apache2/conf-available/fqdn.conf && a2enconf fqdn \
    && a2enmod rewrite expires \
    && a2enmod ssl \
    && service apache2 restart \
    && mkdir -p /app ~/.wp-cli \
    && rm -fr /var/www/html \
    && ln -s /app /var/www/html

WORKDIR /app
EXPOSE 80 443
CMD ["/run.sh"]
