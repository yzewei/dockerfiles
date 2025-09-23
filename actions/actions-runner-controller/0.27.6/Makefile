src:
	git clone --depth 1 -b v0.27.6 https://github.com/actions/actions-runner-controller src && cd src && \
		git apply ../0001-add.patch

image: src
	cd src && make docker-buildx
push:
	docker  push cr.loongnix.cn/actions/actions-runner-controller:0.27.6

