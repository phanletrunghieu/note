install:
	# helm repo add gitlab https://charts.gitlab.io
	# kubectl create namespace gitlab
	helm install --namespace gitlab gitlab-runner gitlab/gitlab-runner \
	--set rbac.create=true,\
	gitlabUrl=http://192.168.1.11:8929,\
	runnerRegistrationToken=yRY5bGJsoDS75yPUKuZQ,\
	image=gitlab/gitlab-runner:alpine-v11.6.0,\
	runners.image=alpine:3.11.3,\
	runners.privileged=true