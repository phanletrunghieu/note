run:
	helm install rancher rancher-stable/rancher \
	--namespace rancher \
	--set hostname=rancher.my.org,\
	tls=external,\
	replicas=1

run-docker:
	docker run -d --restart=unless-stopped \
	-p 80:80 -p 443:443 \
	rancher/rancher:latest