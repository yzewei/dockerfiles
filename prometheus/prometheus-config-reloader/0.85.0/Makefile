src:
	git clone --depth 1 -b v0.85.0 https://github.com/prometheus-operator/prometheus-operator
image: src
	cd prometheus-operator && git apply  ../0001.patch && make prometheus-config-reloader-image
push:
	docker push cr.loongnix.cn/prometheus/prometheus-config-reloader:0.85.0

