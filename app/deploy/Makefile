#!/usr/bin/env make
# Copyright 2020 Cloud Elements. All rights reserved.
.DEFAULT_GOAL := help

ifeq ($(shell which jq),)
  $(error jq binary not found)
endif

myname := "installers"
mydir := $(shell pwd)
image := cloudelements/myapp-installer
tag := $(shell jq -r ".version" "$(mydir)/package.json")

# clean: @ cleans temporary files from repo
clean:
	@printf "\033[36m## $(myname): clean\033[0m\n"
	docker rmi $(image)
	docker image prune --force
	rm -rf /tmp/trivy/cache
.PHONY: clean

# help: @ lists available tasks
help:
	@grep -E '[a-zA-Z\.\-]+:.*?@ .*$$' $(MAKEFILE_LIST)| tr -d '#'  | awk 'BEGIN {FS = ":.*?@ "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'
.PHONY: help

# release.artifact: @ creates deployable release artifact
release.artifact:
	@printf "\033[36m## $(myname): release.artifact\033[0m\n"
	@which docker 1>/dev/null || { printf "docker binary not found\n" && exit 1 ;}
	docker build --pull --no-cache -f "$(mydir)/Dockerfile" -t "$(image)" "$(mydir)"
	docker tag "$(image)" "$(image):$(tag)"
	docker tag "$(image)" "$(image):latest"
.PHONY: release.artifact

# release.publish: @ publishes release artifact(s)
release.publish:
	@printf "\033[36m## $(myname): release.publish\033[0m\n"
	@{ test -n "$$DOCKER_USERNAME" && test -n "$$DOCKER_PASSWORD" \
	  && docker login --username "$$DOCKER_USERNAME" --password "$$DOCKER_PASSWORD" ;} || true
	docker push "$(image):$(tag)"
	docker push "$(image):latest"
.PHONY: release.publish

# release.security: @ performs security scans on artifact
release.security:
	@printf "\033[36m## $(myname): release.security\033[0m\n"
	@which docker 1>/dev/null || { printf "docker binary not found\n" && exit 1 ;}
	@mkdir -p /tmp/trivy/cache
	docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
	  -v /tmp/trivy/cache/:/root/.cache/ aquasec/trivy --clear-cache
	docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
	  -v /tmp/trivy/cache/:/root/.cache/ aquasec/trivy \
	  --no-progress --exit-code 0 --severity LOW,MEDIUM --ignore-unfixed $(image):$(tag)
	docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
	  -v /tmp/trivy/cache/:/root/.cache/ aquasec/trivy \
	  --no-progress --exit-code 1 --severity HIGH,CRITICAL --ignore-unfixed $(image):$(tag)
.PHONY: release.security

# test.lint: @ runs linting checks
test.lint:
	@printf "\033[36m## $(myname): test.lint\033[0m\n"
	@which docker 1>/dev/null || { printf "docker binary not found\n" && exit 1 ;}
	docker run -i --rm hadolint/hadolint < $(mydir)/Dockerfile
.PHONY: test.lint
