FROM php:7.0

STOPSIGNAL SIGINT

ENV	ADMINER_VERSION 4.3.1
ENV	ADMINER_DOWNLOAD_SHA256 71688c5db4d4e504c48c26ec2966ad85e721ef61e0377c5505cfb50b026d5491
ENV	ADMINER_SRC_DOWNLOAD_SHA256 fd016c93bb3bb8e85ed9cf826064584012c4db83f30c4948d701250e29a6d953
ENV ORACLE_HOME /opt/oracle

RUN	addgroup --system adminer \
  && adduser --system --ingroup adminer adminer \
  && mkdir -p /var/www/html \
  && mkdir -p /var/www/html/plugins-enabled \
  && chown -R adminer:adminer /var/www/html

WORKDIR /var/www/html

COPY instantclient_12_1.zip /tmp

RUN	set -x \
  && export DEBIAN_FRONTEND=noninteractive \
  && apt-get update -y \
  && apt-get install -y unzip libpq-dev libsqlite3-dev libaio1 \
  && unzip /tmp/instantclient_12_1.zip -d /tmp \
  && rm -rf /tmp/instantclient_12_1.zip \
  && mv /tmp/instantclient_12_1 $ORACLE_HOME \
  && ln -s $ORACLE_HOME/libclntsh.so.12.1 $ORACLE_HOME/libclntsh.so \
  && ln -s $ORACLE_HOME/libclntshcore.so.12.1 $ORACLE_HOME/libclntshcore.so \
  && ln -s $ORACLE_HOME/libocci.so.12.1 $ORACLE_HOME/libocci.so \
  && docker-php-ext-configure oci8 --with-oci8=instantclient,$ORACLE_HOME \
  && docker-php-ext-install pdo_mysql pdo_pgsql pdo_sqlite oci8 \
  && apt-get remove -y unzip libpq-dev libsqlite3-dev \
  && rm -rf /var/lib/apt/lists/*

COPY	*.php /var/www/html/

RUN	set -x \
  && curl -fsSL https://github.com/vrana/adminer/releases/download/v$ADMINER_VERSION/adminer-$ADMINER_VERSION-en.php -o adminer.php \
  && echo "$ADMINER_DOWNLOAD_SHA256  adminer.php" | sha256sum -c - \
  && curl -fsSL https://github.com/vrana/adminer/archive/v$ADMINER_VERSION.tar.gz -o source.tar.gz \
  && echo "$ADMINER_SRC_DOWNLOAD_SHA256  source.tar.gz" | sha256sum -c - \
  && tar xzf source.tar.gz --strip-components=1 "adminer-$ADMINER_VERSION/designs/" "adminer-$ADMINER_VERSION/plugins/" \
  && rm source.tar.gz

COPY	entrypoint.sh /usr/local/bin/
ENTRYPOINT	[ "entrypoint.sh", "docker-php-entrypoint" ]

USER	adminer
CMD	[ "php", "-S", "[::]:8080", "-t", "/var/www/html" ]

EXPOSE 8080
