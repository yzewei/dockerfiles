src:
	git clone --depth 1 -b v1.4.1 https://github.com/bats-core/bats-core
image: src
	cd bats-core && git apply ../0001.patch && docker build -t cr.loongnix.cn/bats/bats:1.4.1 .
push:
	docker push cr.loongnix.cn/bats/bats:1.4.1

