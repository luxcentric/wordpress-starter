FROM phpstorm/php-72-apache-xdebug-27
ENV TERM=xterm
LABEL version="0.16.2-php7.2"

# Install base requirements & sensible defaults + required PHP extensions
RUN echo "deb http://ftp.debian.org/debian $(sed -n 's/^VERSION=.*(\(.*\)).*/\1/p' /etc/os-release)-backports main" >> /etc/apt/sources.list \
    && apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
        bash-completion \
        less \
        libjpeg-dev \
        libpng-dev \
        libxml2-dev \
        mariadb-client \
        sudo \
        unzip \
        vim \
        zip \
    && DEBIAN_FRONTEND=noninteractive apt-get -t $(sed -n 's/^VERSION=.*(\(.*\)).*/\1/p' /etc/os-release)-backports install -y \
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
    && echo 'memory_limit = 512M' >> /usr/local/etc/php/php.ini \
    && echo 'xdebug.profiler_enable=0' >> /usr/local/etc/php/php.ini \
    && echo 'xdebug.profiler_enable_trigger=1' >> /usr/local/etc/php/php.ini \
    && echo 'xdebug.profiler_output_dir=/profile' >> /usr/local/etc/php/php.ini \
    && { \
        echo 'opcache.memory_consumption=128'; \
        echo 'opcache.interned_strings_buffer=8'; \
        echo 'opcache.max_accelerated_files=4000'; \
        echo 'opcache.revalidate_freq=2'; \
        echo 'opcache.fast_shutdown=1'; \
        echo 'opcache.enable_cli=1'; \
    } > /usr/local/etc/php/conf.d/opcache-recommended.ini

# add the application to the container outside of the web root
# COPY app /app
COPY run.sh /run.sh

# Install wp-cli, configure apache, add scripts, create install directory & symlink
RUN curl -s \
        -o /usr/local/bin/wp https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar \
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
