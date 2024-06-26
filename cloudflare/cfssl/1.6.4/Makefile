# This file is generated by the template.

REGISTRY?=cr.loongnix.cn
ORGANIZATION?=cloudflare
REPOSITORY?=cfssl
TAG?=v1.6.4
LATEST?=true

GO_IMG?=cr.loongnix.cn/library/golang:1.19
GOPROXY?=https://goproxy.cn,https://mirrors.aliyun.com/goproxy/,https://goproxy.io,direct

IMAGE=$(REGISTRY)/$(ORGANIZATION)/$(REPOSITORY):$(patsubst v%,%,$(TAG))
LATEST_IMAGE=$(REGISTRY)/$(ORGANIZATION)/$(REPOSITORY):latest

SOURCE_URL=https://github.com/cloudflare/cfssl.git
SOURCE=$(shell echo $(SOURCE_URL) | awk -F '/' '{print $$NF}' | awk -F '.' '{print $$1}')
PATCH=0001-support-loong64.patch

default: image

src/$(SOURCE):
	git clone -b $(TAG) --depth=1 $(SOURCE_URL) $@
	cd $@ && \
		git apply ../../$(PATCH)

build: src/$(SOURCE)
	docker run --rm \
		-e GOARCH=loong64 \
		-e GOPROXY=$(GOPROXY) \
		-e CGO_ENABLED=0 \
		-v `pwd`/src/$(SOURCE):/v -w /v \
		$(GO_IMG) sh -c 'TARGETOS=linux TARGETARCH=loong64 make all'

image: src/$(SOURCE) build
# Commands for building.
	docker build \
		-t $(IMAGE) \
		--build-arg=IMG=cr.loongnix.cn/prometheus/busybox:glibc \
		src/$(SOURCE)

push:
	docker push $(IMAGE)
	@if [ $(LATEST) = "true" ]; \
		then\
		docker tag $(IMAGE) $(LATEST_IMAGE); \
		docker push $(LATEST_IMAGE); \
	fi

clean:
	rm -rf src
