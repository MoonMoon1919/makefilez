# Docker stuff

PWD := $(shell pwd)
MAKEFILES_DIR := $(PWD)/makefilez/modules

include $(MAKEFILES_DIR)/project/Makefile
include $(MAKEFILES_DIR)/git/Makefile

DOCKER_CMD := docker

check/docker:
	@command -v $(DOCKER_CMD) > /dev/null 2>&1 || { echo >&2 echo "${DOCKER_CMD} not found, please install"; exit 1; }

docker/build: check/docker
	@$(DOCKER_CMD) build . -t $(PROJECT_NAME):$(GIT_SHA)
