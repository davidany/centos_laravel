FROM centos:centos7

RUN yum -y install deltarpm
RUN yum -y update && \
    yum -y install \
    epel-release \
    httpd \
    httpd-tools \
    beanstalkd \
    unzip ksh zip unzip \
    yum-utils nano which iproute xclip

RUN  yum -y update

RUN yum install -y wget
RUN yum install -y epel-release

RUN yum install -y https://rpms.remirepo.net/enterprise/remi-release-7.rpm

RUN yum install -y --enablerepo=remi-php80 php php-mcrypt php-cli php-gd php-curl php-mysql php-ldap php-zip php-fileinfo  php-xdebug  php-pear php-devel php-sockets php80-php-oci8

#composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer


#sql server
RUN yum install -y --enablerepo=remi-php80  unixODBC-devel
RUN pecl install sqlsrv pdo_sqlsrv

RUN curl https://packages.microsoft.com/config/rhel/7/prod.repo > /etc/yum.repos.d/mssql-release.repo
RUN ACCEPT_EULA=Y yum install -y msodbcsql17
RUN ACCEPT_EULA=Y yum install -y mssql-tools
RUN echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
RUN source ~/.bashrc

RUN echo extension=pdo_sqlsrv.so >> `php --ini | grep "Scan for additional .ini files" | sed -e "s|.*:\s*||"`/30-pdo_sqlsrv.ini
RUN echo extension=sqlsrv.so >> `php --ini | grep "Scan for additional .ini files" | sed -e "s|.*:\s*||"`/20-sqlsrv.ini

#db2
RUN mkdir /opt/ibm
COPY docker/linuxx64_odbc_cli.tar.gz /opt/ibm/
COPY docker/ibm_data_server_driver_package_linuxx64_v11.5.tar.gz /opt/ibm/
WORKDIR "/opt/ibm"
RUN tar xzf ibm_data_server_driver_package_linuxx64_v11.5.tar.gz -C /opt/ibm
WORKDIR "/opt/ibm/dsdriver"

RUN chmod 755 installDSDriver
RUN ksh installDSDriver
RUN export IBM_DB_HOME=/opt/ibm/dsdriver && \
    env IBM_DB_HOME=/opt/ibm/dsdriver && \
    pecl install ibm_db2 && \
    echo "extension=ibm_db2.so" >> /etc/php.d/ibm_db2.ini

RUN #touch /var/log/xdebug.log


RUN touch /var/log/xdebug.log
RUN /bin/bash -c "mkdir -p /var/log/php"
RUN touch /var/log/php/mono.log
RUN chmod -R +rwX  /var/log/
RUN chown -R apache:apache /var/log/
RUN chown -R apache:apache /var/www/html


RUN echo ' xdebug.mode=develop,coverage,debug '  >> /etc/php.ini
RUN echo ' xdebug.discover_client_host=true '  >> /etc/php.ini
RUN echo ' xdebug.log=/var/log/xdebug.log '  >> /etc/php.ini
RUN echo ' xdebug.client_host=host.docker.internal '  >> /etc/php.ini
RUN echo ' xdebug.client_port=9003 '  >> /etc/php.ini
RUN echo ' xdebug.start_with_request=yes '  >> /etc/php.ini
RUN echo ' xdebug.var_display_max_data=512 '  >> /etc/php.ini
RUN echo ' xdebug.var_display_max_depth=3 '  >> /etc/php.ini
RUN echo ' xdebug.var_display_max_children=128 '  >> /etc/php.ini
RUN echo ' xdebug.cli_color=1 '  >> /etc/php.ini
RUN echo ' xdebug.show_local_vars=0 '  >> /etc/php.ini
RUN echo ' xdebug.dump_globals=true '  >> /etc/php.ini
RUN echo ' xdebug.dump_once=true '  >> /etc/php.ini
RUN echo ' xdebug.dump_undefined=false; '  >> /etc/php.ini
RUN echo ' xdebug.dump.SERVER=REMOTE_ADDR,REQUEST_METHOD '  >> /etc/php.ini
RUN echo ' xdebug.dump.GET=*  '  >> /etc/php.ini
RUN echo ' xdebug.dump.POST=*  '  >> /etc/php.ini
RUN echo ' xdebug.max_stack_frames=-1  '  >> /etc/php.ini
RUN echo ' xdebug.show_exception_trace=0 '  >> /etc/php.ini

# Update Apache Configuration
ENV HTTPD_MAIN_CONF_PATH=/etc/httpd/conf

RUN sed -E -i -e '/<Directory "\/var\/www\/html">/,/<\/Directory>/s/AllowOverride None/AllowOverride All/' ${HTTPD_MAIN_CONF_PATH}/httpd.conf
RUN sed -E -i -e 's/DocumentRoot "\/var\/www\/html"/DocumentRoot "\/var\/www\/html\/public"/' ${HTTPD_MAIN_CONF_PATH}/httpd.conf
RUN sed -E -i -e 's/DirectoryIndex (.*)$/DirectoryIndex index.php \1/g' ${HTTPD_MAIN_CONF_PATH}/httpd.conf
#RUN  sed -i "s/^User apache/User apache/" ${HTTPD_MAIN_CONF_PATH}/httpd.conf
#RUN  sed -i "s/^Group apache/Group root/" ${HTTPD_MAIN_CONF_PATH}/httpd.conf

ENV HTTPD_DATA_ORIG_PATH=/var/www
ENV HTTPD_MAIN_CONF_D_PATH=/etc/httpd/conf.d
ENV APP_DATA=${APP_ROOT}/src
ENV HTTPD_VAR_PATH=/var






#RUN yum install -y nodejs zsh redis

# ORACLE oci
RUN mkdir /opt/oracle \
    && cd /opt/oracle

ADD https://download.oracle.com/otn_software/linux/instantclient/199000/instantclient-basic-linux.x64-19.9.0.0.0dbru.zip /opt/oracle
ADD https://download.oracle.com/otn_software/linux/instantclient/199000/instantclient-sdk-linux.x64-19.9.0.0.0dbru.zip /opt/oracle

# Install Oracle Instantclient
RUN  unzip /opt/oracle/instantclient-basic-linux.x64-19.9.0.0.0dbru.zip -d /opt/oracle \
    && unzip /opt/oracle/instantclient-sdk-linux.x64-19.9.0.0.0dbru.zip -d /opt/oracle \
    && ln -s /opt/oracle/instantclient_19_9/libclntshcore.so.19.9 /opt/oracle/instantclient_19_9/libclntshcore.so \
    && rm -rf /opt/oracle/*.zip

ENV LD_LIBRARY_PATH  /opt/oracle/instantclient_19_9:${LD_LIBRARY_PATH}

RUN yum install -y yum install systemtap-sdt-devel libaio \
	&& export PHP_DTRACE=yes \
	&& wget https://www.php.net/distributions/php-8.0.14.tar.gz \
	&& tar xvzf php-8.0.14.tar.gz \
	&& cd php-8.0.14/ext/pdo_oci \
	&& phpize \
	&& ./configure --with-pdo-oci=instantclient,/opt/oracle/instantclient_19_9/ \
	&& make && make install
#&& echo "extension=pdo_oci.so" > /etc/php.d/pdo_oci.ini

RUN  pear download pecl/oci8-3.0.1 \
	&& cd /opt/ibm/dsdriver/ \
	&& tar xvzf oci8-3.0.1.tgz \
	&& cd oci8-3.0.1 \
	&& export PHP_DTRACE=yes \
	&& phpize \
	&& ./configure --with-oci8=instantclient,/opt/oracle/instantclient_19_9/ \
	&& make && make install

COPY ./docker/php/30-pdo_oci.ini /etc/php.d/30-pdo_oci.ini
COPY ./docker/php/20-oci8.ini /etc/php.d/20-oci8.ini

RUN echo 'alias vi="vim"' >> /root/.bash_profile
RUN echo 'alias ll="ls -la"' >> /root/.bash_profile
RUN echo 'alias cdp="cd /var/www/html"' >> /root/.bash_profile
RUN echo 'alias pbcopy="xclip -selection clipboard"' >> /root/.bash_profile
RUN echo 'alias pbpaste="xclip -selection clipboard -o"' >> /root/.bash_profile
RUN source /root/.bash_profile




#RUN useradd -r  -g apache apache
#USER apache
##WORKDIR /var/www/html
#
#USER 0
#RUN chown -R 1001:0 /var/www/html
#USER 1001
#
## Set permissions for Apache user and group
##	&& chown -R www-data:www-data /var/www/src
#
## Set timezone
#RUN #ln -sf /usr/share/zoneinfo/Americas/New_York /etc/localtime


ENV SLIM_MODE=testing
ENV CI_ENV=testing
ENV SOME_KEY=someValue
#ENV PHP_INI_SCAN_DIR=/usr/local/etc/php/conf.d
EXPOSE 80
#COPY . /var/www/html

# Change uid and gid of apache to docker user uid/gid
#RUN usermod -u 1000 www-data \
#    && groupmod -g 1000 www-data

# RUN chown -R apache:apache /var/www/html
#RUN usermod -u 1000 www-data \
#    && groupmod -g 1000 www-data

RUN chown -R apache:apache /var/www/html
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
