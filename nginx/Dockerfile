# syntax=docker/dockerfile:1.18.0

# nginx builder
FROM alpine:3.22.2 AS builder

RUN apk add --no-cache \
    perl \
    build-base \
    pcre-dev \
    zlib-dev \
    openssl openssl-dev \
    linux-headers

WORKDIR /tmp/build

ARG NGINX_VERSION
ENV NGINX_VERSION=${NGINX_VERSION:-1.29.2}
RUN wget -q "https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz" && \
    tar -xzf "nginx-${NGINX_VERSION}.tar.gz"

RUN cd "nginx-${NGINX_VERSION}" && \
    ./configure \
    --prefix=/usr/local/nginx \
    --sbin-path=/usr/local/nginx/sbin/nginx \
    --modules-path=/usr/local/nginx/modules \
    --conf-path=/etc/nginx/conf/nginx.conf \
    --error-log-path=/var/log/nginx/error.log \
    --http-log-path=/var/log/nginx/access.log \
    --pid-path=/var/run/nginx/nginx.pid \
    --lock-path=/var/run/nginx/nginx.lock \
    --http-client-body-temp-path=/var/cache/nginx/client_temp \
    --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
    --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
    --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
    --http-scgi-temp-path=/var/cache/nginx/scgi_temp \
    --user=nginx \
    --group=nginx \
    --with-compat \
    --with-threads \
    --with-file-aio \
    --with-http_ssl_module \
    --with-http_v2_module \
    --with-http_v3_module \
    --with-http_realip_module \
    --with-http_stub_status_module \
    --with-http_gzip_static_module \
    --with-http_gunzip_module \
    --with-http_sub_module \
    --with-http_secure_link_module \
    --with-http_auth_request_module \
    --with-http_slice_module \
    --with-stream \
    --with-stream_ssl_module \
    --with-stream_ssl_preread_module \
    --with-pcre-jit && \
    make -j"$(nproc)" && \
    make install

# nginx
FROM alpine:3.22.2

RUN apk add --no-cache pcre curl ca-certificates logrotate

COPY --from=builder /usr/local/nginx /usr/local/nginx
ENV PATH="/usr/local/nginx/sbin:${PATH}"

COPY --from=builder /etc/nginx/conf /etc/nginx/conf
COPY --from=builder /usr/local/nginx/html /usr/local/nginx/html

RUN addgroup -S nginx && \
    adduser -S -D -H -s /sbin/nologin -G nginx nginx && \
    mkdir -p \
        /var/log/nginx \
        /var/cache/nginx \
        /var/run/nginx && \
    chown -R nginx:nginx /var/log/nginx /var/cache/nginx /var/run/nginx

VOLUME ["/etc/nginx/conf", "/var/log/nginx", "/var/cache/nginx", "/usr/local/nginx/html"]

EXPOSE 80 443 443/udp

HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
  CMD nginx -t || exit 1

USER nginx:nginx

CMD ["nginx", "-g", "daemon off;"]