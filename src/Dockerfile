FROM arm32v7/debian
MAINTAINER romeOz <serggalka@gmail.com>

ENV PHP_VERSION=5.3 \
	OS_LOCALE="en_US.UTF-8"
RUN apt-get update
RUN apt-get install -y locales
RUN locale-gen ${OS_LOCALE}
ENV LANG=${OS_LOCALE} \
    LANGUAGE=${OS_LOCALE} \
    LC_ALL=${OS_LOCALE}

COPY qemu-arm-static /usr/bin

RUN apt-get install -y build-essential zip autoconf 
RUN apt-get install -y libfcgi-dev libfcgi0ldbl libmcrypt-dev libssl-dev libc-client2007e-dev libkrb5-dev libcurl4-openssl-dev wget
WORKDIR /opt
RUN wget https://www.openssl.org/source/old/1.0.1/openssl-1.0.1u.tar.gz
RUN tar xzf openssl-1.0.1u.tar.gz
WORKDIR /opt/openssl-1.0.1u
RUN ./config shared -std=gnu11 --openssldir=/usr/local/openssl/
RUN make depend
RUN make
RUN make install
RUN ln -s /usr/local/openssl/lib /usr/local/openssl/lib/arm-linux-gnueabihf
WORKDIR /tmp
RUN wget https://mirrors.mediatemple.net/debian-archive/debian/pool/main/u/uw-imap/uw-imap_2007f\~dfsg-2.dsc
RUN wget https://mirrors.mediatemple.net/debian-archive/debian/pool/main/u/uw-imap/uw-imap_2007f~dfsg.orig.tar.gz
RUN wget https://mirrors.mediatemple.net/debian-archive/debian/pool/main/u/uw-imap/uw-imap_2007f~dfsg-2.debian.tar.gz
RUN dpkg-source -x uw-imap_2007f\~dfsg-2.dsc imap-2007f
RUN mv imap-2007f /usr/local/
WORKDIR /usr/local/imap-2007f/
RUN touch {ipv6,lnxok}
RUN make slx SSLINCLUDE=/usr/local/openssl/include/ SSLLIB=/usr/local/openssl/lib EXTRAAUTHENTICATORS=gss ; exit 0
RUN mkdir lib include
RUN cp c-client/*.c lib/
RUN cp c-client/*.h include/
RUN cp c-client/c-client.a lib/libc-client.a
RUN ls  /usr/lib/
RUN ln -s /usr/lib/libc-client.a /usr/lib/arm-linux-gnueabihf/libc-client.a

WORKDIR /opt
RUN wget https://curl.haxx.se/download/curl-7.26.0.tar.gz
RUN tar xzf curl-7.26.0.tar.gz
WORKDIR /opt/curl-7.26.0
RUN env PKG_CONFIG_PATH=/usr/local/openssl/lib/pkgconfig LDFLAGS=-Wl,-rpath=/usr/local/openssl/lib ./configure --with-ssl=/usr/local/openssl --with-zlib --prefix=/usr/local/curl
RUN make
RUN make install

WORKDIR /opt
RUN echo "deb http://http.us.debian.org/debian sid main non-free contrib" >> /etc/apt/sources.list
RUN apt-get update
RUN apt-get install -y libxml2-dev zlib1g-dev libbz2-dev libjpeg-dev libpng-dev libfreetype6-dev libmysqlclient-dev apache2 apache2-dev
# default-libmysqlclient-dev
 
ENV PHP_VER=5.3.29
RUN wget http://museum.php.net/php5/php-${PHP_VER}.tar.gz
RUN tar xzf php-${PHP_VER}.tar.gz
RUN ls /usr/local/curl
WORKDIR /opt/php-${PHP_VER}
RUN LDFLAGS="-Wl,-rpath=/usr/local/openssl/lib,-rpath=/usr/local/curl/lib" './configure'  --prefix=/usr/local/php-${PHP_VER} '--with-zlib-dir' '--with-freetype-dir' '--enable-mbstring' '--with-libxml-dir=/usr' '--enable-soap' '--enable-calendar' '--with-curl=/usr/local/curl' '--with-mcrypt' '--with-zlib' '--with-gd' '--disable-rpath' '--enable-inline-optimization' '--with-bz2' '--with-zlib' '--enable-sockets' '--enable-sysvsem' '--enable-sysvshm' '--enable-mbregex' '--with-mhash' '--enable-zip' '--with-pcre-regex' '--with-mysql' '--with-pdo-mysql' '--with-jpeg-dir=/usr' '--with-png-dir=/usr' '--enable-gd-native-ttf' '--with-pear' '--enable-memcache' '--with-openssl=/usr/local/openssl' '--with-kerberos' '--with-imap-ssl' '--with-libdir=lib/arm-linux-gnueabihf' '--enable-so' '--with-apxs2=/usr/bin/apxs2'
RUN LDFLAGS="-Wl,-rpath=/usr/local/openssl/lib,-rpath=/usr/local/curl/lib" make
RUN make install

RUN cp -p .libs/libphp5.so /usr/lib/apache2/modules/
RUN mkdir /etc/apache2/php/
RUN cp -p php.ini-production /etc/apache2/php/php.ini




WORKDIR /var/www/app/

EXPOSE 80 443
COPY entrypoint.sh /sbin/entrypoint.sh
RUN echo "LoadModule php5_module	modules/libphp5.so" >>  /etc/apache2/apache2.conf
RUN echo "AddHandler php5-script	.php" >> /etc/apache2/apache2.conf

RUN  sed -i 's@${APACHE_RUN_DIR}@/var/run/apache2@g' /etc/apache2/apache2.conf
RUN  mkdir -p /var/run/apache2

RUN a2dismod mpm_event
RUN a2enmod mpm_prefork

RUN rm /usr/bin/qemu-arm-static

# By default, simply start apache.
CMD ["/sbin/entrypoint.sh"]

