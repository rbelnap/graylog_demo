FROM fluent/fluentd:v1.9.0-debian-1.0
USER root
RUN apt-get update \
 && apt-get -y install --no-install-recommends wget \
 && rm -rf /var/lib/apt/lists/*
RUN gem install fluent-plugin-rewrite-tag-filter
RUN gem install gelf
RUN cd /fluentd/plugins \
 && wget -q https://raw.githubusercontent.com/emsearcy/fluent-plugin-gelf/master/lib/fluent/plugin/out_gelf.rb
COPY fluent.conf /fluentd/etc/
