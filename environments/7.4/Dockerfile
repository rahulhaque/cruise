FROM ubuntu:22.04

LABEL maintainer="Rahul Haque"

ARG WWWGROUP
ARG WWWUSER
ARG PHP_VERSION=7.4
ARG NODE_VERSION=16

WORKDIR /var/www/html

ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=UTC
ENV NPM_CONFIG_PREFIX=/home/cruise/.npm-global
ENV PATH=$PATH:/home/cruise/.npm-global/bin

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Install PHP
RUN apt-get update \
    && apt-get install -y gnupg gosu curl nano fontconfig nginx ca-certificates \
       zip unzip supervisor sqlite3 libcap2-bin libpng-dev python2 apt-utils \
    && mkdir -p ~/.gnupg \
    && chmod 600 ~/.gnupg \
    && echo "disable-ipv6" >> ~/.gnupg/dirmngr.conf \
    && echo "keyserver hkp://keyserver.ubuntu.com:80" >> ~/.gnupg/dirmngr.conf \
    && gpg --recv-key 0x14aa40ec0831756756d7f66c4f4ea0aae5267a6c \
    && gpg --export 0x14aa40ec0831756756d7f66c4f4ea0aae5267a6c > /usr/share/keyrings/ppa_ondrej_php.gpg \
    && echo "deb [signed-by=/usr/share/keyrings/ppa_ondrej_php.gpg] https://ppa.launchpadcontent.net/ondrej/php/ubuntu jammy main" > /etc/apt/sources.list.d/ppa_ondrej_php.list \
    && apt-get update \
    && apt-get install -y php$PHP_VERSION-fpm php$PHP_VERSION-cli php$PHP_VERSION-dev \
       php$PHP_VERSION-pgsql php$PHP_VERSION-sqlite3 php$PHP_VERSION-gd \
       php$PHP_VERSION-curl \
       php$PHP_VERSION-imap php$PHP_VERSION-mysql php$PHP_VERSION-mbstring \
       php$PHP_VERSION-xml php$PHP_VERSION-zip php$PHP_VERSION-bcmath php$PHP_VERSION-soap \
       php$PHP_VERSION-intl php$PHP_VERSION-readline \
       php$PHP_VERSION-ldap \
       php$PHP_VERSION-msgpack php$PHP_VERSION-igbinary php$PHP_VERSION-redis \
       php$PHP_VERSION-pcov php$PHP_VERSION-xdebug \
    && php -r "readfile('https://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer

# Install Node
RUN curl -L https://deb.nodesource.com/setup_$NODE_VERSION.x | bash - \
    && apt-get install -y nodejs \
    && curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | tee /usr/share/keyrings/yarn.gpg >/dev/null \
    && echo "deb [signed-by=/usr/share/keyrings/yarn.gpg] https://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list \
    && apt-get update \
    && apt-get install -y yarn \
    && mkdir -p ~/.npm-global/lib \
    && mkdir -p ~/.npm-global/bin

# Install Bangla and MS Fonts
RUN mkdir -p /usr/share/fonts \
    && curl -L https://www.omicronlab.com/download/fonts/Bangla.ttf -o /usr/share/fonts/Bangla.ttf \
    && echo "ttf-mscorefonts-installer msttcorefonts/accepted-mscorefonts-eula select true" | debconf-set-selections \
    && apt-get install -y ttf-mscorefonts-installer

# Install Wkhtmltopdf
RUN apt-get install -y libxext6 libxrender1 xfonts-base xfonts-75dpi \
    && curl -L https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-2/wkhtmltox_0.12.6.1-2.jammy_amd64.deb -o /wkhtmltopdf.deb \
    && dpkg -i /wkhtmltopdf.deb \
    && rm /wkhtmltopdf.deb \
    && fc-cache -fv

# Install Oh-my-zsh
RUN apt-get install -y git zsh \
    && export HOME=/home/cruise \
    && sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" --unattended \
    && git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions \
    && zsh -i -c 'omz theme set steeef' \
    && zsh -i -c 'omz plugin enable laravel composer zsh-autosuggestions' \
    && sed -i "/zstyle ':omz:update' mode disabled/s/^#\s*//" $HOME/.zshrc \
    && sed -i "3 i export PATH=./vendor/bin:\$PATH" $HOME/.zshrc \
    && sed -i "4 i export PATH=\$HOME/.config/composer/vendor/bin:\$PATH" $HOME/.zshrc

# Clean any temporary files
RUN apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Copy all required config files
COPY $PHP_VERSION/start-server /usr/local/bin/start-server
COPY $PHP_VERSION/node-switch /usr/local/bin/node-switch
COPY $PHP_VERSION/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY $PHP_VERSION/pool.conf /etc/php/$PHP_VERSION/fpm/pool.d/z-www-custom.conf
COPY $PHP_VERSION/php.ini /etc/php/$PHP_VERSION/fpm/conf.d/99-php.ini
COPY $PHP_VERSION/nginx.conf /etc/nginx/conf.d/default.conf

# Update runtime permissions
RUN setcap "cap_net_bind_service=+ep" /usr/bin/php$PHP_VERSION \
    && groupadd --force -g $WWWGROUP cruise \
    && useradd -s /usr/bin/zsh --no-create-home --no-user-group -g $WWWGROUP -u $WWWUSER cruise \
    && chown -R $WWWUSER:$WWWGROUP /home/cruise \
    && rm /etc/nginx/sites-enabled/default \
    && sed -i 's/www-data;/cruise;/g' /etc/nginx/nginx.conf \
    && chmod +x /usr/local/bin/start-server \
    && chmod +x /usr/local/bin/node-switch

EXPOSE 80

CMD ["zsh"]
