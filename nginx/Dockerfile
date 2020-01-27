FROM debian:stable

# Install dependencies
RUN apt-get update && apt-get install --assume-yes \
    wget \
    build-essential \
    zlib1g-dev \
    libssl-dev \
    libpcre3-dev

# Download and extract the nginx source code
RUN wget "https://nginx.org/download/nginx-1.16.1.tar.gz" --output-document=- | \
    tar --extract --gzip --file=- --directory /

WORKDIR /nginx-1.16.1

# Compile and install nginx (dynamically linked)
RUN ./configure \
    --http-log-path=/dev/stdout \
    --error-log-path=/dev/stderr \
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
    --without-http_rewrite_module \
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
RUN make install

# Create necessary folders for the build
RUN mkdir --parents \
    /build/etc \
    /build/lib64 \
    /build/lib/x86_64-linux-gnu \
    /build/usr/local \
    /build/usr/lib/x86_64-linux-gnu

# Delete all users except "root" and "nobody"
RUN for user in $(awk -F':' '{print $1}' /etc/passwd | egrep -v 'root|nobody'); do \
        userdel $user; \
    done

# Delete all groups except "root" and "nogroup"
RUN for group in $(awk -F':' '{print $1}' /etc/group | egrep -v 'root|nogroup'); do \
        groupdel $group; \
    done

# If we don't include these, we get an error:
# nginx: [emerg] getpwnam("nobody") failed (2: No such file or directory)
RUN cp --archive \
        /etc/passwd \
        /etc/group \
        /build/etc/ && \
    cp --archive \
        /lib/x86_64-linux-gnu/libnss_files.so.2 \
        /lib/x86_64-linux-gnu/libnss_files-2.28.so \
        /build/lib/x86_64-linux-gnu/

RUN cp --archive \
        /lib64/ld-linux-x86-64.so.2 \
        /build/lib64/ && \
    cp --archive \
        /lib/x86_64-linux-gnu/ld-linux-x86-64.so.2 \
        /lib/x86_64-linux-gnu/ld-2.28.so \
        /build/lib/x86_64-linux-gnu/

# Copy all dynamically linked libraries for nginx
RUN for library in $(ldd /usr/local/nginx/sbin/nginx | grep '=> ' | awk '{print $3}'); do \
        mkdir --parents "/build$(dirname $library)"; \
        cp --archive $library "/build$(dirname $library)"; \
        \
        if [ -L $library ]; then \
            mkdir --parents "/build$(dirname `realpath $library`)"; \
            cp --archive $(realpath $library) "/build$(dirname `realpath $library`)"; \
        fi \
    done

# Copy nginx binary and configuration files
RUN cp --archive /usr/local/nginx /build/usr/local/
COPY nginx.conf /build/usr/local/nginx/conf/nginx.conf

FROM scratch

COPY --from=0 /build /

WORKDIR /usr/local/nginx/html

EXPOSE 80

CMD [ "/usr/local/nginx/sbin/nginx", "-g", "daemon off;"]
