FROM centos:7.8.2003
MAINTAINER chenchao

RUN yum -y install epel-release && \
    yum -y install gcc gcc-c++ gd-devel libxml2 libxml2-devel libcurl-devel \
    openssl openssl-devel curl curl-devel libjpeg libjpeg-devel libpng  \
    freestyle freestyle-devel pcre pcre-devel libxslt libxslt-devel bzip2 bzip2-devel \
    php-mcrypt libmcrypt libmcrypt-devel freetype-devel libpng-devel libsqlite3x-devel oniguruma-devel \
    openldap-devel libtidy-devel gmp-devel autoconf re2c libsodium-devel cmake && \
    yum clean all -y

ARG PHP_VERSION
# Timezone
RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo 'Asia/Shanghai' > /etc/timezone
# PHP
ADD source/php-${PHP_VERSION}.tar.bz2 /tmp/
ADD source/etc.tar.gz /tmp/php-${PHP_VERSION}/
RUN cd /tmp/php-${PHP_VERSION} \
    && ./configure --prefix=/opt/app/php7 \
    --with-config-file-path=/opt/app/php7/etc \
    --disable-ipv6 \
    --enable-bcmath \
    --enable-dba \
    --enable-ftp \
    --enable-mbstring \
    --enable-sockets \
    --enable-exif \
    --enable-opcache \
    --enable-pcntl \
    --enable-soap \
    --enable-zip \
    --with-curl \
    --with-iconv \
    --with-gettext \
    --enable-gd \
    --with-gmp \
    --with-freetype-dir \
    --with-jpeg-dir \
    --with-png-dir \
    --with-mcrypt \
    --with-mhash \
    --with-openssl \
    --with-tidy \
    --with-zlib \
    --with-mysqli=mysqlnd \
    --with-pdo-mysql=mysqlnd \
    --enable-fpm \
    --enable-xml \
    && make && make install \
    && cp ./sapi/fpm/init.d.php-fpm /opt/app/php7/sbin/init.d.php-fpm \
    && chmod +x /opt/app/php7/sbin/init.d.php-fpm \
    && groupadd -g 2000 httpd \
    && useradd -g 2000 -u 2000 httpd \
    && rm -rf /opt/app/php7/etc \
    && mv ./etc/ /opt/app/php7/ \
    && mkdir -p /opt/logs/php7 && chown httpd /opt/logs/php7 \
    && rm -r /tmp/php-${PHP_VERSION}

# Redis extension
ADD source/redis-5.2.2.tgz /tmp/
RUN cd /tmp/redis-5.2.2 \
    && /opt/app/php7/bin/phpize \
    && ./configure --with-php-config=/opt/app/php7/bin/php-config \
    && make && make install \
    && rm -rf /tmp/redis-5.2.2

# Swoole extension
ADD source/swoole-4.5.8.tar.gz /tmp/
RUN cd /tmp && \
    mv swoole-src* swoole-src && \
    cd swoole-src && \
    /opt/app/php7/bin/phpize && \
    ./configure \
    --with-php-config=/opt/app/php7/bin/php-config \
    --enable-coroutine \
    --enable-openssl  \
    --enable-http2  \
    --enable-async-redis \
    --enable-sockets \
    --enable-mysqlnd && \
    make clean && make && make install && \
    rm -rf /tmp/swoole-src

# Apcu extension
ADD source/apcu-5.1.18.tgz /tmp/
RUN cd /tmp/apcu-5.1.18 \
    && /opt/app/php7/bin/phpize \
    && ./configure --with-php-config=/opt/app/php7/bin/php-config \
    && make && make install \
    && rm -rf /tmp/apcu-5.1.18

# Amqp extension
ADD source/amqp-1.9.4.tgz /tmp/
ADD source/rabbitmq-c-0.9.0.tar.gz /tmp/
RUN cd /tmp/rabbitmq-c-0.9.0 \
    && mkdir build \
    && cd build \
    && cmake .. -DCMAKE_INSTALL_PREFIX=/opt/app/rabbitmq-c -DBUILD_TOOLS:BOOL=OFF \
    && make all && make install \
    && cd /opt/app/rabbitmq-c \
    && ln -s lib64 lib \
    && cd /tmp/amqp-1.9.4 \
    && /opt/app/php7/bin/phpize \
    && ./configure \
    --with-php-config=/opt/app/php7/bin/php-config \
    --with-amqp --with-librabbitmq-dir=/opt/app/rabbitmq-c \
    && make && make install \
    && rm -rf /tmp/rabbitmq-c-0.9.0 \
    && rm -rf /tmp/amqp-1.9.4

# Mongo extension
ADD source/mongodb-1.5.3.tgz /tmp/
RUN cd /tmp/mongodb-1.5.3 \
    && /opt/app/php7/bin/phpize \
    && ./configure --with-php-config=/opt/app/php7/bin/php-config \
    && make && make install \
    && rm -r /tmp/mongodb-1.5.3 \
    && mkdir -p /opt/app/php7/tmp/sessions

EXPOSE 9000

CMD ["/opt/app/php7/sbin/php-fpm", "--nodaemonize"]
