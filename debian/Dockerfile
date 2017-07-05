FROM pihvio/base:debian

MAINTAINER Onni Hakala <onni@keksi.io>

# Build Arguments for openresty/nginx
ARG RESTY_VERSION="1.11.2.3"
ARG RESTY_OPENSSL_VERSION="1.0.2k"

# Nginx Pagespeed version
ARG NPS_RELEASE_NUMBER="1.12.34.2"
ARG NPS_VERSION="${NPS_RELEASE_NUMBER}-stable"

# Fix apt-get and show colors
ARG DEBIAN_FRONTEND=noninteractive
ARG TERM=xterm-color

ARG RESTY_CONFIG_OPTIONS="\
    --with-http_addition_module \
    --with-http_auth_request_module \
    --with-http_flv_module \
    --with-http_gunzip_module \
    --with-http_gzip_static_module \
    --with-http_mp4_module \
    --with-http_random_index_module \
    --with-http_realip_module \
    --with-http_secure_link_module \
    --with-http_slice_module \
    --with-http_ssl_module \
    --with-http_stub_status_module \
    --with-http_sub_module \
    --with-http_v2_module \
    --with-http_geoip_module=dynamic \

    --with-file-aio \
    --with-ipv6 \
    --with-pcre-jit \
    --with-stream \
    --with-stream_ssl_module \
    --with-threads \

    --without-http_autoindex_module \
    --without-http_browser_module \
    --without-http_userid_module \
    --without-mail_pop3_module \
    --without-mail_imap_module \
    --without-mail_smtp_module \
    --without-http_split_clients_module \
    --without-http_uwsgi_module \
    --without-http_scgi_module \
    --without-http_referer_module \

    --without-http_redis_module \

    --user=nginx \
    --group=nginx \

    --sbin-path=/usr/sbin \
    --modules-path=/usr/lib/nginx \

    --prefix=/etc/nginx \
    --conf-path=/etc/nginx/nginx.conf \
    --http-log-path=/var/log/nginx/access.log \
    --error-log-path=/var/log/nginx/error.log \
    --pid-path=/var/run/nginx.pid \
    --lock-path=/var/run/nginx/nginx.lock \

    --http-fastcgi-temp-path=/tmp/nginx/fastcgi \
    --http-proxy-temp-path=/tmp/nginx/proxy \
    --http-client-body-temp-path=/tmp/nginx/client_body \

    --add-module=/tmp/redis-nginx-module-master \
    --add-module=/tmp/ngx_pagespeed-${NPS_VERSION} \
    --with-openssl=/tmp/openssl-${RESTY_OPENSSL_VERSION} \
    "


# These are only needed during the installation
ARG BUILD_DEPS='build-essential wget libreadline-dev libncurses5-dev libpcre3-dev libgeoip-dev zlib1g-dev'
ARG RUNTIME_DEPS='ca-certificates'

# Install base utils
RUN \
    set -x && \
    apt-get update && \
    apt-get -y install $BUILD_DEPS $RUNTIME_DEPS --no-install-recommends && \

    cd /tmp/ && \

    ### Download Tarballs ###
    # Download PageSpeed
    echo "Downloading PageSpeed..." && \
    wget https://github.com/pagespeed/ngx_pagespeed/archive/v${NPS_VERSION}.tar.gz -O - | tar -zx && \

    # psol needs to be inside ngx_pagespeed module
    # Download PageSpeed Optimization Library and extract it to nginx source dir
    cd ngx_pagespeed-${NPS_VERSION}/ && \
    echo "Downloading PSOL..." && \
    wget https://dl.google.com/dl/page-speed/psol/${NPS_RELEASE_NUMBER}-x64.tar.gz -O - | tar -zx && \

    cd /tmp/ && \

    # Download OpenSSL
    echo "Downloading OpenSSL..." && \
    wget https://www.openssl.org/source/openssl-${RESTY_OPENSSL_VERSION}.tar.gz -O - | tar -zx && \

    # Download Openresty bundle
    echo "Downloading openresty..." && \
    wget https://openresty.org/download/openresty-${RESTY_VERSION}.tar.gz -O - | tar -zx && \

    # Download custom redis module with AUTH support
    echo "Downloading ngx_http_redis..." && \
    wget https://github.com/onnimonni/redis-nginx-module/archive/master.tar.gz -O - | tar -zx && \

    # Use all cores available in the builds with -j${NPROC} flag
    readonly NPROC=$(grep -c ^processor /proc/cpuinfo 2>/dev/null || 1)  && \
    echo "using up to $NPROC threads" && \

    ### Configure Nginx ###
    cd openresty-${RESTY_VERSION} && \
    ./configure -j${NPROC} ${_RESTY_CONFIG_DEPS} ${RESTY_CONFIG_OPTIONS} && \

    # Build Nginx
    make -j${NPROC} && \
    make -j${NPROC} install && \

    mkdir -p /var/lib/nginx /var/log/nginx && \

    ## Cleanup
    rm -rf /var/lib/apt/lists/* && \
    apt-get remove --purge -y $BUILD_DEPS && \
    apt-get autoremove -y && \
    rm -rf /tmp/* /var/log/apt/*

RUN \
    # Temp directory
    mkdir /tmp/nginx/ \

    # Symlink modules path to config path for easier usage
    && ln -sf /usr/lib/nginx /etc/nginx/modules \

    # Create nginx group
    && groupadd -g 8888 nginx \
    && useradd -u 8888 -g nginx nginx \

    # Symlink nginx logs to system output
    && ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log
