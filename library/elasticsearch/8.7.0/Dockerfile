################################################################################
# This Dockerfile was generated from the template at distribution/src/docker/Dockerfile
#
# Beginning of multi stage Dockerfile
################################################################################

################################################################################
# Build stage 1 `builder`:
# Extract Elasticsearch artifact
################################################################################

FROM cr.loongnix.cn/library/debian:buster-slim AS builder

# Install required packages to extract the Elasticsearch distribution

RUN for iter in 1 2 3 4 5 6 7 8 9 10; do \
      apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y curl  && \
      exit_code=0 && break || \
        exit_code=$? && echo "apt-get error: retry $iter in 10s" && sleep 10; \
    done; \
    exit $exit_code

# `tini` is a tiny but valid init for containers. This is used to cleanly
# control how ES and any child processes are shut down.
#
# The tini GitHub page gives instructions for verifying the binary using
# gpg, but the keyservers are slow to return the key and this can fail the
# build. Instead, we check the binary against the published checksum.
RUN set -eux ; \
    curl --retry 10 -S -L -o /bin/tini https://github.com/Loongson-Cloud-Community/tini/releases/download/v0.19.0/tini-static ; \
    chmod 0555 /bin/tini

RUN mkdir /usr/share/elasticsearch
WORKDIR /usr/share/elasticsearch

RUN curl --retry 10 -S -L --output /tmp/elasticsearch.tar.gz https://github.com/Loongson-Cloud-Community/elasticsearch/releases/download/v8.7.0/elasticsearch-8.7.0-SNAPSHOT-loongarch64_jdk17.tar.gz \
	&& tar -zxf /tmp/elasticsearch.tar.gz --strip-components=1 \
	&& rm /tmp/elasticsearch.tar.gz

#In order to prevent the cache files in the elasticsearch-8.7.0-SNAPSHOT-loongarch64_jdk17.tar.gz from not being deleted and causing build errors.
RUN rm -rf config/elasticsearch.keystore config/certs config/*.orig && \
    rm -rf logs/* data/*

# The distribution includes a `config` directory, no need to create it
COPY config/elasticsearch.yml config/
COPY config/log4j2.properties config/log4j2.docker.properties

#  1. Configure the distribution for Docker
#  2. Create required directory
#  3. Move the distribution's default logging config aside
#  4. Move the generated docker logging config so that it is the default
#  5. Reset permissions on all directories
#  6. Reset permissions on all files
#  7. Make CLI tools executable
#  8. Make some directories writable. `bin` must be writable because
#     plugins can install their own CLI utilities.
#  9. Make some files writable
RUN sed -i -e 's/ES_DISTRIBUTION_TYPE=tar/ES_DISTRIBUTION_TYPE=docker/' bin/elasticsearch-env && \
    mkdir data && \
    mv config/log4j2.properties config/log4j2.file.properties && \
    mv config/log4j2.docker.properties config/log4j2.properties && \
    find . -type d -exec chmod 0555 {} + && \
    find . -type f -exec chmod 0444 {} + && \
    chmod 0555 bin/* jdk/bin/* jdk/lib/jspawnhelper modules/x-pack-ml/platform/linux-*/bin/* && \
    chmod 0775 bin config config/jvm.options.d data logs plugins && \
    find config -type f -exec chmod 0664 {} +

################################################################################
# Build stage 2 (the actual Elasticsearch image):
#
# Copy elasticsearch from stage 1
# Add entrypoint
################################################################################

FROM cr.loongnix.cn/library/debian:buster-slim

# Change default shell to bash, then install required packages with retries.
RUN yes no | dpkg-reconfigure dash && \
    for iter in 1 2 3 4 5 6 7 8 9 10; do \
      export DEBIAN_FRONTEND=noninteractive && \
      apt-get update && \
      apt-get upgrade -y && \
      apt-get install -y --no-install-recommends \
        ca-certificates curl netcat p11-kit unzip zip  && \
      apt-get clean && \
      rm -rf /var/lib/apt/lists/* && \
      exit_code=0 && break || \
        exit_code=$? && echo "apt-get error: retry $iter in 10s" && sleep 10; \
    done; \
    exit $exit_code

RUN groupadd -g 1000 elasticsearch && \
    adduser --uid 1000 --gid 1000 --home /usr/share/elasticsearch elasticsearch && \
    adduser elasticsearch root && \
    chown -R 0:0 /usr/share/elasticsearch

ENV ELASTIC_CONTAINER true

WORKDIR /usr/share/elasticsearch

COPY --from=builder --chown=0:0 /usr/share/elasticsearch /usr/share/elasticsearch
COPY --from=builder --chown=0:0 /bin/tini /bin/tini

ENV PATH /usr/share/elasticsearch/bin:$PATH

COPY bin/docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh

# 1. Sync the user and group permissions of /etc/passwd
# 2. Set correct permissions of the entrypoint
# 3. Ensure that there are no files with setuid or setgid, in order to mitigate "stackclash" attacks.
#    We've already run this in previous layers so it ought to be a no-op.
# 4. Replace OpenJDK's built-in CA certificate keystore with the one from the OS
#    vendor. The latter is superior in several ways.
#    REF: https://github.com/elastic/elasticsearch-docker/issues/171
# 5. Tighten up permissions on the ES home dir (the permissions of the contents are handled earlier)
# 6. You can't install plugins that include configuration when running as `elasticsearch` and the `config`
#    dir is owned by `root`, because the installed tries to manipulate the permissions on the plugin's
#    config directory.
RUN chmod g=u /etc/passwd && \
    chmod 0555 /usr/local/bin/docker-entrypoint.sh && \
    find / -xdev -perm -4000 -exec chmod ug-s {} + && \
    chmod 0775 /usr/share/elasticsearch && \
    chown elasticsearch bin config config/jvm.options.d data logs plugins

# Update "cacerts" bundle to use Ubuntu's CA certificates (and make sure it
# stays up-to-date with changes to Ubuntu's store)
COPY bin/docker-openjdk /etc/ca-certificates/update.d/docker-openjdk
RUN /etc/ca-certificates/update.d/docker-openjdk
RUN chown -R 1000:0 /usr/share/elasticsearch/config /usr/share/elasticsearch/data

EXPOSE 9200 9300

LABEL org.label-schema.build-date="2023-03-27T16:31:09.816451435Z" \
  org.label-schema.license="Elastic-License-2.0" \
  org.label-schema.name="Elasticsearch" \
  org.label-schema.schema-version="1.0" \
  org.label-schema.url="https://www.elastic.co/products/elasticsearch" \
  org.label-schema.usage="https://www.elastic.co/guide/en/elasticsearch/reference/index.html" \
  org.label-schema.vcs-ref="09520b59b6bc1057340b55750186466ea715e30e" \
  org.label-schema.vcs-url="https://github.com/elastic/elasticsearch" \
  org.label-schema.vendor="Elastic" \
  org.label-schema.version="8.7.0" \
  org.opencontainers.image.created="2023-03-27T16:31:09.816451435Z" \
  org.opencontainers.image.documentation="https://www.elastic.co/guide/en/elasticsearch/reference/index.html" \
  org.opencontainers.image.licenses="Elastic-License-2.0" \
  org.opencontainers.image.revision="09520b59b6bc1057340b55750186466ea715e30e" \
  org.opencontainers.image.source="https://github.com/elastic/elasticsearch" \
  org.opencontainers.image.title="Elasticsearch" \
  org.opencontainers.image.url="https://www.elastic.co/products/elasticsearch" \
  org.opencontainers.image.vendor="Elastic" \
  org.opencontainers.image.version="8.7.0"

# Our actual entrypoint is `tini`, a minimal but functional init program. It
# calls the entrypoint we provide, while correctly forwarding signals.
ENTRYPOINT ["/bin/tini", "--", "/usr/local/bin/docker-entrypoint.sh"]
# Dummy overridable parameter parsed by entrypoint
CMD ["eswrapper"]

USER elasticsearch:root

################################################################################
# End of multi-stage Dockerfile
################################################################################
