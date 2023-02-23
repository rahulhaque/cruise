FROM ubuntu:22.04

LABEL maintainer="Rahul Haque"

ARG WWWGROUP
ARG WWWUSER
ARG NODE_VERSION=16

WORKDIR /var/www/html

ENV DEBIAN_FRONTEND noninteractive
ENV TZ=UTC

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

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
    && apt-get install -y php7.4-fpm php7.4-cli php7.4-dev \
       php7.4-pgsql php7.4-sqlite3 php7.4-gd \
       php7.4-curl \
       php7.4-imap php7.4-mysql php7.4-mbstring \
       php7.4-xml php7.4-zip php7.4-bcmath php7.4-soap \
       php7.4-intl php7.4-readline \
       php7.4-ldap \
       php7.4-msgpack php7.4-igbinary php7.4-redis \
       php7.4-pcov php7.4-xdebug \
    && php -r "readfile('https://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer

RUN curl -L https://deb.nodesource.com/setup_$NODE_VERSION.x | bash - \
    && apt-get install -y nodejs \
    && npm install -g npm \
    && curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | tee /usr/share/keyrings/yarn.gpg >/dev/null \
    && echo "deb [signed-by=/usr/share/keyrings/yarn.gpg] https://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list \
    && apt-get update \
    && apt-get install -y yarn

RUN mkdir -p /usr/share/fonts \
    && curl -L https://www.omicronlab.com/download/fonts/Bangla.ttf -o /usr/share/fonts/Bangla.ttf \
    && echo "ttf-mscorefonts-installer msttcorefonts/accepted-mscorefonts-eula select true" | debconf-set-selections \
    && apt-get install -y ttf-mscorefonts-installer

RUN apt-get install -y libxext6 libxrender1 xfonts-base xfonts-75dpi \
    && curl -L https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-2/wkhtmltox_0.12.6.1-2.jammy_amd64.deb -o /wkhtmltopdf.deb \
    && dpkg -i /wkhtmltopdf.deb \
    && rm /wkhtmltopdf.deb \
    && fc-cache -fv

RUN apt-get install -y git zsh \
    && export HOME=/home/cruise \
    && sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" --unattended \
    && git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions \
    && zsh -i -c 'omz theme set steeef' \
    && zsh -i -c 'omz plugin enable laravel composer zsh-autosuggestions' \
    && sed -i "/zstyle ':omz:update' mode disabled/s/^#\s*//" $HOME/.zshrc \
    && sed -i "3 i export PATH=./vendor/bin:\$PATH" $HOME/.zshrc \
    && sed -i "4 i export PATH=\$HOME/.config/composer/vendor/bin:\$PATH" $HOME/.zshrc

RUN apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN setcap "cap_net_bind_service=+ep" /usr/bin/php7.4

RUN groupadd --force -g $WWWGROUP cruise \
    && useradd -s /bin/bash --no-create-home --no-user-group -g $WWWGROUP -u $WWWUSER cruise \
    && chown -R $WWWUSER:$WWWGROUP /home/cruise

COPY start-server /usr/local/bin/start-server
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY php.ini /etc/php/7.4/fpm/conf.d/99-php.ini
COPY nginx.conf /etc/nginx/conf.d/default.conf
RUN rm /etc/nginx/sites-enabled/default
RUN chmod +x /usr/local/bin/start-server

EXPOSE 80

CMD ["zsh"]
