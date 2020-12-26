ARG PHP_IMAGE
FROM ${PHP_IMAGE:-'php:7.2-fpm'}
#  设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak \
    && echo 'deb http://mirrors.aliyun.com/debian/ buster main non-free contrib' > /etc/apt/sources.list \
    && echo 'deb http://mirrors.aliyun.com/debian-security buster/updates main' >> /etc/apt/sources.list \
    && echo 'deb http://mirrors.aliyun.com/debian/ buster-updates main non-free contrib' >> /etc/apt/sources.list \
    && echo 'deb http://mirrors.aliyun.com/debian/ buster-backports main non-free contrib' >> /etc/apt/sources.list \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
        libmagickwand-dev \
        libmcrypt-dev \
    && rm -rf /var/lib/apt/lists/* \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-ext-install mysqli \
    && pecl install imagick-3.4.4 && pecl install mcrypt-1.0.2 && pecl install redis-5.1.1 \
    && pecl install xdebug-2.7.2 && pecl install swoole-4.4.3 \
    && docker-php-ext-enable imagick mcrypt redis xdebug swoole \
    && curl -sS https://getcomposer.org/installer |php -- --install-dir=/usr/local/bin --filename=composer
    && docker-php-source delete \
    && rm -r /tmp/* /var/cache/* /var/www/html/*

WORKDIR /data
