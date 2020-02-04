FROM debian:10.2

ARG PHP_VERSION=7.4.2

# Install dependencies
RUN apt-get update && apt-get install --assume-yes \
    wget \
    xz-utils \
    build-essential \
    pkg-config

# Download and extract the PHP source code
RUN wget "https://www.php.net/distributions/php-$PHP_VERSION.tar.xz" --output-document=- | \
    tar --extract --xz --file=- --directory /

WORKDIR /php-$PHP_VERSION

# Compile PHP (dynamically linked)
RUN ./configure \
    --disable-all \
    --disable-cgi \
    --disable-cli \
    --disable-phpdbg \
    --enable-fpm \
    --with-fpm-user=nobody \
    --with-fpm-group=nogroup
RUN make

# Create necessary folders for the build
RUN mkdir --parents \
    /build/bin \
    /build/etc/php

# Delete all users except "root" and "nobody"
RUN for user in $(awk -F':' '{print $1}' /etc/passwd | egrep --invert-match 'root|nobody'); do \
        userdel $user; \
    done

# Delete all groups except "root" and "nogroup"
RUN for group in $(awk -F':' '{print $1}' /etc/group | egrep --invert-match 'root|nogroup'); do \
        groupdel $group; \
    done

RUN cp --archive /etc/passwd /etc/group /build/etc/

# Copy all dynamically linked libraries for PHP-FPM
RUN for library in $(ldd /php-$PHP_VERSION/sapi/fpm/php-fpm | grep '=> ' | awk '{print $3}') \
        "/lib64/ld-linux-x86-64.so.2" \
        # Needed to run PHP-FPM as different user
        "/lib/x86_64-linux-gnu/libnss_files.so.2"; \
    do \
        mkdir --parents "/build$(dirname $library)"; \
        cp --archive $library "/build$(dirname $library)"; \
        \
        if [ -L $library ]; then \
            mkdir --parents "/build$(dirname `realpath $library`)"; \
            cp --archive $(realpath $library) "/build$(dirname `realpath $library`)"; \
        fi \
    done

# Copy PHP binary and configuration files
RUN cp --archive /php-$PHP_VERSION/sapi/fpm/php-fpm /build/bin/php-fpm
COPY resources /build/etc/php/

# Copy the actual PHP source code
COPY --chown=nobody:nogroup . /build/scripts/

# Everything below this line will be the actual content of the image
FROM scratch

COPY --from=0 /build /

USER nobody:nogroup

EXPOSE 9000

ENTRYPOINT [ "/bin/php-fpm", "--nodaemonize", "-c", "/etc/php/php.ini", "--fpm-config", "/etc/php/php-fpm.conf" ]
