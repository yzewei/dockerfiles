FROM cr.loongnix.cn/library/debian:buster as installer
#FROM cr.loongnix.cn/openanolis/anolisos:8.8 as installer
#FROM cr.loongnix.cn/loongson/loongnix-server:8.4 as installer

ARG NACOS_VERSION=2.2.3
ARG HOT_FIX_FLAG=""

RUN apt-get update -y && \
apt-get install -y --no-install-recommends ca-certificates curl gnupg netbase wget && \
rm -rf /var/lib/apt/lists/*
#RUN dnf update -y && dnf intall -y curl gnupg glibc-common iproute net-tools bind-utils  wget
#RUN dnf update -y && dnf intall -y curl gnupg wget

RUN set -x \
    && https_proxy="" curl -SL https://cloud.loongnix.cn/releases/loongarch64abi1/alibaba/nacos/2.2.3/nacos-server-2.2.3.tar.gz --output /var/tmp/nacos-server.tar.gz  \
    && tar -xzvf /var/tmp/nacos-server.tar.gz -C /home \
    && rm -rf /var/tmp/nacos-server.tar.gz /home/nacos/bin/* /home/nacos/conf/*.properties /home/nacos/conf/*.example

FROM cr.loongnix.cn/library/openjdk:17-buster

# set environment
ENV MODE="cluster" \
    PREFER_HOST_MODE="ip"\
    BASE_DIR="/home/nacos" \
    CLASSPATH=".:/home/nacos/conf:$CLASSPATH" \
    CLUSTER_CONF="/home/nacos/conf/cluster.conf" \
    FUNCTION_MODE="all" \
    NACOS_USER="nacos" \
    JAVA="/usr/local/openjdk-17/bin/java" \
    JVM_XMS="1g" \
    JVM_XMX="1g" \
    JVM_XMN="512m" \
    JVM_MS="128m" \
    JVM_MMS="320m" \
    NACOS_DEBUG="n" \
    TOMCAT_ACCESSLOG_ENABLED="false" \
    TZ="Asia/Shanghai"

WORKDIR $BASE_DIR

# copy nacos bin
COPY --from=installer ["/home/nacos", "/home/nacos"]
RUN mkdir -p /home/nacos/init.d/

ADD bin/docker-startup.sh bin/docker-startup.sh
#ADD conf/application.properties conf/application.properties
ADD conf/ conf/


# set startup log dir
RUN mkdir -p logs \
    && cd logs \
    && touch start.out \
    && ln -sf /dev/stdout start.out \
    && ln -sf /dev/stderr start.out
RUN chmod +x bin/docker-startup.sh

EXPOSE 8848
ENTRYPOINT ["bin/docker-startup.sh"]
