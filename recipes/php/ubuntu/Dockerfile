# Copyright (c) 2012-2016 Codenvy, S.A.
# All rights reserved. This program and the accompanying materials
# are made available under the terms of the Eclipse Public License v1.0
# which accompanies this distribution, and is available at
# http://www.eclipse.org/legal/epl-v10.html
# Contributors:
# Codenvy, S.A. - initial API and implementation

FROM eclipse/ubuntu_jre

ENV DEBIAN_FRONTEND noninteractive
ENV CHE_MYSQL_PASSWORD=che
ENV CHE_MYSQL_DB=che_db
ENV CHE_MYSQL_USER=che

# install php with a set of most widely used extensions
RUN sudo apt-get update && \
    sudo apt-get install -y \
    apache2 \
    php5 \
    php5-mhash \
    php5-mcrypt \
    php5-curl \
    php5-mysql \
    php5-gd \
    libapache2-mod-php5 \
    php5-cli \
    php5-json \
    php5-cgi \
    php5-sqlite

RUN sudo sed -i 's/\/var\/www\/html/\/projects/g'  /etc/apache2/sites-available/000-default.conf && \
    sudo sed -i 's/None/All/g' /etc/apache2/sites-available/000-default.conf && \
    echo "ServerName localhost" | sudo tee -a /etc/apache2/apache2.conf && \
    sudo a2enmod rewrite

# Install the Zend Debugger php module
# NOTE: The below commands must be updated when switching the image to 
#       PHP 5.6, PHP 7 or future PHP version
RUN sudo wget http://repos.zend.com/zend-server/8.5.6/deb_apache2.4/pool/zend-server-php-5.5-common_8.5.6+b731_amd64.deb && \
    dpkg-deb --fsys-tarfile zend-server-php-5.5-common_8.5.6+b731_amd64.deb | sudo tar -xf - --strip-components=7 ./usr/local/zend/lib/debugger/php-5.5.x/ZendDebugger.so && \
    sudo rm zend-server-php-5.5-common_8.5.6+b731_amd64.deb && \
    sudo mv ZendDebugger.so /usr/lib/php5/20121212 && \
    sudo sh -c 'echo "; configuration for php ZendDebugger module\n; priority=90\nzend_extension=ZendDebugger.so" > /etc/php5/mods-available/zenddebugger.ini' && \
    sudo ln -s ../../mods-available/zenddebugger.ini /etc/php5/cli/conf.d/90-zenddebugger.ini && \
    sudo ln -s ../../mods-available/zenddebugger.ini /etc/php5/apache2/conf.d/90-zenddebugger.ini && \
    sudo sed -i 's/;opcache.enable=0/opcache.enable=0/g' /etc/php5/apache2/php.ini

RUN curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer && \
    echo -e "MySQL password: $CHE_MYSQL_PASSWORD" >> /home/user/.mysqlrc && \
    echo -e "MySQL user    : $CHE_MYSQL_USER" >> /home/user/.mysqlrc && \
    echo -e "MySQL Database: $CHE_MYSQL_DB" >> /home/user/.mysqlrc && \
    sudo -E bash -c "apt-get -y --no-install-recommends install mysql-server" && \
    sudo apt-get clean && \
    sudo apt-get -y autoremove && \
    sudo apt-get -y clean && \
    sudo rm -rf /var/lib/apt/lists/* && \
    sudo sed -i.bak 's/127.0.0.1/0.0.0.0/g' /etc/mysql/my.cnf && \
    sudo service mysql restart && \
    sudo mysql -uroot -e "GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'; FLUSH PRIVILEGES;" && \
    sudo service mysql restart && \
    sudo service mysql restart && sudo mysql -uroot -e "CREATE USER '$CHE_MYSQL_USER'@'%' IDENTIFIED BY '"$CHE_MYSQL_PASSWORD"'" && \
    sudo mysql -uroot -e "GRANT ALL PRIVILEGES ON *.* TO '$CHE_MYSQL_USER'@'%' IDENTIFIED BY '"$CHE_MYSQL_PASSWORD"'; FLUSH PRIVILEGES;" && \
    sudo mysql -uroot -e "CREATE DATABASE $CHE_MYSQL_DB;"

# label is used in Servers tab to display mapped port for Apache process on 80 port in the container
LABEL che:server:80:ref=apache2 che:server:80:protocol=http

EXPOSE 80 3306
WORKDIR /projects

CMD sudo chown -R www-data:www-data /projects && \
    sudo chmod -R 777 /projects && \
    tail -f /dev/null
