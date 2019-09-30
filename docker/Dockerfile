FROM debian:buster-slim AS sourcecode
RUN apt-get update && apt-get install  --no-install-recommends --no-install-suggests -y \
git ca-certificates
RUN git clone --depth 1 https://github.com/thibaud-rohmer/PhotoShow.git /var/www/PhotoShow && \
    rm -rf /var/www/PhotoShow/.git

FROM debian:buster-slim
MAINTAINER CYOSP <cyosp@cyosp.com>

COPY --from=sourcecode /var/www/PhotoShow /var/www/PhotoShow
RUN apt-get update && \
    apt-get install --no-install-recommends --no-install-suggests -y \
	zip \
	python3-certbot-nginx \
	php-fpm \
	php-gd \
	php-xml \
	libgd-dev \
	supervisor \
	exiftran

RUN mkdir -p /opt/PhotoShow/Photos /opt/PhotoShow/generated && chown -R www-data:www-data /opt/PhotoShow/Photos /opt/PhotoShow/generated

RUN sed -i -e 's/$config->photos_dir.\+/$config->photos_dir = "\/opt\/PhotoShow\/Photos";/' /var/www/PhotoShow/config.php
RUN sed -i -e 's/$config->ps_generated.\+/$config->ps_generated = "\/opt\/PhotoShow\/generated";/' /var/www/PhotoShow/config.php

RUN rm -f /etc/nginx/sites-enabled/default && echo "daemon off;" >> /etc/nginx/nginx.conf && mkdir -p /var/run/php
ADD nginx.conf /etc/nginx/conf.d/photoshow.conf

RUN sed -i -e 's/^.\+daemonize.\+$/daemonize = no/' /etc/php/7.3/fpm/php-fpm.conf
ADD fpm.conf /etc/php/7.3/fpm/pool.d/photoshow.conf

RUN sed -i -e 's/^\(\[supervisord\]\)$/\1\nnodaemon=true/' /etc/supervisor/supervisord.conf
ADD supervisor.conf /etc/supervisor/conf.d/photoshow.conf

ADD run_certbot.sh /root
RUN chmod u+x /root/run_certbot.sh

VOLUME ["/opt/PhotoShow", "/var/log", "/etc/letsencrypt"]
EXPOSE 80
EXPOSE 443

CMD ["/usr/bin/supervisord"]
