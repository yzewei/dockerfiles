FROM cr.loongnix.cn/library/alpine:3.11.11 AS builder

LABEL maintainer="wangweijie@loongson.cn"

###############################################################
# set build args
###############################################################
ARG CURL_RELEASE_TAG=latest
ARG CURL_GIT_REPO=https://github.com/curl/curl.git
ARG CURL_CONFIGURE_OPTION=--with-openssl
ARG LABEL_VERSION=1.0.0
ARG LABEL_NAME=curl
ARG LABEL_DESC=curl

###############################################################
# build curl
###############################################################
# install deps and use latest curl release source
RUN apk update
RUN apk upgrade
RUN apk add libssh2 libssh2-dev libssh2-static \
        autoconf automake build-base                    \
        groff openssl curl-dev                          \
        python3 python3-dev                             \
        libtool curl stunnel perl                       \
        nghttp2 brotli brotli-dev

RUN mkdir /src
COPY "curl"  "/src/curl"
WORKDIR /src/curl

###############################################################
# get ca cert bundle from curl.haxx.se
###############################################################
RUN curl https://curl.haxx.se/ca/cacert.pem -L -o /cacert.pem

###############################################################
# build the tag version
###############################################################
RUN ./buildconf && \
    autoreconf -vif && \
    ./configure ${CURL_CONFIGURE_OPTION} &&\
    make -j$(nproc) &&\
    #make test &&\
    make DESTDIR="/alpine/" install  -j$(nproc)

###############################################################
# pinning image to Alpine 3.11.11
###############################################################
FROM cr.loongnix.cn/library/alpine:3.11.11

ARG CURL_RELEASE_TAG=latest
ARG CURL_RELEASE_VERSION
ARG CURL_GIT_REPO=https://github.com/curl/curl.git

ENV CURL_VERSION ${CURL_RELEASE_VERSION}
ENV CURL_RELEASE_TAG ${CURL_RELEASE_TAG}
ENV CURL_GIT_REPO ${CURL_GIT_REPO}

###############################################################
# dependencies
###############################################################
# RUN apk update
RUN apk upgrade
RUN apk add --no-cache brotli brotli-dev libssh2 nghttp2-dev && \
    rm -fr /var/cache/apk/*

###############################################################
# add non privileged curl user
###############################################################
RUN addgroup -S curl_group && adduser -S curl_user -G curl_group

###############################################################
# set curl ca bundle
###############################################################
COPY --from=builder "/cacert.pem" "/cacert.pem"
ENV CURL_CA_BUNDLE="/cacert.pem"

###############################################################
# install curl built from builder
###############################################################
COPY --from=builder "/alpine/usr/local/lib/libcurl.so.4.8.0" "/usr/lib/"
COPY --from=builder "/alpine/usr/local/bin/curl" "/usr/bin/curl"
COPY --from=builder "/alpine/usr/local/include/curl" "/usr/include/curl"

# explicitly set symlinks
RUN ln -s /usr/lib/libcurl.so.4.8.0 /usr/lib/libcurl.so.4
RUN ln -s /usr/lib/libcurl.so.4 /usr/lib/libcurl.so

###############################################################
# set user
###############################################################
#USER curl_user

###############################################################
# set entrypoint
###############################################################
COPY "entrypoint.sh" "/entrypoint.sh"
CMD ["curl"]
ENTRYPOINT ["/entrypoint.sh"]


