#
#--------------------------------------------------------------------------
# Image Setup
#--------------------------------------------------------------------------
#

ARG PHP_VERSION=${PHP_VERSION}
# FROM php:${PHP_VERSION}-alpine
FROM php:${PHP_VERSION}

LABEL maintainer="Mahmoud Zalt <mahmoud@zalt.me>"

RUN  apt-get update -yqq && apt-get install -yqq wget \
  curl \
  git \
  libmcrypt-dev \
  libxml2-dev \
  autoconf \
  supervisor \
  procps

RUN docker-php-ext-install mbstring pdo tokenizer xml pcntl
RUN pecl channel-update pecl.php.net && pecl install  mcrypt-1.0.1

#Install BCMath package:
ARG INSTALL_BCMATH=false
RUN if [ ${INSTALL_BCMATH} = true ]; then \
  docker-php-ext-install bcmath \
  ;fi

# Install PostgreSQL drivers:
ARG INSTALL_PGSQL=false
RUN if [ ${INSTALL_PGSQL} = true ]; then \
   apt-get update -yqq && apt-get install -yqq postgresql-dev \
  && docker-php-ext-install pdo_pgsql \
  ;fi

#
#--------------------------------------------------------------------------
# Optional Supervisord Configuration
#--------------------------------------------------------------------------
#
# Modify the ./supervisor.conf file to match your App's requirements.
# Make sure you rebuild your container with every change.
#

COPY supervisord.conf /etc/supervisord.conf

ENTRYPOINT ["/usr/bin/supervisord", "-n", "-c",  "/etc/supervisord.conf"]

#
#--------------------------------------------------------------------------
# Optional Software's Installation
#--------------------------------------------------------------------------
#
# If you need to modify this image, feel free to do it right here.
#
# -- Your awesome modifications go here -- #
#####################################
# OCI8:
#####################################
ARG INSTALL_OCI8=false
ENV INSTALL_OCI8 ${INSTALL_OCI8}

ENV LD_LIBRARY_PATH /usr/local/instantclient/

COPY ./instantclient-basic-linux.x64-12.2.0.1.0.zip /tmp/
COPY ./instantclient-sdk-linux.x64-12.2.0.1.0.zip /tmp/

RUN if [ ${INSTALL_OCI8} = true ]; then \
    # apt-get update && \
    # apt-get install -y zip unzip libaio1 wget && \
     apt-get update -yqq && apt-get install -yqq zip unzip libaio-dev wget && \
    # wget "https://s3-sa-east-1.amazonaws.com/4success-assets/instantclient-basic-linux.x64-12.2.0.1.0.zip" -P "/tmp" && \
    # wget "https://s3-sa-east-1.amazonaws.com/4success-assets/instantclient-sdk-linux.x64-12.2.0.1.0.zip" -P "/tmp" && \
    unzip /tmp/instantclient-basic-linux.x64-12.2.0.1.0.zip -d /usr/local/ && \
    unzip /tmp/instantclient-sdk-linux.x64-12.2.0.1.0.zip -d /usr/local/ && \
    ln -s /usr/local/instantclient_12_2 /usr/local/instantclient && \
    ln -s /usr/local/instantclient/libclntsh.so.12.1 /usr/local/instantclient/libclntsh.so && \
    export LD_LIBRARY_PATH=/usr/local/instantclient && \
    docker-php-ext-configure oci8 --with-oci8=instantclient,/usr/local/instantclient && \
    docker-php-ext-install oci8 && \
    rm -rf /usr/local/*.zip \
;fi

#
#--------------------------------------------------------------------------
# Check PHP version
#--------------------------------------------------------------------------
#

RUN php -v | head -n 1 | grep -q "PHP ${PHP_VERSION}."

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

WORKDIR /etc/supervisor/conf.d/
