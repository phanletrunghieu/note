# CI/CD with GitLab Runner

## config.toml
```toml
concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "Runner Hieu Dep Trai"
  url = "https://xxx.xxx"
  token = "sacdscsd"
  executor = "docker"
  [runners.custom_build_dir]
  [runners.docker]
    tls_verify = false
    image = "alpine:latest"
    privileged = true
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache"]
    shm_size = 0
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
```

## Cài đặt GitLab Runner
```bash
docker run -t -i --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:alpine-v11.10.1
```

## Run
```bash
docker exec -it gitlab-runner gitlab-runner run &
```

## .gitlab-ci.yml
Example with Go
```yml
# This file is a template, and might need editing before it works on your project.
image:  golang:1.10 

services:
	- postgres:10.4
	- mysql:5.7.20

variables:
	PACKAGE_PATH:  /go/src/gitlab.com/phanletrunghieu/bot-net
	# Configure postgres service (https://hub.docker.com/_/postgres/)
	POSTGRES_DB:  nice_marmot
	POSTGRES_USER:  runner
	POSTGRES_PASSWORD:  ""
	# Configure mysql environment variables (https://hub.docker.com/_/mysql/)
	MYSQL_DATABASE:  el_duderino
	MYSQL_ROOT_PASSWORD:  mysql_strong_password

cache:
	paths:
		- /apt-cache
		- /go/src/github.com
		- /go/src/golang.org
		- /go/src/google.golang.org
		- /go/src/gopkg.in

stages:
	- test
	- build

before_script:
	- go get -u github.com/golang/dep/cmd/dep
	- curl -sfL https://install.goreleaser.com/github.com/golangci/golangci-lint.sh | sh -s -- -b $(go env GOPATH)/bin v1.15.0
	- mkdir -p $(dirname ${PACKAGE_PATH})
	- ln -s ${CI_PROJECT_DIR} ${PACKAGE_PATH}
	- cd ${PACKAGE_PATH}
	- dep ensure

connect_pg:
	image: postgres
	script:
		# official way to provide password to psql: http://www.postgresql.org/docs/9.3/static/libpq-envars.html
		- export PGPASSWORD=$POSTGRES_PASSWORD
		- psql -h "postgres" -U "$POSTGRES_USER" -d "$POSTGRES_DB" -c "SELECT 'OK' AS status;"

connect_mysql:
	image: mysql
	script:
		- echo "SELECT 'OK';" | mysql --user=root --password="$MYSQL_ROOT_PASSWORD" --host=mysql "$MYSQL_DATABASE"

unit_tests:
	stage: test
	script:
		- go test ./...
		- golangci-lint run

build:
	stage:  build
	script:
		- make build
```
