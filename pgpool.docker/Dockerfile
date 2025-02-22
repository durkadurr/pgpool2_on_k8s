FROM alpine:3.21 AS builder

ARG PGPOOL_VER

ENV PGPOOL_INSTALL_DIR /opt/pgpool-II

# Install packages
RUN set -eux \
    \
    && apk add --no-cache --virtual fetch-dependencies \
    ca-certificates \
    tar \
    \
    && apk add --no-cache --virtual build-dependencies \
    bison \
    flex \
    file \
    gcc \
    g++ \
    libbsd-doc \
    linux-headers \
    make \
    patch \
    openssl-dev \
    \
    && apk add --no-cache \
    bash \
    postgresql-dev

COPY fix_compile_error.patch /tmp/pgpool/

# Install Pgpool-II
RUN set -eux \
    \
    && wget -O /tmp/pgpool.tar.gz "https://pgpool.net/mediawiki/images/pgpool-II-${PGPOOL_VER}.tar.gz" \
    && tar -zxf /tmp/pgpool.tar.gz -C /tmp/pgpool --strip-components 1 \
    && cd /tmp/pgpool \
    && patch -p1 < fix_compile_error.patch \
    && ./configure \
    --prefix=${PGPOOL_INSTALL_DIR} \
    --with-openssl \
    && make -j "$(nproc)" \
    && make install


FROM alpine:3.21

ENV PGPOOL_INSTALL_DIR /opt/pgpool-II
ENV PGPOOL_CONF_VOLUME /config

COPY --from=builder ${PGPOOL_INSTALL_DIR} ${PGPOOL_INSTALL_DIR}

# Create postgres user used to start Pgpool-II
RUN set -ex; \
    addgroup -g 70 -S postgres; \
    adduser -u 70 -S -D -G postgres -H -h /var/lib/pgsql -s /bin/sh postgres; \
    mkdir -p /var/lib/pgsql; \
    chown -R postgres:postgres /var/lib/pgsql

# Install packages
RUN set -eux \
    \
    && apk add --no-cache \
    bash \
    postgresql \
    openssl \
    sed \
    sudo

# Install Pgpool-II
RUN set -eux \
    \
    && mkdir /var/run/pgpool \
    && mkdir /var/run/postgresql \
    && chown -R postgres:postgres /var/run/pgpool /var/run/postgresql ${PGPOOL_INSTALL_DIR} \
    && echo 'postgres ALL=NOPASSWD: /sbin/ip' | sudo EDITOR='tee -a' visudo >/dev/null 2>&1 \
    && echo 'postgres ALL=NOPASSWD: /usr/sbin/arping' | sudo EDITOR='tee -a' visudo >/dev/null 2>&1 \
    && rm -rf /tmp/*

COPY entrypoint.sh ${PGPOOL_INSTALL_DIR}/bin/
COPY start.sh ${PGPOOL_INSTALL_DIR}/bin/

USER postgres

ENTRYPOINT ["/opt/pgpool-II/bin/entrypoint.sh"]
CMD ["/opt/pgpool-II/bin/start.sh"]

EXPOSE 9999
