FROM fluent/fluentd:v1.4.2-debian-2.0

LABEL maintainer="Andreas Ntalakas <antalakas@tiledb.io>"
USER root
WORKDIR /home/fluent
ENV PATH /fluentd/vendor/bundle/ruby/2.6.0/bin:$PATH
ENV GEM_PATH /fluentd/vendor/bundle/ruby/2.6.0
ENV GEM_HOME /fluentd/vendor/bundle/ruby/2.6.0
# skip runtime bundler installation
ENV FLUENTD_DISABLE_BUNDLER_INJECTION 1

COPY Gemfile* /fluentd/
  RUN buildDeps="sudo make gcc g++ libc-dev ruby-dev libffi-dev" \
     && apt-get update \
     && apt-get upgrade -y \
     && apt-get install \
     -y --no-install-recommends \
     $buildDeps build-essential curl libgeoip-dev ca-certificates libjemalloc1 ruby-dev libmaxminddb-dev net-tools \
    && gem install bundler --version 1.16.2 \
    && bundle config silence_root_warning true \
    && bundle install --gemfile=/fluentd/Gemfile --path=/fluentd/vendor/bundle \
    && SUDO_FORCE_REMOVE=yes \
    apt-get purge -y --auto-remove \
                  -o APT::AutoRemove::RecommendsImportant=false \
                  $buildDeps \
    && rm -rf /var/lib/apt/lists/* \
    && gem sources --clear-all \
    && rm -rf /tmp/* /var/tmp/* /usr/lib/ruby/gems/*/cache/*.gem \
    && curl https://geolite.maxmind.com/download/geoip/database/GeoLite2-City.tar.gz | tar xz \
    && mv "GeoLite2-City_"* "GeoLite2-City" \
    && mv GeoLite2-City/GeoLite2-City.mmdb /home/fluent \
    && ulimit -n 65536

# Copy configuration file
COPY fluent.conf /etc/fluent/fluent.conf

# Copy script to execute
COPY run.sh /run.sh

EXPOSE 9880
EXPOSE 80

ENV LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libjemalloc.so.1

# Overwrite ENTRYPOINT to run fluentd as root for /var/log / /var/lib
ENTRYPOINT ["tini", "--", "/run.sh"]