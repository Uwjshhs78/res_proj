FROM ghcr.io/linuxserver/baseimage-alpine-nginx:3.13

# set version label
ARG BUILD_DATE
ARG VERSION
ARG LYCHEE_VERSION
LABEL build_version="Linuxserver.io version:- ${VERSION} Build-date:- ${BUILD_DATE}"
LABEL maintainer="hackerman"

RUN \
  echo "**** install build packages ****" && \
  apk add --no-cache --virtual=build-dependencies \
    composer \
    git && \
  echo "**** install runtime packages ****" && \
  apk add --no-cache \
    curl \
    exiftool \
    ffmpeg \
    gd \
    imagemagick \
    jpegoptim \
    php7-bcmath \
    php7-ctype \
    php7-dom \
    php7-exif \
    php7-gd \
    php7-imagick \
    php7-intl \
    php7-json \
    php7-mbstring \
    php7-mysqli \
    php7-pdo_mysql \
    php7-session \
    php7-tokenizer \
    php7-xml \
    php7-zip && \
  echo "**** install lychee ****" && \
  mkdir -p /app/lychee && \
  if [ -z ${LYCHEE_VERSION} ]; then \
    LYCHEE_VERSION=$(curl -sX GET "https://api.github.com/repos/LycheeOrg/Lychee/releases/latest" \
    | awk '/tag_name/{print $4;exit}' FS='[""]'); \
  fi && \
  curl -o \
    /tmp/lychee.tar.gz -L \
    "https://github.com/LycheeOrg/Lychee/archive/${LYCHEE_VERSION}.tar.gz" && \
  tar xf \
  /tmp/lychee.tar.gz -C \
    /app/lychee/ --strip-components=1 && \
  cd /app/lychee && \
  echo "**** install composer dependencies ****" && \
  composer install \
    -d /app/lychee \
    --no-dev \
    --no-suggest \
    --no-interaction && \
  echo "**** cleanup ****" && \
  apk del --purge \
    build-dependencies && \
  rm -rf \
    /root/.cache \
    /root/.composer \
    /tmp/*

# add local files
COPY root/ /
