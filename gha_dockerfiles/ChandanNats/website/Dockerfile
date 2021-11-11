ARG RUBY_VERSION=3

FROM ruby:${RUBY_VERSION}-buster as dev

ENV TZ=US/Pacific

ARG NODE_VERSION=15
ENV NODE_VERSION=$NODE_VERSION

RUN curl -sL https://deb.nodesource.com/setup_${NODE_VERSION}.x -o node_setup.sh && \
      bash node_setup.sh 1> /dev/null
RUN apt-get update && apt-get install -yq --no-install-recommends \
      build-essential \
      vim \
      git \
      diffutils \
      xdg-user-dirs \
      nodejs && \
      rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Install Ruby deps
ENV JEKYLL_ENV=development
RUN gem install bundler
COPY Gemfile Gemfile.lock ./
RUN bundle install

# Install Node deps
ENV NODE_ENV=development
COPY package.json package-lock.json ./
RUN npm install -g npm firebase-tools superstatic
RUN npm install

COPY ./ ./

# So that we have the most up to date submodules
RUN git submodule update --init --recursive

ARG FLUTTER_BRANCH
ENV FLUTTER_BRANCH=$FLUTTER_BRANCH
ENV FLUTTER_ROOT=flutter
ENV FLUTTER_BIN=flutter/bin
ENV PATH="/app/flutter/bin:$PATH"

# Used if wanting to build the container with a different branch
# e.g. `make FLUTTER_BRANCH=dev build`
# This is not to be confused with the $TEST_TARGET_CHANNEL
RUN if test -n "$FLUTTER_BRANCH" -a "$FLUTTER_BRANCH" != "stable" ; then \
      cd flutter && \
      git fetch && \
      git remote set-branches origin "$FLUTTER_BRANCH" && \
      git fetch --depth 1 origin "$FLUTTER_BRANCH" && \
      git checkout "$FLUTTER_BRANCH" -- && \
      git pull; \
    fi

# Set up Flutter
RUN flutter doctor --suppress-analytics --quiet
RUN flutter --version
RUN dart pub get

EXPOSE 35729
EXPOSE 4002


# -- Test target
# NOTE that instead of have a script that tests all targets,
# we could build a new docker container for each test and 
# start clean
FROM dev as test
ARG DISABLE_TESTS
ENV DISABLE_TESTS=$DISABLE_TESTS
ARG TEST_TARGET_CHANNEL=stable
ENV TEST_TARGET_CHANNEL=$TEST_TARGET_CHANNEL
RUN tool/test.sh --target $TEST_TARGET_CHANNEL --check-links --null-safety


# -- Build target
FROM test AS builder

ENV JEKYLL_ENV=production
ENV NODE_ENV=production

RUN bundle install
RUN npm install
RUN cd flutter && \
      git fetch && \
      git remote set-branches origin stable && \
      git fetch origin stable && \
      git checkout stable && \
      git pull

RUN flutter doctor

# NOTE this is a bit sneaky and we could be more clear about it 
# by having an actual production template for robots.txt rather 
# than changing the contents of the file at build/test time. 
# This is similarly seen in tool/check-links.sh
RUN echo "User-agent: *" > src/robots.txt && echo "Allow: /" >> src/robots.txt

RUN bundle exec jekyll build --config _config.yml


# -- Deploy target
FROM builder AS deploy

ARG FIREBASE_TOKEN
ENV FIREBASE_TOKEN=$FIREBASE_TOKEN
ARG FIREBASE_ALIAS=default
ENV FIREBASE_ALIAS=${FIREBASE_ALIAS:-default}
ARG COMMIT=$(git rev-parse --short HEAD)
ENV COMMIT=$COMMIT

RUN firebase use $FIREBASE_ALIAS
RUN firebase deploy -m $COMMIT \
      --only hosting \
      --non-interactive \
      --token $FIREBASE_TOKEN \
      --project $FIREBASE_ALIAS \
      --debug \
	--json
