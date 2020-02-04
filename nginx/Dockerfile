FROM debian:10.2

ARG NGINX_VERSION=1.16.1

# Install dependencies
RUN apt-get update && apt-get install --assume-yes \
    wget \
    build-essential \
    zlib1g-dev \
    libssl-dev \
    libpcre3-dev

# Download and extract the nginx source code
# https://nginx.org/en/docs/configure.html
RUN wget "https://nginx.org/download/nginx-$NGINX_VERSION.tar.gz" --output-document=- | \
    tar --extract --gzip --file=- --directory /

WORKDIR /nginx-$NGINX_VERSION

# Compile nginx (dynamically linked)
# https://nginx.org/en/docs/configure.html
RUN ./configure \
    --conf-path=/etc/nginx/nginx.conf \
    --sbin-path=/bin/nginx \
    --pid-path=/dev/null \
    --http-client-body-temp-path=/tmp/client_body_temp \
    --http-fastcgi-temp-path=/tmp/fastcgi_temp \
    --http-log-path=/dev/stdout \
    --error-log-path=/dev/stderr \
    --user=nobody \
    --group=nogroup \
    --with-http_ssl_module \
    --with-http_v2_module \
    --with-pcre \
    --without-http_ssi_module \
    --without-http_userid_module \
    --without-http_access_module \
    --without-http_auth_basic_module \
    --without-http_mirror_module \
    --without-http_autoindex_module \
    --without-http_geo_module \
    --without-http_map_module \
    --without-http_split_clients_module \
    --without-http_referer_module \
    --without-http_proxy_module \
    --without-http_uwsgi_module \
    --without-http_scgi_module \
    --without-http_grpc_module \
    --without-http_memcached_module \
    --without-http_limit_conn_module \
    --without-http_limit_req_module \
    --without-http_empty_gif_module \
    --without-http_browser_module \
    --without-http_upstream_hash_module \
    --without-http_upstream_ip_hash_module \
    --without-http_upstream_least_conn_module \
    --without-http_upstream_keepalive_module \
    --without-http_upstream_zone_module
RUN make

# Create necessary folders for the build
RUN mkdir --parents \
    /build/bin \
    /build/etc/nginx \
    /build/tmp/client_body_temp \
    /build/tmp/fastcgi_temp

# Delete all users except "root" and "nobody"
RUN for user in $(awk -F':' '{print $1}' /etc/passwd | egrep --invert-match 'root|nobody'); do \
        userdel $user; \
    done

# Delete all groups except "root" and "nogroup"
RUN for group in $(awk -F':' '{print $1}' /etc/group | egrep --invert-match 'root|nogroup'); do \
        groupdel $group; \
    done

RUN cp --archive /etc/passwd /etc/group /build/etc/

# Copy all dynamically linked libraries for nginx
RUN for library in $(ldd /nginx-$NGINX_VERSION/objs/nginx | grep '=> ' | awk '{print $3}') \
        "/lib64/ld-linux-x86-64.so.2" \
        # Needed to run nginx as different user
        "/lib/x86_64-linux-gnu/libnss_files.so.2" \
        # Needed to resolve the hostname for FastCGI
        "/lib/x86_64-linux-gnu/libresolv.so.2" \
        "/lib/x86_64-linux-gnu/libnss_dns.so.2"; \
    do \
        mkdir --parents "/build$(dirname $library)"; \
        cp --archive $library "/build$(dirname $library)"; \
        \
        if [ -L $library ]; then \
            mkdir --parents "/build$(dirname `realpath $library`)"; \
            cp --archive $(realpath $library) "/build$(dirname `realpath $library`)"; \
        fi \
    done

# Copy nginx binary and configuration files
RUN cp --archive /nginx-$NGINX_VERSION/objs/nginx /build/bin/nginx
COPY resources /build/etc/nginx/

# Set correct permissions, since nginx will run as user nobody:nogroup
RUN chown --recursive nobody:nogroup \
    /build/etc/nginx \
    /build/tmp/client_body_temp \
    /build/tmp/fastcgi_temp

# Everything below this line will be the actual content of the image
FROM scratch

COPY --from=0 /build /

USER nobody:nogroup

WORKDIR /srv/http

EXPOSE 8080 44300

ENTRYPOINT [ "/bin/nginx", "-g", "daemon off;" ]
