FROM php:8.4-fpm-alpine

WORKDIR /var/www/html

USER root

# Install system dependencies
RUN apk add --no-cache --virtual .build-deps \
        $PHPIZE_DEPS \
        oniguruma-dev \
        postgresql-dev \
        linux-headers \
    && apk add --no-cache \
        git \
        shadow \
        postgresql-client \
        nodejs \
        npm \
    && pecl install \
        pcov \
        redis \
    && docker-php-ext-enable \
        pcov \
        redis \
    && docker-php-ext-configure pgsql -with-pgsql=/usr/local/pgsql \
    && docker-php-ext-install \
        exif \
        pdo \
        pdo_pgsql \
        intl \
        opcache \
    && apk del -f .build-deps

# Composer
COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer

# Set up global Composer path for www-data
RUN mkdir -p /var/www/.composer && \
    chown -R www-data:www-data /var/www/.composer

# Add Composer binaries to PATH
#ENV PATH="/var/www/.composer/vendor/bin:$PATH"
ENV PATH="/home/www-data/.composer/vendor/bin:$PATH"

# Set up user permissions
ARG PUID=1000
ENV PUID=${PUID}
ARG PGID=1000
ENV PGID=${PGID}

RUN groupmod -o -g ${PGID} www-data && \
    usermod -o -u ${PUID} -g www-data www-data

# Set up working directory permissions
RUN mkdir -p /var/www/html \
    && chown -R www-data:www-data /var/www/html

# Switch to non-root user
USER www-data

# Install Laravel globally
RUN composer global require laravel/installer && \
    laravel --version

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
    CMD php -v || exit 1