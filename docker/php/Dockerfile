FROM ubuntu:14.04

RUN apt-get update \
 && apt-get install -y software-properties-common \
 && add-apt-repository ppa:costamagnagianfranco/ettercap-stable-backports \
 && apt-get update \
 && apt-get install -y libvirtodbc0 raptor-utils curl \
 && apt-get install -y php5-fpm php5-cli php5-mysql php5-odbc php5-xdebug php5-curl \
 && apt-get install -y php5-common php5-tidy php5-xsl php5-xmlrpc php5-gd php5-memcache php5-memcached \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

RUN set -ex \
 && cd /etc/php5/fpm/pool.d \
 && rm www.conf \
 && { \
  echo '[global]'; \
  echo 'daemonize = no'; \
  echo; \
  echo '[www]'; \
  echo 'user = www-data'; \
  echo 'group = www-data'; \
  echo 'listen = 9000'; \
  echo 'pm = dynamic'; \
  echo 'pm.max_children = 5'; \
  echo 'pm.start_servers = 2'; \
  echo 'pm.min_spare_servers = 1'; \
  echo 'pm.max_spare_servers = 3'; \
  echo 'chdir = /'; \
  echo; \
  echo 'php_flag[display_errors] = on'; \
 } | tee docker.conf

RUN set -ex \
 && { \
  echo '[ODBC Data Sources]'; \
  echo 'VOS = Virtuoso'; \
  echo 'VOS_TEST = Virtuoso Test'; \
  echo; \
  echo '[VOS]'; \
  echo 'Driver = /usr/lib/odbc/virtodbc.so'; \
  echo 'Address = virtuoso-dev:1111'; \
  echo; \
  echo '[VOS_TEST]'; \
  echo 'Driver = /usr/lib/odbc/virtodbc.so'; \
  echo 'Address = virtuoso-test:1111'; \
 } | tee /etc/odbc.ini

EXPOSE 9000
CMD ["/usr/sbin/php5-fpm"]