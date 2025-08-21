FROM us-east4-docker.pkg.dev/rso-dev-authorized-images-slh5/ltcr-ironbank-repo/opensource/php/php:8.2.28

ENV DRUPAL_DATABASE_HOST=localhost \
    DRUPAL_DATABASE_PORT=5432 \
    DRUPAL_DATABASE_DRIVER=pgsql \
    DRUPAL_DATABASE_PASSWORD=required_but_not_used

USER root

RUN dnf install -y git zip unzip php-pgsql php-opcache

RUN sed -i 's/\(^max_execution_time = 30$\)/max_execution_time = 300/g' /usr/local/etc/php/php.ini \
    && sed -i 's/\(^;curl.cainfo =$\)/curl.cainfo = \/var\/secrets\/devportal.trust.pem/g' /usr/local/etc/php/php.ini \
    && echo "php_admin_value[memory_limit] = 2048M" >> /usr/local/etc/php-fpm.d/www.conf \
    && sed -i 's/\(^;request_terminate_timeout = 0$\)/request_terminate_timeout = 300/g' /usr/local/etc/php-fpm.d/www.conf

USER 1001

WORKDIR /opt/drupal

COPY config/composer.json /opt/drupal/composer.json
RUN php -d memory_limit=-1 /usr/local/bin/composer install -o --working-dir=/opt/drupal --no-interaction

COPY config/settings.php /opt/drupal/web/sites/default/settings.php

USER root

RUN mkdir -p /opt/drupal/web/sites/default/files && \
    mkdir -p /opt/drupal/web/sites/default/private && \
    chown -R 1001:1001 /opt/drupal

COPY config/startup.sh /startup.sh
RUN chmod +x /startup.sh

ENV PATH=${PATH}:/opt/drupal/vendor/bin

USER 1001

CMD ["/startup.sh"]