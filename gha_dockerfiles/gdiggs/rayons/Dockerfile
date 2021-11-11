FROM gordondiggs/ruby-node:2.6

RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add --no-tty - && \
    apt-get update -qq && \
    apt-get install -y build-essential libpq-dev nodejs postgresql-client && \
    rm -rf /var/lib/apt/lists/*

ENV BUNDLE_BUILD__SASSC="--disable-march-tune-native"
ENV BINSTUBS_DIR="/usr/local/binstubs"
ENV PATH="$BINSTUBS_DIR:$PATH"

RUN mkdir -p "$BINSTUBS_DIR"

RUN gem update --system
RUN gem install bundler --version 2.1.2

RUN mkdir /app
WORKDIR /app

COPY Gemfile* ./

RUN bundle install \
    --jobs "$(nproc)" \
    --binstubs="$BINSTUBS_DIR"

COPY . .

RUN rake RAILS_ENV=production assets:precompile
