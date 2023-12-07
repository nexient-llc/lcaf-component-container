SHELL := /bin/bash

# Binaries

DOCKER ?= docker
DOCKER_COMPOSE ?= docker-compose

TERRAFORM ?= terraform
RM ?= rm -rf
CONFTEST ?= conftest
TFLINT ?= tflint
FIND ?= find

# Variables

CONFTEST_POLICY_DIRECTORIES ?= $(MODULE_DIR)/policy
DOCKER_BULD_ARCH ?= linux/amd64,linux/arm64

# Functions

define conftest_terraform_module
	$(CONFTEST) test $(1)/terraform.tfplan.json $(foreach dir,$(CONFTEST_POLICY_DIRECTORIES),$(shell echo "-p $(dir) "));
endef

# The names of the required environment variables
CONTAINER_REGISTRY ?=
CONTAINER_IMAGE_NAME ?=
CONTAINER_IMAGE_VERSION ?=

# The target that checks if the required environment variables are set
.PHONY: docker/check-env-vars
docker/check-env-vars:
ifeq ($(CONTAINER_REGISTRY),)
    $(error CONTAINER_REGISTRY is not set. Please set it before running make.)
endif
ifeq ($(CONTAINER_IMAGE_NAME),)
    $(error CONTAINER_IMAGE_NAME is not set. Please set it before running make.)
endif
ifeq ($(CONTAINER_IMAGE_VERSION),)
    $(error CONTAINER_IMAGE_VERSION is not set. Please set it before running make.)
endif


# Tasks

.PHONY: docker/aws_ecr_login
docker/aws_ecr_login:
	aws ecr get-login-password | $(DOCKER) login --username AWS --password-stdin $(CONTAINER_REGISTRY)

.PHONY: docker/build
docker/build : docker/check-env-vars
	@echo "Building with CONTAINER_REGISTRY=$(CONTAINER_REGISTRY), CONTAINER_IMAGE_NAME=$(CONTAINER_IMAGE_NAME), CONTAINER_IMAGE_VERSION=$(CONTAINER_IMAGE_VERSION)"
	$(DOCKER) buildx build --platform $(DOCKER_BULD_ARCH) $(BUILD_ARGS) -t $(CONTAINER_REGISTRY)/$(CONTAINER_IMAGE_NAME):$(CONTAINER_IMAGE_VERSION) --load .

.PHONY: docker/push
docker/push : docker/build
	$(DOCKER) buildx build --platform $(DOCKER_BULD_ARCH) $(BUILD_ARGS) -t $(CONTAINER_REGISTRY)/$(CONTAINER_IMAGE_NAME):$(CONTAINER_IMAGE_VERSION) --push .

.PHONY: check
check::
	$(MAKE) docker/build

.PHONY: docker_compose/start
docker_compose/start:
	@if [ -f 'compose.yaml' ] || [ -f 'compose.yml' ]; then \
		$(DOCKER_COMPOSE) up -d --remove-orphans; \
	fi

.PHONY: docker_compose/build
docker_compose/build:
	@if [ -f 'compose.yaml' ] || [ -f 'compose.yml' ]; then \
		$(DOCKER_COMPOSE) build; \
	fi

.PHONY: docker_compose/stop
docker_compose/stop:
	@if [ -f 'compose.yaml' ] || [ -f 'compose.yml' ]; then \
		$(DOCKER_COMPOSE) down; \
	fi
