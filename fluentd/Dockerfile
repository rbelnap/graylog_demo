FROM fluent/fluentd:v1.9.0-debian-1.0
USER root
RUN apt-get update \
 && apt-get -y install --no-install-recommends wget \
 && rm -rf /var/lib/apt/lists/*
RUN gem install fluent-plugin-rewrite-tag-filter
RUN gem install fluent-plugin-gelf-hs
RUN gem install gelf -v 3.0.0
COPY fluent.conf /fluentd/etc/
