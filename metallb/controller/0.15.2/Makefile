src: clean
	git clone --depth 1 -b v0.15.2-abi1 https://github.com/Loongson-Cloud-Community/metallb
image: src
	cd metallb && docker buildx build --build-arg https_proxy=${https_proxy}  -t cr.loongnix.cn/metallb/controller:0.15.2  -f controller/Dockerfile .

push:
	docker push cr.loongnix.cn/metallb/controller:0.15.2

clean:
	rm -rf  metallb
