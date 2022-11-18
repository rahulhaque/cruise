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
    && apt-get install -y php8.1-fpm php8.1-cli php8.1-dev \
       php8.1-pgsql php8.1-sqlite3 php8.1-gd \
       php8.1-curl \
       php8.1-imap php8.1-mysql php8.1-mbstring \
       php8.1-xml php8.1-zip php8.1-bcmath php8.1-soap \
       php8.1-intl php8.1-readline \
       php8.1-ldap \
       php8.1-msgpack php8.1-igbinary php8.1-redis php8.1-swoole \
       php8.1-pcov php8.1-xdebug \
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

RUN setcap "cap_net_bind_service=+ep" /usr/bin/php8.1

RUN groupadd --force -g $WWWGROUP cruise \
    && useradd -s /bin/bash --no-create-home --no-user-group -g $WWWGROUP -u $WWWUSER cruise \
    && chown -R $WWWUSER:$WWWGROUP /home/cruise

COPY start-server /usr/local/bin/start-server
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY supervisord.octane.conf /etc/supervisor/conf.d/supervisord.octane.conf
COPY php.ini /etc/php/8.1/cli/conf.d/99-php.ini
COPY php.ini /etc/php/8.1/fpm/conf.d/99-php.ini
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY nginx.octane.conf /etc/nginx/conf.d/octane.conf.bak
RUN chmod +x /usr/local/bin/start-server

EXPOSE 80

CMD ["zsh"]
