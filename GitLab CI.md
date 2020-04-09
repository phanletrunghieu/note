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
variables:
  PACKAGE_PATH: /go/src/xxx
  # Configure postgres service (https://hub.docker.com/_/postgres/)
  POSTGRES_DB: db
  POSTGRES_USER: postgres
  POSTGRES_PASSWORD: "123456"
  DOCKER_DRIVER: overlay2
  CONTAINER_TEST_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
  CONTAINER_RELEASE_IMAGE: $CI_REGISTRY_IMAGE:release
  SERVER_TEST: 127.0.0.1
  SERVER_PROD: 127.0.0.2

cache:
  paths:
    - /apt-cache
    - /go/src/github.com
    - /go/src/golang.org
    - /go/src/google.golang.org
    - /go/src/gopkg.in
    - /go/src/firebase.google.com
    - ${PACKAGE_PATH}/vendor

stages:
  - prepare
  - test
  - build
  - deploy

test:
  image: $CONTAINER_DEP_IMAGE
  stage: test
  services:
    - postgres:10.4
    - redis:5.0.3-alpine
  before_script:
    - apk add --update git make curl postgresql-client
    - cd ${PACKAGE_PATH}
    - cd ../
    - mv ${PACKAGE_PATH} temp
    - ln -s ${CI_PROJECT_DIR} ${PACKAGE_PATH}
    - mv temp/vendor ${PACKAGE_PATH}
    - cd ${PACKAGE_PATH}
  script:
    - export PGPASSWORD=$POSTGRES_PASSWORD
    - export POSTGREST_DB_HOST=postgres
    - psql -h "postgres" -U "$POSTGRES_USER" -d "$POSTGRES_DB" -f database/db.sql
    - make test

build-test:
  image: docker:stable
  stage: build
  only:
    - development
  services:
    - docker:dind
  script:
    - docker login -u gitlab-ci-token -p $CI_JOB_TOKEN $CI_REGISTRY
    - docker build -t $CONTAINER_TEST_IMAGE .
    - docker push $CONTAINER_TEST_IMAGE

build-release:
  image: docker:stable
  stage: build
  only:
    - /^v.+$/
  except:
    - branches
  services:
    - docker:dind
  script:
    - docker login -u gitlab-ci-token -p $CI_JOB_TOKEN $CI_REGISTRY
    - docker build -t $CONTAINER_RELEASE_IMAGE .
    - docker push $CONTAINER_RELEASE_IMAGE

deploy-test:
  image: docker:stable
  stage: deploy
  only:
    - development
  services:
    - docker:dind
  environment:
    name: Test
  before_script:
    - apk --update add openssh-client
    - eval $(ssh-agent -s)
    - cat docker-compose/app/deploy_key_test | tr -d '\r' | ssh-add - > /dev/null
    - ssh-add -l
    - mkdir -p ~/.ssh
    - echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config
    - apk add socat
  script:
    - docker login -u gitlab-ci-token -p $CI_JOB_TOKEN $CI_REGISTRY
    - docker pull $CONTAINER_TEST_IMAGE
    - docker save $CONTAINER_TEST_IMAGE > image.tar
    - scp image.tar root@$SERVER_TEST:~/xxx/

    - scp .env.test root@$SERVER_TEST:~/xxx/.env
    - scp docker-compose/app/docker-compose.yml root@$SERVER_TEST:~/xxx
    - scp -r database/sql/* root@$SERVER_TEST:~/xxx/migrate/sql
    - ssh -tt root@$SERVER_TEST "cd xxx &&
      docker load < image.tar &&
      docker-compose -f docker-compose.yml -p xxx down &&
      docker-compose -f docker-compose.yml -p xxx up -d &&
      cd migrate &&
      ./migrate-up.sh"

deploy-release:
  image: docker:stable
  stage: deploy
  only:
    - /^v.+$/
  except:
    - branches
  services:
    - docker:dind
  environment:
    name: Production
  before_script:
    - apk --update add openssh-client
    - eval $(ssh-agent -s)
    - cat docker-compose/app/deploy_key_prod | tr -d '\r' | ssh-add - > /dev/null
    - ssh-add -l
    - mkdir -p ~/.ssh
    - echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config
    - apk add socat
  script:
    - docker login -u gitlab-ci-token -p $CI_JOB_TOKEN $CI_REGISTRY
    - docker pull $CONTAINER_RELEASE_IMAGE
    - docker save $CONTAINER_RELEASE_IMAGE > image.tar
    - scp image.tar root@$SERVER_PROD:~/xxx/

    - scp .env.prod root@$SERVER_PROD:~/xxx/.env
    - scp docker-compose/app/docker-compose.yml root@$SERVER_PROD:~/xxx
    - scp -r database/sql/* root@$SERVER_PROD:~/xxx/migrate/sql
    - ssh -tt root@$SERVER_PROD "cd xxx &&
      docker load < image.tar &&
      docker-compose -f docker-compose.yml -p xxx down &&
      docker-compose -f docker-compose.yml -p xxx up -d &&
      cd migrate &&
      ./migrate-up.sh"

    - ssh -tt root@$SERVER_PROD_2 "cd xxx &&
      docker load < image.tar &&
      docker-compose -f docker-compose.yml -p xxx down &&
      docker-compose -f docker-compose.yml -p xxx up -d"
```

## Use docker images from a private registry

Set project env DOCKER_AUTH_CONFIG with value in .docker/config.json

https://docs.gitlab.com/ee/ci/docker/using_docker_images.html#define-an-image-from-a-private-container-registry
