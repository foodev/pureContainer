FROM debian:stable

# Install dependencies
RUN apt-get update && apt-get install --assume-yes \
    wget \
    xz-utils \
    build-essential \
    pkg-config

# Download and extract the PHP source code
RUN wget "https://www.php.net/distributions/php-7.4.2.tar.xz" --output-document=- | \
    tar --extract --xz --file=- --directory /

WORKDIR /php-7.4.2

# Compile and install PHP
RUN ./configure \
    --prefix=/usr/local/php \
    --disable-all \
    --disable-cgi \
    --disable-cli \
    --disable-phpdbg \
    --enable-fpm \
    --with-fpm-user=nobody \
    --with-fpm-group=nogroup
RUN make
RUN make install

# Create necessary folders for the build
RUN mkdir --parents \
    /build/etc \
    /build/lib64 \
    /build/lib/x86_64-linux-gnu \
    /build/usr/local \
    /build/tmp

# Delete all users except "root" and "nobody"
RUN for user in $(awk -F':' '{print $1}' /etc/passwd | egrep -v 'root|nobody'); do \
        userdel $user; \
    done

# Delete all groups except "root" and "nogroup"
RUN for group in $(awk -F':' '{print $1}' /etc/group | egrep -v 'root|nogroup'); do \
        groupdel $group; \
    done

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

# Copy all dynamically linked libraries for PHP-FPM
RUN for library in $(ldd /usr/local/php/sbin/php-fpm | grep '=> ' | awk '{print $3}'); do \
        mkdir --parents "/build$(dirname $library)"; \
        cp --archive $library "/build$(dirname $library)"; \
        \
        if [ -L $library ]; then \
            mkdir --parents "/build$(dirname `realpath $library`)"; \
            cp --archive $(realpath $library) "/build$(dirname `realpath $library`)"; \
        fi \
    done

# Copy PHP binary and configuration files
RUN cp --archive /usr/local/php /build/usr/local/ && \
    cp --archive /php-7.4.2/php.ini-development /build/usr/local/php/php.ini && \
    mv /build/usr/local/php/etc/php-fpm.conf.default /build/usr/local/php/etc/php-fpm.conf
COPY www.conf /build/usr/local/php/etc/php-fpm.d/www.conf
COPY info.php /build/scripts/info.php

FROM scratch

COPY --from=0 /build /

EXPOSE 9000

CMD [ "/usr/local/php/sbin/php-fpm", "--nodaemonize" ]
