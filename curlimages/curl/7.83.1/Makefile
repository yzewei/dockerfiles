# This file is generated by the template.

REGISTRY?=cr.loongnix.cn
ORGANIZATION?=curlimages
REPOSITORY?=curl
TAG?=7.83.1

IMAGE=$(REGISTRY)/$(ORGANIZATION)/$(REPOSITORY):$(TAG)

CURL_RELEASE_TAG?=curl-7_83_1
CURL_GIT_REPO?=https://github.com/curl/curl.git

default: image

image: source replace build


source:
	git clone --depth 1 --branch $(CURL_RELEASE_TAG) $(CURL_GIT_REPO)

TARGET_FILE_PATH:=curl/lib
TARGET_FILE:=easy_lock.h

replace:
	cp $(TARGET_FILE) $(TARGET_FILE_PATH)/	

build:
	docker build --no-cache \
		-t $(IMAGE) \
		.

push:
	docker push $(IMAGE)
