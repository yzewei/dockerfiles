# This file is generated by the template.

REGISTRY     ?= cr.loongnix.cn
ORGANIZATION ?= library
REPOSITORY   ?= golang
TAG          ?= 1.22-buster

IMAGE  = $(REGISTRY)/$(ORGANIZATION)/$(REPOSITORY):$(TAG)
ALIAS  = $(REGISTRY)/$(ORGANIZATION)/$(REPOSITORY):1.22
LATEST = $(REGISTRY)/$(ORGANIZATION)/$(REPOSITORY):latest

default: image

image:
	docker build \
		-t $(IMAGE) \
		.

push: alias
	docker push $(IMAGE)
	docker push $(ALIAS)

ifneq ($(LATEST),)
	docker tag $(IMAGE) $(LATEST)
	docker push $(LATEST)
endif

alias:
	docker tag $(IMAGE) $(ALIAS)
