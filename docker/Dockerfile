FROM php:7.2-fpm

ARG TIMEZONE
ARG USER
ARG USER_NAME
ARG USER_EMAIL
ARG USER_ID
ARG XDEBUG_PORT

# Install APT basic packages
RUN apt update && apt install -y git-all git-flow unzip vim curl gnupg bash-completion

# Install APT repositories
RUN curl -sL https://deb.nodesource.com/setup_6.x | bash -
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list

# Install APT packages nodejs & yarn
RUN apt-get update && apt-get install -y nodejs yarn

# Set timezone
RUN ln -snf /usr/share/zoneinfo/${TIMEZONE} /etc/localtime && echo ${TIMEZONE} > /etc/timezone
RUN printf '[PHP]\ndate.timezone = "%s"\n', ${TIMEZONE} > /usr/local/etc/php/conf.d/tzone.ini
RUN "date"

# Type docker-php-ext-install to see available extensions
RUN docker-php-ext-install pdo_mysql bcmath
RUN pecl install apcu-5.1.8
RUN docker-php-ext-enable apcu

# install xdebug
RUN pecl install xdebug
RUN docker-php-ext-enable xdebug
RUN echo "error_reporting = E_ALL" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "display_startup_errors = On" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "display_errors = On" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_autostart=1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_enable=1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_connect_back=0" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.idekey=\"PHPSTORM\"" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_port=${XDEBUG_PORT}" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php && \
    mv composer.phar /usr/local/bin/composer

# Install Deptrac (https://github.com/sensiolabs-de/deptrac)
RUN curl -LS https://github.com/sensiolabs-de/deptrac/releases/download/0.5.0/deptrac.phar -o deptrac.phar && \
    chmod +x deptrac.phar && \
    mv deptrac.phar /usr/local/bin/deptrac

# Sonar Scanner
RUN npm install -g sonarqube-scanner

# Create user
RUN groupadd ${USER}
RUN useradd -g ${USER} -u ${USER_ID} -d /home/${USER} -m -s /bin/bash ${USER}
COPY docker/bash/ssh/id_rsa* /home/${USER}/.ssh/
COPY docker/bash/.gitconfig /home/${USER}/
COPY docker/bash/.bashrc /home/${USER}/.bashrc
COPY docker/bash/.bash_aliases /home/${USER}/.bash_aliases

RUN \
    chown -R ${USER}:${USER} /home/${USER} && \
    sed -i "s/EMAIL/${USER_EMAIL}/g" /home/${USER}/.gitconfig && \
    sed -i "s/NAME/${USER_NAME}/g"  /home/${USER}/.gitconfig && \
    chmod 600 /home/${USER}/.ssh/id_rsa && \
    chmod 600 /home/${USER}/.ssh/id_rsa.pub && \
    /bin/bash -c "eval '$(ssh-agent -s)'; ssh-add /home/${USER}/.ssh/id_rsa" && \
    mkdir /var/www/app && \
    chown -R ${USER}:${USER} /var/www/app && \
    mkdir -p /var/log/nginx/ && \
    chmod -R 0777 /var/log/nginx

WORKDIR /var/www/app
USER ${USER}
